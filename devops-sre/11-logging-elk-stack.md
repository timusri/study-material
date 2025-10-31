# 11. Logging & ELK Stack

## 📚 Quick Summary

Centralized logging is essential for troubleshooting distributed systems and understanding application behavior!

**What You'll Learn:**
- **Logging Fundamentals**: Structured logging, log levels, best practices
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Fluentd/Fluent Bit**: Log collection and forwarding
- **Log Management**: Retention, rotation, parsing
- **Search & Analysis**: Query logs, create visualizations
- **Alerting**: Log-based alerts and anomaly detection

**Why This Matters:**
- Debug production issues faster
- Compliance and audit trails
- Security monitoring
- Performance analysis
- Interview questions: 10% are logging-based

**Interview Reality:**
"How do you troubleshoot issues across 100 microservices?" = Centralized logging!

---

## 📖 Simple Explanation

**What is Centralized Logging?**

**Without Centralized Logging:**
```
Service A: logs in server1:/var/log/app.log
Service B: logs in server2:/var/log/app.log
Service C: logs in server3:/var/log/app.log

Problem:
- SSH into each server
- Search each log file
- Correlate manually
- 😰 Takes hours
```

**With Centralized Logging:**
```
All services → Log collector → Central storage → Search UI

One search finds logs from all services
Correlation is automatic
😊 Takes seconds
```

**Real-World Analogy:**
```
Without: Searching through 100 filing cabinets
With: Google search across all documents
```

---

## ELK Stack Architecture

```
┌─────────────┐
│ Application │──┐
└─────────────┘  │
                 │ Logs
┌─────────────┐  │
│   Docker    │──┤
└─────────────┘  │
                 │
┌─────────────┐  │
│    Nginx    │──┤
└─────────────┘  │
                 ↓
            ┌─────────────┐
            │  Filebeat/  │ (Shipper)
            │  Logstash   │
            └──────┬──────┘
                   │
                   ↓
            ┌─────────────┐
            │Elasticsearch│ (Storage & Search)
            └──────┬──────┘
                   │
                   ↓
            ┌─────────────┐
            │   Kibana    │ (Visualization)
            └─────────────┘
```

---

## Elasticsearch

### Installation (Docker)

```bash
# Single node
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e "xpack.security.enabled=false" \
  elasticsearch:8.9.0

# Verify
curl http://localhost:9200
```

### Basic Operations

```bash
# Create index
curl -X PUT "localhost:9200/logs-2024.01.01"

# Index document
curl -X POST "localhost:9200/logs-2024.01.01/_doc" -H 'Content-Type: application/json' -d'
{
  "timestamp": "2024-01-01T10:00:00Z",
  "level": "ERROR",
  "service": "api",
  "message": "Database connection failed",
  "user_id": "user123"
}'

# Search
curl -X GET "localhost:9200/logs-*/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "level": "ERROR"
    }
  }
}'

# Aggregations
curl -X GET "localhost:9200/logs-*/_search" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "errors_by_service": {
      "terms": {
        "field": "service.keyword"
      }
    }
  }
}'
```

---

## Logstash

### Configuration

```ruby
# logstash.conf
input {
  # File input
  file {
    path => "/var/log/nginx/access.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
  
  # Beats input
  beats {
    port => 5044
  }
  
  # Syslog
  syslog {
    port => 514
  }
}

filter {
  # Parse Nginx logs
  if [path] =~ "access" {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
    
    date {
      match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
    
    geoip {
      source => "clientip"
    }
  }
  
  # Parse JSON logs
  if [type] == "app" {
    json {
      source => "message"
    }
  }
  
  # Add custom fields
  mutate {
    add_field => { "environment" => "production" }
    remove_field => [ "message" ]
  }
}

output {
  # Elasticsearch
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  
  # Stdout for debugging
  stdout {
    codec => rubydebug
  }
}
```

### Run Logstash

```bash
# Docker
docker run -d \
  --name logstash \
  -v /path/to/logstash.conf:/usr/share/logstash/pipeline/logstash.conf \
  -p 5044:5044 \
  logstash:8.9.0

# Binary
logstash -f logstash.conf
```

---

## Filebeat

### Configuration

```yaml
# filebeat.yml
filebeat.inputs:
  # Application logs
  - type: log
    enabled: true
    paths:
      - /var/log/app/*.log
    fields:
      service: myapp
      environment: production
    
    # Multiline for stack traces
    multiline.pattern: '^[[:space:]]'
    multiline.negate: false
    multiline.match: after
  
  # Docker containers
  - type: container
    paths:
      - '/var/lib/docker/containers/*/*.log'

# Processors
processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~
  - add_docker_metadata: ~

# Output to Elasticsearch
output.elasticsearch:
  hosts: ["localhost:9200"]
  index: "filebeat-%{+yyyy.MM.dd}"

# Or output to Logstash
output.logstash:
  hosts: ["localhost:5044"]

# Kibana dashboard
setup.kibana:
  host: "localhost:5601"

# Enable modules
filebeat.modules:
  - module: nginx
    access:
      enabled: true
      var.paths: ["/var/log/nginx/access.log"]
    error:
      enabled: true
      var.paths: ["/var/log/nginx/error.log"]
```

### Run Filebeat

```bash
# Docker
docker run -d \
  --name filebeat \
  --user=root \
  -v /path/to/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro \
  -v /var/lib/docker/containers:/var/lib/docker/containers:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  docker.elastic.co/beats/filebeat:8.9.0

# Setup dashboards
filebeat setup --dashboards
```

---

## Kibana

### Installation

```bash
# Docker
docker run -d \
  --name kibana \
  -p 5601:5601 \
  -e "ELASTICSEARCH_HOSTS=http://localhost:9200" \
  kibana:8.9.0

# Access: http://localhost:5601
```

### KQL (Kibana Query Language)

```
# Simple search
error

# Field search
level: ERROR
service: "api"
status_code: 500

# Range
response_time > 1000
timestamp >= "2024-01-01" and timestamp < "2024-01-02"

# Wildcards
message: "database*"
user: user*

# Boolean
level: ERROR and service: api
level: ERROR or level: WARN
level: ERROR and not service: monitoring

# Exists
_exists_: user_id

# Complex
(level: ERROR or level: FATAL) and service: api and response_time > 1000
```

---

## Structured Logging

### Application Logging Best Practices

**Python Example:**
```python
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno
        }
        
        # Add extra fields
        if hasattr(record, 'user_id'):
            log_data['user_id'] = record.user_id
        if hasattr(record, 'request_id'):
            log_data['request_id'] = record.request_id
            
        return json.dumps(log_data)

# Configure logger
logger = logging.getLogger('myapp')
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Usage
logger.info('User logged in', extra={'user_id': 'user123', 'request_id': 'req456'})
logger.error('Database error', extra={'error': str(e), 'query': query})
```

**Node.js Example:**
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  defaultMeta: { service: 'myapp' },
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Usage
logger.info('User logged in', { userId: 'user123', requestId: 'req456' });
logger.error('Database error', { error: err.message, query: query });
```

---

## Log Levels

```
TRACE   - Very detailed, typically only during development
DEBUG   - Detailed information for debugging
INFO    - General informational messages
WARN    - Warning messages, potential issues
ERROR   - Error messages, but app continues
FATAL   - Critical errors, app may crash

Production: INFO and above
Development: DEBUG and above
Troubleshooting: TRACE
```

---

## Fluentd

### Configuration

```ruby
# fluent.conf
<source>
  @type tail
  path /var/log/nginx/access.log
  pos_file /var/log/td-agent/nginx-access.pos
  tag nginx.access
  <parse>
    @type nginx
  </parse>
</source>

<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<filter **>
  @type record_transformer
  <record>
    hostname "#{Socket.gethostname}"
    environment "production"
  </record>
</filter>

<match nginx.**>
  @type elasticsearch
  host localhost
  port 9200
  index_name nginx-%Y.%m.%d
  type_name nginx_log
  logstash_format true
  <buffer>
    flush_interval 10s
  </buffer>
</match>
```

---

## Docker Logging

### Docker Compose with Logging

```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        labels: "service,environment"
    labels:
      - "service=myapp"
      - "environment=production"
  
  # Fluentd for log collection
  fluentd:
    image: fluent/fluentd:latest
    ports:
      - "24224:24224"
    volumes:
      - ./fluent.conf:/fluentd/etc/fluent.conf
```

### Kubernetes Logging

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:latest
    # Logs go to stdout/stderr (standard practice)
  
  # Sidecar for log processing
  - name: log-processor
    image: busybox
    command: ['sh', '-c', 'tail -f /var/log/app.log']
    volumeMounts:
    - name: logs
      mountPath: /var/log
  
  volumes:
  - name: logs
    emptyDir: {}
```

---

## Log Retention & Management

### Index Lifecycle Management

```json
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "1d"
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "freeze": {}
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

### Curator (Old Way)

```yaml
# curator.yml
actions:
  1:
    action: delete_indices
    description: Delete logs older than 30 days
    options:
      ignore_empty_list: True
    filters:
      - filtertype: pattern
        kind: prefix
        value: logs-
      - filtertype: age
        source: name
        direction: older
        timestring: '%Y.%m.%d'
        unit: days
        unit_count: 30
```

---

## Alerting on Logs

### ElastAlert

```yaml
# elastalert_rules/error_spike.yaml
name: Error Spike Alert
type: frequency
index: logs-*

# Alert if > 10 errors in 5 minutes
num_events: 10
timeframe:
  minutes: 5

filter:
  - term:
      level: "ERROR"

alert:
  - slack:
      slack_webhook_url: "https://hooks.slack.com/YOUR/WEBHOOK"
  - email:
      email: "ops@example.com"

alert_text: |
  Error spike detected!
  Service: {service}
  Count: {num_hits}
```

---

## Best Practices

### 1. Structured Logging

```python
# ✅ GOOD - Structured (JSON)
logger.info(
    'User action',
    extra={
        'user_id': '123',
        'action': 'purchase',
        'item_id': '456',
        'amount': 99.99
    }
)

# ❌ BAD - Unstructured
logger.info(f'User 123 purchased item 456 for $99.99')
```

### 2. Include Context

```python
# ✅ GOOD - With context
logger.error(
    'Payment failed',
    extra={
        'user_id': user.id,
        'payment_id': payment.id,
        'error_code': e.code,
        'gateway': 'stripe',
        'request_id': request_id
    }
)

# ❌ BAD - No context
logger.error('Payment failed')
```

### 3. Correlation IDs

```python
import uuid
from contextvars import ContextVar

request_id_var = ContextVar('request_id', default=None)

@app.middleware('http')
async def add_request_id(request, call_next):
    request_id = request.headers.get('X-Request-ID', str(uuid.uuid4()))
    request_id_var.set(request_id)
    response = await call_next(request)
    response.headers['X-Request-ID'] = request_id
    return response

# All logs include request_id
logger.info('Processing request', extra={'request_id': request_id_var.get()})
```

### 4. Don't Log Sensitive Data

```python
# ❌ BAD
logger.info(f'User login: {email}, password: {password}')

# ✅ GOOD
logger.info('User login', extra={'email': email})  # No password!

# ✅ GOOD - Mask sensitive data
logger.info('Credit card', extra={'card': f"****{card_number[-4:]}"})
```

---

## Interview Questions

### Q1: What is the ELK stack?
**Answer:**
```
ELK = Elasticsearch + Logstash + Kibana

Elasticsearch:
- Search and analytics engine
- Stores logs in indexes
- Full-text search

Logstash:
- Log collection and processing
- Parse, transform, enrich
- Multiple inputs/outputs

Kibana:
- Visualization layer
- Dashboards and graphs
- Search interface

Modern stack often adds:
- Beats (lightweight shippers)
- Now called "Elastic Stack"
```

---

### Q2: Structured vs unstructured logs?
**Answer:**
```
Unstructured:
"2024-01-01 User john purchased item 123 for $50"
- Hard to parse
- Can't filter easily
- No aggregation

Structured (JSON):
{
  "timestamp": "2024-01-01T10:00:00Z",
  "user": "john",
  "action": "purchase",
  "item_id": "123",
  "amount": 50
}
- Easy to parse
- Filterable fields
- Enable aggregations
- Better for analysis

Always use structured logging in production!
```

---

## Quick Reference

```bash
# Elasticsearch
curl -X GET "localhost:9200/_cat/indices?v"
curl -X GET "localhost:9200/logs-*/_search?q=error"

# Filebeat
filebeat test config
filebeat test output
filebeat setup --dashboards

# Logstash
logstash -f logstash.conf --config.test_and_exit

# Kibana Query Language
level: ERROR
service: api AND status: 500
timestamp >= "2024-01-01"
```

---

## Summary

**Key Takeaways:**
1. Centralized logging is essential for distributed systems
2. ELK stack = Elasticsearch + Logstash + Kibana
3. Always use structured logging (JSON)
4. Include correlation IDs
5. Don't log sensitive data
6. Implement proper retention policies
7. Set up alerts on log patterns

**Next Steps:**
1. Set up ELK stack locally
2. Implement structured logging in your apps
3. Create Kibana dashboards
4. Set up log-based alerts
5. Implement log retention
6. Practice KQL queries

**Happy Logging! 📝**

