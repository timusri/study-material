# 10. Monitoring & Observability

## 📚 Quick Summary

Monitoring and Observability are essential for understanding system health and performance in production!

**What You'll Learn:**
- **Monitoring Fundamentals**: Metrics, logs, traces
- **Prometheus**: Industry-standard metrics collection
- **Grafana**: Visualization and dashboards
- **Alerting**: Alert design, notification, escalation
- **Observability**: The three pillars
- **APM Tools**: Application Performance Monitoring
- **Best Practices**: What to monitor, alert fatigue

**Why This Matters:**
- Detect issues before users notice
- Understand system behavior
- Debug production problems
- Capacity planning
- Interview questions: 20% are monitoring-based

**Interview Reality:**
"How do you monitor production systems?" = Must-know for DevOps/SRE!

---

## 📖 Simple Explanation

**What is Monitoring?**

**Before Monitoring:**
```
User: "Website is down!"
You: "Let me check..." (30 minutes later)
You: "Found it! Server crashed 2 hours ago"
😰
```

**With Monitoring:**
```
Alert: "CPU at 95%"  
You: Scale up servers
User: Never notices any issue
😊
```

**Real-World Analogy:**
```
Monitoring = Car Dashboard
- Speedometer (current speed)
- Fuel gauge (resource usage)
- Warning lights (alerts)
- Temperature gauge (health)

You don't open hood while driving,
you check dashboard!
```

---

**The Three Pillars of Observability:**

```
1. Metrics (Numbers)
   - CPU usage: 75%
   - Response time: 250ms
   - Error rate: 0.1%

2. Logs (Events)
   - "User logged in"
   - "Database connection failed"
   - "Payment processed"

3. Traces (Flows)
   - Request path through system
   - Which services were called
   - Where time was spent

Metrics = WHAT is happening
Logs = WHY it happened
Traces = WHERE it happened
```

---

## Table of Contents
- [Monitoring Fundamentals](#monitoring-fundamentals)
- [Prometheus](#prometheus)
- [Grafana](#grafana)
- [Alerting](#alerting)
- [Log Aggregation](#log-aggregation)
- [Distributed Tracing](#distributed-tracing)
- [APM Tools](#apm-tools)
- [Cloud Monitoring](#cloud-monitoring)
- [Best Practices](#best-practices)
- [Interview Questions](#interview-questions)

---

## Monitoring Fundamentals

### Types of Monitoring

**1. Infrastructure Monitoring**
```
- CPU usage
- Memory usage
- Disk I/O
- Network traffic
- System load

Tools: Prometheus, Datadog, CloudWatch
```

**2. Application Monitoring**
```
- Response time
- Error rate
- Request rate
- Database query time
- Cache hit rate

Tools: New Relic, AppDynamics, Datadog APM
```

**3. Synthetic Monitoring**
```
- Uptime checks
- API health checks
- User journey simulation

Tools: Pingdom, UptimeRobot, StatusCake
```

**4. Real User Monitoring (RUM)**
```
- Actual user experience
- Page load times
- JavaScript errors
- Geographic performance

Tools: Google Analytics, New Relic Browser
```

---

### Key Metrics (The Four Golden Signals)

```
1. Latency
   - How long requests take
   - Target: < 200ms for web
   - P50, P95, P99 percentiles

2. Traffic
   - Requests per second
   - Concurrent users
   - Bandwidth usage

3. Errors
   - Error rate (%)
   - Error types
   - 4xx vs 5xx

4. Saturation
   - Resource utilization
   - CPU, memory, disk
   - Queue depths
```

---

### RED Method (For Services)

```
Rate:    Request rate (requests/sec)
Errors:  Error rate (%)
Duration: Request duration (latency)

# All you need for microservices!
```

---

### USE Method (For Resources)

```
Utilization: % time resource is busy
Saturation:  Queue depth/wait time
Errors:      Error count

# For: CPU, memory, disk, network
```

---

## Prometheus

### 📖 Simple Explanation

Prometheus is open-source monitoring system with powerful query language. Industry standard for Kubernetes.

---

### Architecture

```
┌─────────────┐
│Prometheus   │←──── Scrape metrics
│  Server     │
└──────┬──────┘
       │
       ├──→ Time Series DB (storage)
       ├──→ Alert Manager (alerts)
       └──→ Grafana (visualization)

Pull Model:
Prometheus scrapes metrics from targets
```

---

### Installation

```bash
# Docker
docker run -d \
  -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus

# Kubernetes (Helm)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack

# Binary
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar xvfz prometheus-2.45.0.linux-amd64.tar.gz
cd prometheus-2.45.0.linux-amd64
./prometheus --config.file=prometheus.yml
```

---

### Configuration

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  
  external_labels:
    cluster: 'production'
    region: 'us-east-1'

# Alert manager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']

# Scrape configurations
scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  # Node Exporter (system metrics)
  - job_name: 'node'
    static_configs:
      - targets: 
          - 'server1:9100'
          - 'server2:9100'
          - 'server3:9100'
  
  # Application
  - job_name: 'myapp'
    scrape_interval: 10s
    static_configs:
      - targets: ['app1:8080', 'app2:8080']
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
  
  # Kubernetes service discovery
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__

# Recording rules
rule_files:
  - 'rules/*.yml'
```

---

### Exporters

**Node Exporter (System Metrics):**
```bash
# Install
docker run -d \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  prom/node-exporter \
  --path.rootfs=/host

# Metrics exposed on :9100/metrics
```

**Application Metrics (Python):**
```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time

# Metrics
request_count = Counter('http_requests_total', 'Total HTTP requests', ['method', 'endpoint'])
request_duration = Histogram('http_request_duration_seconds', 'HTTP request duration')
active_connections = Gauge('active_connections', 'Number of active connections')

@request_duration.time()
def process_request(request):
    request_count.labels(method=request.method, endpoint=request.path).inc()
    # Process request
    time.sleep(0.1)
    return "OK"

# Start metrics server
if __name__ == '__main__':
    start_http_server(8000)  # Metrics on :8000/metrics
    # Start main application
```

**Custom Exporter (Go):**
```go
package main

import (
    "net/http"
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    requests = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total HTTP requests",
        },
        []string{"method", "endpoint"},
    )
    
    duration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "http_request_duration_seconds",
            Help: "HTTP request duration",
            Buckets: []float64{.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5, 10},
        },
        []string{"method", "endpoint"},
    )
)

func init() {
    prometheus.MustRegister(requests)
    prometheus.MustRegister(duration)
}

func main() {
    http.Handle("/metrics", promhttp.Handler())
    http.ListenAndServe(":8080", nil)
}
```

---

### PromQL (Query Language)

**Basic Queries:**
```promql
# Current CPU usage
node_cpu_seconds_total

# Rate of requests (per second)
rate(http_requests_total[5m])

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# Memory usage %
100 * (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes))

# Top 10 endpoints by request count
topk(10, sum by (endpoint) (rate(http_requests_total[5m])))

# Disk usage per mount
100 - ((node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100)
```

**Advanced Queries:**
```promql
# Request rate by service
sum by (service) (rate(http_requests_total[5m]))

# Average response time
avg(rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m]))

# Requests per second with prediction
predict_linear(http_requests_total[1h], 3600)

# Aggregation across instances
avg(rate(cpu_usage[5m])) by (cluster, environment)

# Alert if error rate > 1% for 5 minutes
(sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))) > 0.01
```

---

### Recording Rules

```yaml
# rules/app.yml
groups:
  - name: app_rules
    interval: 30s
    rules:
      # Pre-compute expensive queries
      - record: job:http_requests:rate5m
        expr: sum by (job) (rate(http_requests_total[5m]))
      
      - record: job:http_request_duration:p95
        expr: histogram_quantile(0.95, sum by (job, le) (rate(http_request_duration_seconds_bucket[5m])))
      
      - record: instance:cpu_usage:ratio
        expr: 1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m]))
```

---

## Grafana

### 📖 Simple Explanation

Grafana turns metrics into beautiful, actionable dashboards.

---

### Installation

```bash
# Docker
docker run -d -p 3000:3000 --name=grafana grafana/grafana

# Kubernetes (Helm)
helm install grafana grafana/grafana

# Binary (Linux)
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_10.0.0_amd64.deb
sudo dpkg -i grafana_10.0.0_amd64.deb
sudo systemctl start grafana-server

# Access: http://localhost:3000 (admin/admin)
```

---

### Dashboard Configuration

**System Overview Dashboard:**
```json
{
  "dashboard": {
    "title": "System Overview",
    "panels": [
      {
        "title": "CPU Usage",
        "targets": [{
          "expr": "100 - (avg(rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)"
        }],
        "type": "graph"
      },
      {
        "title": "Memory Usage",
        "targets": [{
          "expr": "100 * (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes))"
        }],
        "type": "gauge"
      },
      {
        "title": "Request Rate",
        "targets": [{
          "expr": "sum(rate(http_requests_total[5m]))"
        }],
        "type": "stat"
      },
      {
        "title": "Error Rate",
        "targets": [{
          "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100"
        }],
        "type": "graph",
        "alert": {
          "conditions": [
            {
              "evaluator": {
                "params": [1],
                "type": "gt"
              }
            }
          ]
        }
      }
    ]
  }
}
```

---

### Dashboard as Code

```yaml
# dashboards/app-dashboard.yml
apiVersion: 1

providers:
  - name: 'default'
    folder: 'Services'
    type: file
    options:
      path: /var/lib/grafana/dashboards

# Provision data source
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```

---

### Grafana Alerts

```yaml
# Alert configuration
groups:
  - name: application_alerts
    interval: 1m
    rules:
      - alert: HighErrorRate
        expr: (sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }}% for {{ $labels.job }}"
      
      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
```

---

## Alerting

### Alert Manager Configuration

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'

route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'slack-notifications'
  
  routes:
    # Critical alerts go to pagerduty
    - match:
        severity: critical
      receiver: pagerduty
      continue: true
    
    # Warning alerts to slack only
    - match:
        severity: warning
      receiver: slack-notifications

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#alerts'
        title: 'Alert: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
        send_resolved: true
  
  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'
        description: '{{ .GroupLabels.alertname }}'
  
  - name: 'email'
    email_configs:
      - to: 'ops-team@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'alertmanager@example.com'
        auth_password: 'password'

inhibit_rules:
  # Don't alert on database down if whole cluster is down
  - source_match:
      severity: 'critical'
      alertname: 'ClusterDown'
    target_match:
      severity: 'warning'
    equal: ['cluster']
```

---

### Alert Rules

```yaml
# alerts/app.yml
groups:
  - name: application
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[5m]))
            /
            sum(rate(http_requests_total[5m]))
          ) > 0.05
        for: 5m
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "High error rate on {{ $labels.instance }}"
          description: "Error rate is {{ $value | humanizePercentage }} (threshold: 5%)"
          runbook: "https://wiki.example.com/runbooks/high-error-rate"
      
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, job)
          ) > 1
        for: 10m
        labels:
          severity: warning
          team: backend
        annotations:
          summary: "High latency detected"
          description: "P95 latency is {{ $value }}s on {{ $labels.job }}"
      
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} has been down for more than 1 minute"
      
      - alert: DiskSpaceLow
        expr: |
          (
            node_filesystem_avail_bytes{mountpoint="/"}
            /
            node_filesystem_size_bytes{mountpoint="/"}
          ) < 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Only {{ $value | humanizePercentage }} disk space remaining"
      
      - alert: HighMemoryUsage
        expr: |
          (
            1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)
          ) > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
```

---

## Log Aggregation

### Loki (Logs)

```yaml
# promtail-config.yml (log collector)
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*log
  
  - job_name: containers
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s
    relabel_configs:
      - source_labels: ['__meta_docker_container_name']
        regex: '/(.*)'
        target_label: 'container'
```

**Query Logs:**
```logql
# All logs from app
{job="myapp"}

# Error logs
{job="myapp"} |= "error"

# JSON parsing
{job="myapp"} | json | status_code >= 500

# Rate of errors
rate({job="myapp"} |= "error" [5m])

# Count by status code
sum by (status_code) (rate({job="myapp"} | json [5m]))
```

---

## Distributed Tracing

### Jaeger

```yaml
# Docker Compose
version: '3'
services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686"  # UI
      - "14268:14268"
      - "9411:9411"
    environment:
      - COLLECTOR_ZIPKIN_HTTP_PORT=9411
```

**Application Instrumentation (Python):**
```python
from jaeger_client import Config
from flask import Flask
from opentracing_instrumentation.client_hooks import install_all_patches

app = Flask(__name__)

# Jaeger config
config = Config(
    config={
        'sampler': {'type': 'const', 'param': 1},
        'logging': True,
    },
    service_name='myapp',
)
tracer = config.initialize_tracer()

# Install patches for automatic instrumentation
install_all_patches()

@app.route('/api/users')
def get_users():
    with tracer.start_span('get_users') as span:
        span.set_tag('endpoint', '/api/users')
        
        # Database call
        with tracer.start_span('db_query', child_of=span) as db_span:
            db_span.set_tag('query', 'SELECT * FROM users')
            users = database.query('SELECT * FROM users')
        
        # External API call
        with tracer.start_span('external_api', child_of=span) as api_span:
            api_span.set_tag('api', 'enrichment-service')
            enriched = enrich_users(users)
        
        return enriched
```

---

## APM Tools

### New Relic

```python
# newrelic.ini configuration
import newrelic.agent
newrelic.agent.initialize('newrelic.ini')

# Application
from flask import Flask
app = Flask(__name__)

# Automatic instrumentation
application = newrelic.agent.WSGIApplicationWrapper(app)

@app.route('/api/endpoint')
def endpoint():
    # Custom transaction attributes
    newrelic.agent.add_custom_attribute('user_id', user_id)
    newrelic.agent.add_custom_attribute('plan', 'premium')
    
    # Custom metrics
    newrelic.agent.record_custom_metric('Custom/ProcessingTime', duration)
    
    return "OK"
```

---

### Datadog

```python
from ddtrace import tracer
from ddtrace.contrib.flask import TraceMiddleware

app = Flask(__name__)
traced_app = TraceMiddleware(app, tracer, service="myapp")

@app.route('/api/endpoint')
def endpoint():
    # Custom span
    with tracer.trace("custom.operation", service="myapp") as span:
        span.set_tag("user.id", user_id)
        span.set_metric("processing.time", duration)
        # Do work
        process_request()
    
    return "OK"
```

---

## Cloud Monitoring

### AWS CloudWatch

```python
import boto3

cloudwatch = boto3.client('cloudwatch')

# Put custom metric
cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[
        {
            'MetricName': 'ProcessedOrders',
            'Value': 150,
            'Unit': 'Count',
            'Timestamp': datetime.utcnow(),
            'Dimensions': [
                {
                    'Name': 'Environment',
                    'Value': 'production'
                }
            ]
        }
    ]
)

# Create alarm
cloudwatch.put_metric_alarm(
    AlarmName='HighCPU',
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=2,
    MetricName='CPUUtilization',
    Namespace='AWS/EC2',
    Period=300,
    Statistic='Average',
    Threshold=80.0,
    ActionsEnabled=True,
    AlarmActions=['arn:aws:sns:us-east-1:123456789:MyTopic'],
    AlarmDescription='Alert when CPU exceeds 80%'
)
```

---

### Azure Monitor

```python
from azure.monitor.opentelemetry import configure_azure_monitor
from opentelemetry import trace

configure_azure_monitor(connection_string="YOUR_CONNECTION_STRING")
tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("my_operation"):
    # Your code
    process_request()
```

---

## Best Practices

### 1. What to Monitor

```yaml
# System Level
- CPU usage
- Memory usage
- Disk usage and I/O
- Network traffic
- System load

# Application Level
- Request rate
- Error rate  
- Response time (P50, P95, P99)
- Database query time
- Queue length
- Cache hit rate

# Business Level
- Active users
- Transactions/orders
- Revenue
- Conversion rate
```

---

### 2. Alert Design

```
Good Alerts:
✓ Actionable (you can fix it)
✓ High signal/noise ratio
✓ Clear resolution steps
✓ Appropriate severity

Bad Alerts:
✗ Can't be acted upon
✗ Too sensitive (alert fatigue)
✗ Unclear what to do
✗ Boy who cried wolf

Example Good Alert:
"API error rate > 5% for 5 minutes"
→ Actionable: Check logs, rollback, scale
→ Clear threshold
→ Has context (5 minutes avoids flapping)
```

---

### 3. Avoid Alert Fatigue

```
Strategies:
1. Alert on symptoms, not causes
   ✓ "Users experiencing errors"
   ✗ "Disk at 85%"

2. Use appropriate thresholds
   - Test and tune
   - Use percentiles, not averages

3. Group related alerts
   - If service down, don't alert on every endpoint

4. Have clear escalation paths
   - Warning → Team channel
   - Critical → Page oncall

5. Review and remove noisy alerts
   - If ignored 5 times, remove or fix
```

---

### 4. Dashboard Design

```
Best Practices:
1. One dashboard per service
2. Most important metrics at top
3. Use consistent colors
   - Green = good
   - Yellow = warning
   - Red = critical
4. Show trends (not just current)
5. Include SLI/SLO targets
6. Add links to runbooks

Example Dashboard Layout:
┌────────────────────────────────┐
│ Overall Health: 🟢              │
├────────────────────────────────┤
│ Request Rate  │ Error Rate     │
│ [Graph]       │ [Gauge 0.1%]   │
├───────────────┴────────────────┤
│ P95 Latency                    │
│ [Graph with SLO target line]   │
├────────────────────────────────┤
│ Top 5 Slow Endpoints           │
│ [Table]                        │
└────────────────────────────────┘
```

---

### 5. Retention and Sampling

```yaml
# Prometheus retention
storage:
  tsdb:
    retention.time: 15d  # Keep 15 days
    retention.size: 50GB  # Or 50GB max

# Longer retention with downsampling
# Use Thanos or Cortex for long-term storage

# Sampling for high-cardinality
# Sample 1% of traces for performance
sample_rate: 0.01
```

---

## Interview Questions

### Q1: Explain the difference between monitoring and observability
**Answer:**
```
Monitoring:
- Known unknowns
- Predefined metrics/dashboards
- "Is CPU high?"
- Answer specific questions

Observability:
- Unknown unknowns
- Ad-hoc investigation
- "Why is this slow?"
- Explore and discover

Three Pillars:
1. Metrics - WHAT is happening
2. Logs - WHY it happened  
3. Traces - WHERE it happened

Example:
Monitoring: Dashboard shows "Error rate high"
Observability: Trace request to find exact failing service and line of code
```

---

### Q2: What are the Four Golden Signals?
**Answer:**
```
1. Latency
   - Request duration
   - P50, P95, P99
   - Target: < 200ms web, < 100ms API

2. Traffic  
   - Requests per second
   - Concurrent users
   - Bandwidth

3. Errors
   - Error rate (%)
   - 4xx vs 5xx
   - Target: < 0.1% for 5xx

4. Saturation
   - Resource utilization
   - Queue depths
   - "How full is it?"

These 4 metrics tell you system health!
```

---

### Q3: How do you design good alerts?
**Answer:**
```
Good Alert Characteristics:
1. Actionable - Can you fix it?
2. Meaningful - Does it matter?
3. Rare - Not constantly firing
4. Clear - What and how to fix

Formula:
Alert = Symptom + Threshold + Duration

Example:
❌ BAD: "Disk 85% full"
   - Not urgent, might be normal

✓ GOOD: "Error rate > 5% for 5 minutes"
   - Affects users
   - Clear threshold
   - Duration prevents flapping
   - Actionable (rollback/scale)

Severity Levels:
- Critical: Page immediately (user-facing)
- Warning: Slack notification (investigate soon)
- Info: Log only (FYI)
```

---

### Q4: Pull vs Push metrics collection?
**Answer:**
```
Pull (Prometheus):
+ Service discovery
+ Targets control their metrics
+ Detect dead targets
+ Less network traffic
- Firewall complexity
- Targets must be reachable

Push (StatsD, CloudWatch):
+ Works through firewalls
+ Short-lived jobs
+ Simpler networking
- No service discovery
- Can't detect dead clients
- More network traffic

Recommendation:
- Pull for long-running services (Prometheus)
- Push for short-lived jobs, Lambda
```

---

### Q5: How do you monitor microservices?
**Answer:**
```
Challenges:
- Many services
- Distributed traces
- Service dependencies
- Dynamic instances

Solutions:

1. Service Mesh (Istio)
   - Automatic metrics
   - Distributed tracing
   - No code changes

2. APM Tools (New Relic, Datadog)
   - Automatic instrumentation
   - Dependency mapping
   - Transaction tracing

3. Centralized Logging (ELK)
   - Correlate across services
   - Request ID tracking

4. Distributed Tracing (Jaeger)
   - Track request path
   - Find bottlenecks

Key Metrics:
- Per-service: Rate, Errors, Duration (RED)
- Dependencies: Call graph, latency
- Infrastructure: K8s metrics
```

---

### Q6: What is SLO/SLI/SLA?
**Answer:**
```
SLI (Service Level Indicator):
- What you measure
- Examples: Latency, availability, error rate

SLO (Service Level Objective):
- Target for SLI
- Example: "99.9% of requests < 200ms"

SLA (Service Level Agreement):
- Contract with customer
- Example: "99.9% uptime or money back"

Relationship:
SLI = Measurement
SLO = Goal
SLA = Promise

Example:
SLI: API response time
SLO: 95% of requests < 200ms (internal goal)
SLA: 99% uptime guarantee (customer contract)

Error Budget:
100% - SLO = Budget for failures
99.9% SLO = 0.1% error budget = 43min downtime/month
```

---

## Quick Reference

```bash
# Prometheus
./prometheus --config.file=prometheus.yml

# PromQL Examples
rate(http_requests_total[5m])
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
up == 0  # Instance down

# Alert Manager
./alertmanager --config.file=alertmanager.yml

# Grafana
docker run -d -p 3000:3000 grafana/grafana

# Node Exporter
docker run -d --net=host prom/node-exporter
```

---

## Summary

**Key Takeaways:**
1. Monitoring = metrics + logs + traces
2. Four Golden Signals: Latency, Traffic, Errors, Saturation
3. Prometheus + Grafana = industry standard
4. Alert on symptoms, not causes
5. Avoid alert fatigue
6. Observability enables debugging unknown issues

**Next Steps:**
1. Set up Prometheus + Grafana
2. Instrument your application
3. Create dashboards
4. Configure alerts
5. Test and tune thresholds
6. Implement distributed tracing

**Remember:**
- Monitor what matters
- Alert thoughtfully
- Visualize clearly
- Trace thoroughly
- Measure business impact

**Happy Monitoring! 📊**

