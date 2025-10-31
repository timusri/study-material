# 14. SRE Principles

## 📚 Quick Summary

Site Reliability Engineering (SRE) is Google's approach to operating large-scale systems reliably!

**What You'll Learn:**
- **SRE Fundamentals**: What SRE is and isn't
- **SLI/SLO/SLA**: Service level indicators, objectives, agreements
- **Error Budgets**: Balance innovation with reliability
- **Toil Reduction**: Automate repetitive work
- **On-Call Best Practices**: Sustainable on-call
- **SRE Culture**: Blameless postmortems, psychological safety

**Why This Matters:**
- Build reliable systems at scale
- Balance feature velocity with stability
- Data-driven reliability decisions
- Sustainable operations
- Interview questions: 20% for SRE roles

**Interview Reality:**
"How do you balance reliability and feature development?" = Error budgets!

---

## 📖 Simple Explanation

**What is SRE?**

```
Traditional Ops:
- Manual operations
- "Keep it running"
- Reactive to outages
- Separated from development

SRE:
- Software engineering for operations
- Automate everything
- Proactive reliability
- Developers + Ops collaboration
```

**SRE Motto:**
"Hope is not a strategy" - Measure everything, automate everything

---

## SLI, SLO, SLA

### Service Level Indicator (SLI)

**What you measure:**
```
Availability SLI:
successful_requests / total_requests

Latency SLI:
requests_under_200ms / total_requests

Freshness SLI (for data pipelines):
records_processed_on_time / total_records
```

**Common SLIs:**
- **Availability**: Is the service up?
- **Latency**: How fast does it respond?
- **Quality**: Are responses correct?
- **Durability**: Is data preserved?

---

### Service Level Objective (SLO)

**Your reliability target:**
```
Examples:
- 99.9% of requests succeed (availability)
- 95% of requests complete in < 200ms (latency)
- 99% of data processing completes within 1 hour (freshness)

SLO = SLI + Target
```

**Choosing SLOs:**
```
Too strict (99.999%):
- Expensive
- Slow feature development
- Unnecessary for most services

Too loose (95%):
- Poor user experience
- Lost customers

Sweet spot: 99.9% - 99.95%
Balance cost and reliability
```

---

### Service Level Agreement (SLA)

**Contract with customers:**
```
SLA = SLO + Consequences

Example:
"We guarantee 99.9% uptime.
If we don't meet this, you get:
- < 99.9%: 10% credit
- < 99%: 25% credit
- < 95%: 50% credit"

SLA should be more lenient than SLO
(SLO: 99.9%, SLA: 99.5%)
```

---

### The Relationship

```
SLI (Measurement):
"99.95% of requests succeeded last month"

SLO (Target):
"99.9% of requests should succeed"

SLA (Promise):
"We guarantee 99.5% or money back"

       SLI ←── Actual measurement
        ↓
       SLO ←── Internal target (stricter)
        ↓
       SLA ←── Customer promise (looser)
```

---

## Error Budgets

### 📖 Simple Explanation

**Error budget = 100% - SLO**

```
SLO: 99.9% uptime
Error budget: 0.1% downtime

Per month (30 days):
0.1% of 720 hours = 43 minutes downtime allowed

This is your budget to:
- Deploy new features
- Run experiments
- Take risks
```

---

### How Error Budgets Work

```
Scenario 1: Budget Remaining
Error budget: 43 minutes/month
Used: 10 minutes

Status: 33 minutes remaining ✓
Action: Ship features aggressively

Scenario 2: Budget Exhausted
Error budget: 43 minutes/month
Used: 50 minutes

Status: Over budget by 7 minutes ✗
Action: Freeze deployments, focus on reliability
```

---

### Error Budget Policy

```yaml
# Example policy
error_budget_policy:
  slo: 99.9%  # 43 min/month
  
  when_budget_remaining:
    - Deploy new features
    - Experiment with new tech
    - Faster release cadence
  
  when_25%_remaining:
    - Slow down deployments
    - Increase test coverage
    - Review recent changes
  
  when_budget_exhausted:
    - Freeze all feature deployments
    - Focus on reliability only
    - Root cause analysis
    - Implement fixes
  
  exceptions:
    - Security patches (always allowed)
    - Critical business features (with approval)
```

---

## Toil Reduction

### What is Toil?

```
Toil = Manual, repetitive, automatable work

Characteristics:
✓ Manual (requires human)
✓ Repetitive (happens regularly)
✓ Automatable (can be programmed)
✓ Tactical (no enduring value)
✓ Scales linearly (more work = more toil)

Examples:
- Manually restarting servers
- Copy-pasting deployments
- Running the same scripts
- Resetting passwords
- Clearing disk space manually
```

**Not Toil:**
- Debugging new issues (not repetitive)
- Writing automation (adds enduring value)
- System design (strategic)

---

### Measuring Toil

```
Target: < 50% of SRE time on toil

Track:
- Hours spent on toil per week
- % of time on toil vs engineering

If > 50% toil:
- Automate common tasks
- Reduce manual interventions
- Improve self-healing
```

---

### Eliminating Toil

```
1. Identify toil
   - What do you do repeatedly?
   - What wakes you up at night?
   - What manual steps in runbooks?

2. Measure impact
   - How often does it happen?
   - How much time does it take?
   - What's the ROI of automation?

3. Automate
   - Write scripts
   - Build tools
   - Implement self-healing

4. Monitor and iterate
   - Did toil decrease?
   - What new toil emerged?
```

**Example:**
```bash
# Manual (toil):
1. SSH into server
2. Run df -h
3. Find large files
4. Delete log files
5. Restart service

# Automated:
#!/bin/bash
# cleanup.sh (runs via cron)
if [ $(df / | tail -1 | awk '{print $5}' | sed 's/%//') -gt 80 ]; then
    find /var/log -name "*.log" -mtime +7 -delete
    systemctl restart myapp
fi
```

---

## On-Call Best Practices

### Sustainable On-Call

```
Google's Rules:
1. Max 25% of time on-call
2. Max 2 events per 12-hour shift
3. At least 8 hours between shifts
4. Compensated (time off or pay)

If violated consistently:
- Hire more SREs
- Reduce service complexity
- Improve automation
- Better monitoring/alerting
```

---

### On-Call Checklist

```
Before On-Call:
□ Review runbooks
□ Test access (VPN, SSH, cloud console)
□ Check pager works
□ Know escalation contacts
□ Familiarize with recent changes
□ Read last week's incidents

During On-Call:
□ Respond within SLA (5-15 min typical)
□ Document all actions
□ Escalate if needed
□ Update incident channel
□ Follow runbooks

After On-Call:
□ Write postmortems
□ Update runbooks
□ File bugs for toil
□ Handoff summary to next on-call
```

---

### On-Call Rotation

```
Typical Setup:
- Primary on-call (first responder)
- Secondary on-call (escalation)
- 1 week rotations
- Daytime + nighttime coverage

Follow-the-sun:
Team A: Americas (8am-8pm EST)
Team B: Europe (8am-8pm GMT)
Team C: Asia (8am-8pm IST)
24/7 coverage without night shifts!
```

---

## Blameless Postmortems

### 📖 Simple Explanation

```
Blameless = Focus on systems, not people

❌ BAD:
"John deployed broken code and caused the outage"

✓ GOOD:
"Deployment lacked sufficient testing.
Action item: Add integration tests to CI/CD"
```

---

### Postmortem Template

```markdown
# Incident Postmortem: [TITLE]

## Summary
Brief description (2-3 sentences)

## Impact
- Duration: 2 hours (14:00-16:00 UTC)
- Users affected: 15% of traffic
- Revenue impact: $50K
- User experience: API timeouts, errors

## Root Cause
Database connection pool exhausted due to query leak

## Timeline
- 14:00: Monitoring alerts for high DB connections
- 14:05: On-call acknowledged
- 14:10: Identified connection leak in user service
- 14:20: Rolled back recent deployment
- 14:30: Connections started decreasing
- 16:00: Fully resolved

## Detection
- Automated alert (Prometheus)
- Customer reports via support

## Response
- Investigated DB metrics
- Reviewed recent deployments
- Rolled back to previous version
- Monitored recovery

## Root Cause
Code change introduced connection leak:
```python
# Bug: Connection not closed
def get_user(user_id):
    conn = db.get_connection()
    user = conn.query(f"SELECT * FROM users WHERE id={user_id}")
    return user  # Connection never closed!
```

## Resolution
Rolled back deployment, fixed code:
```python
def get_user(user_id):
    with db.get_connection() as conn:
        return conn.query(f"SELECT * FROM users WHERE id={user_id}")
```

## Action Items
1. [HIGH] Add connection leak detection to tests (@alice, 2024-02-01)
2. [HIGH] Implement connection pool monitoring (@bob, 2024-02-05)
3. [MED] Review all DB connection code (@team, 2024-02-10)
4. [MED] Add runbook for connection pool issues (@charlie, 2024-02-15)
5. [LOW] Improve canary deployment to catch this (@diana, 2024-02-20)

## Lessons Learned
- Connection pool monitoring was missing
- Tests didn't catch resource leaks
- Canary deployment was too quick

## What Went Well
- Alert fired immediately
- Team responded quickly
- Rollback was smooth
- Communication was clear
```

---

## Monitoring and Alerting Philosophy

### Monitoring Best Practices

```
Four Golden Signals:
1. Latency - How long requests take
2. Traffic - How many requests
3. Errors - How many fail
4. Saturation - How full is the system

Monitor:
✓ User-facing behavior (symptoms)
✗ Internal implementation (causes)

Example:
✓ "API returning errors" (symptom)
✗ "CPU at 80%" (cause)
```

---

### Alert Design

```
Good Alert:
- Actionable (you can fix it)
- User-impacting (matters to users)
- Novel (not already known)

Bad Alert:
- Can't be fixed immediately
- Doesn't affect users
- Already handled elsewhere

Example:
✓ "Error rate > 5% for 5 minutes"
  → Users affected, needs action

✗ "Disk 80% full"
  → Not urgent, not user-facing
  → Should be ticket, not page
```

---

## Capacity Planning

### 📖 Simple Explanation

```
Capacity Planning = Ensure enough resources for demand

Process:
1. Measure current usage
2. Forecast future demand
3. Provision resources ahead of time
```

---

### Capacity Planning Steps

```
1. Collect Data:
   - Current traffic
   - Resource usage (CPU, memory, disk)
   - Growth rate

2. Forecast:
   - Linear growth
   - Seasonal patterns
   - Special events

3. Plan:
   - When will we run out?
   - How much to add?
   - Lead time for provisioning?

4. Provision:
   - Add resources before needed
   - Test new capacity
   - Monitor results
```

**Example:**
```
Current: 1000 req/s
Growth: 10% per month
Capacity: 1500 req/s

Forecast:
Month 1: 1100 req/s (73% capacity)
Month 2: 1210 req/s (80% capacity) ← Warning
Month 3: 1331 req/s (89% capacity) ← Add capacity
Month 4: 1464 req/s (97% capacity) ← Critical

Action: Add capacity before Month 2
```

---

## Change Management

### 📖 Simple Explanation

```
Most outages are caused by changes

Change Management = Process to reduce change risk
```

---

### Progressive Rollouts

```
1. Dev → 2. Canary → 3. Staging → 4. Production

Canary Deployment:
- Deploy to 1% of production traffic
- Monitor for 30 minutes
- If good, increase to 10%
- Then 25%, 50%, 100%

Rollback at first sign of issues:
- Error rate increase
- Latency increase
- Resource usage spike
```

---

### Pre-Production Checklist

```
Before deploying to production:
□ Code reviewed
□ Tests pass (unit, integration, E2E)
□ Security scanned
□ Performance tested
□ Runbook updated
□ Rollback plan ready
□ Monitoring in place
□ Team notified
□ Error budget available
```

---

## Reliability vs Features Trade-off

### 📖 Simple Explanation

```
100% reliability = No new features
100% velocity = Unreliable system

Need balance!
```

---

### Decision Framework

```
Consider:
1. Current reliability (vs SLO)
2. Error budget status
3. User impact
4. Business priority

Decision Matrix:
┌─────────────────┬──────────────────┬───────────────────┐
│                 │ Budget Remaining │ Budget Exhausted  │
├─────────────────┼──────────────────┼───────────────────┤
│ High User Impact│ Deploy carefully │ Only if critical  │
│ Low User Impact │ Deploy freely    │ Defer feature     │
└─────────────────┴──────────────────┴───────────────────┘
```

---

## SRE Team Structure

### SRE vs DevOps vs Ops

```
Traditional Ops:
- Separate from development
- Manual operations
- Reactive

DevOps:
- Devs own operations
- Collaboration culture
- Shared responsibility

SRE:
- Google's implementation of DevOps
- Engineering approach to operations
- 50% cap on toil
- Error budgets
```

---

### SRE Engagement Model

```
1. Build and Run:
   - SREs build and operate service
   - Full ownership

2. Consult:
   - Dev team owns service
   - SREs provide guidance
   - SLA agreements

3. Product/Platform:
   - SREs build tools for dev teams
   - Self-service platforms
```

---

## Interview Questions

### Q1: Explain SLI, SLO, SLA
**Answer:**
```
SLI (Indicator):
- What you measure
- Example: Request success rate

SLO (Objective):
- Your target
- Example: 99.9% success rate

SLA (Agreement):
- Customer contract
- Example: 99.5% guaranteed or refund

Relationship:
SLI = measurement
SLO = internal goal (stricter)
SLA = customer promise (looser)

Always: SLA < SLO < actual performance
```

---

### Q2: What is an error budget?
**Answer:**
```
Error budget = 100% - SLO

Example:
SLO: 99.9% availability
Error budget: 0.1% = 43 minutes/month

Use budget for:
- Feature deployments
- Experiments
- Risky changes

When exhausted:
- Stop feature development
- Focus on reliability
- No non-critical deploys

Benefits:
- Quantifies reliability vs velocity
- Data-driven decisions
- Aligns dev and ops
```

---

### Q3: How do you reduce toil?
**Answer:**
```
Toil = Manual, repetitive, automatable work

Target: < 50% time on toil

Reduction strategies:
1. Automate common tasks
   - Scripts for frequent operations
   - Self-healing systems

2. Improve monitoring
   - Catch issues earlier
   - Reduce manual checks

3. Better design
   - Reduce complexity
   - Increase reliability

4. Eliminate root causes
   - Fix bugs permanently
   - Don't band-aid

Example:
Manual: Clear disk space weekly
Automated: Cron job + monitoring
```

---

### Q4: What makes a good SLO?
**Answer:**
```
Good SLO characteristics:

1. User-focused
   ✓ "95% of requests < 200ms"
   ✗ "CPU < 80%"

2. Achievable
   - Not too strict (99.999%)
   - Not too loose (95%)
   - Sweet spot: 99.9%

3. Measurable
   - Clear metrics
   - Automated collection

4. Actionable
   - Can you improve it?
   - Clear when violated

Examples:
✓ "99.9% availability"
✓ "95% of requests < 200ms"
✓ "99% of batch jobs complete in < 1 hour"
```

---

### Q5: Explain blameless postmortems
**Answer:**
```
Blameless = Focus on systems, not people

Principles:
1. No finger-pointing
2. Everyone acted reasonably with info they had
3. Focus on what, not who
4. Learn and improve

Bad: "Alice broke production"
Good: "Deploy process lacked safeguards"

Key elements:
- Timeline of events
- Root cause analysis
- Action items (with owners)
- What went well
- Lessons learned

Culture:
- Psychological safety
- Learn from failures
- Share knowledge
- Continuous improvement
```

---

## Quick Reference

```
SLO Examples:
- 99.9% availability = 43 min downtime/month
- 99.95% availability = 21.6 min/month
- 99.99% availability = 4.32 min/month

Error Budget:
Budget = 100% - SLO
Exhausted? → Focus on reliability

Toil:
Target: < 50% of time
Reduce through automation

On-Call:
- Max 25% time on-call
- Sustainable rotations
- Clear escalation

Postmortems:
- Blameless
- Focus on systems
- Actionable items
```

---

## Summary

**Key Takeaways:**
1. SRE = Software engineering for operations
2. SLI/SLO/SLA = Measure, target, promise
3. Error budgets balance reliability and velocity
4. Toil should be < 50% of time
5. Blameless postmortems for learning
6. Sustainable on-call practices
7. Automate everything

**Next Steps:**
1. Define SLOs for your services
2. Calculate error budgets
3. Measure toil percentage
4. Write postmortem template
5. Improve monitoring/alerting
6. Automate common tasks
7. Establish on-call rotation

**Remember:**
- Hope is not a strategy
- Measure everything
- Automate toil
- Learn from failures
- Balance reliability and velocity

**Happy Engineering! 🔧**

