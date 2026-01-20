# Lab M6.04 - Diagnose Degraded System and Write Root Cause Note

**Repository:** [https://github.com/cloud-engineering-bootcamp/ce-lab-diagnose-degraded-system](https://github.com/cloud-engineering-bootcamp/ce-lab-diagnose-degraded-system)

**Activity Type:** Individual  
**Estimated Time:** 60 minutes

## Learning Objectives

- [ ] Apply RED/USE troubleshooting methods systematically
- [ ] Analyze monitoring data to identify root cause
- [ ] Write professional root cause analysis (RCA) document
- [ ] Propose immediate and long-term fixes
- [ ] Document lessons learned

## Your Task

Investigate a degraded system using monitoring data, identify the root cause, and write a comprehensive RCA document.

**Scenario:** Your production web application is experiencing high latency. Users are complaining about slow page loads. Your manager has asked you to investigate and write an RCA.

**Success Criteria:**
- Systematic investigation using RED/USE methods
- Root cause identified with evidence
- Professional RCA document written
- Immediate and long-term fixes proposed

## 📤 What to Submit

**Submission Type:** GitHub Repository

Create a **public** GitHub repository named `ce-lab-system-diagnosis-rca` containing:

### Required Files

**1. README.md**
- Investigation summary
- Key findings
- Timeline of discovery

**2. RCA Document** (`rca/`)
- `root-cause-analysis.md` - Complete RCA following template
- `investigation-notes.md` - Your investigation process
- `evidence/` - Screenshots and data supporting your findings

**3. Metrics Analysis** (`analysis/`)
- `red-method-analysis.md` - RED method findings
- `use-method-analysis.md` - USE method findings
- `correlation-analysis.md` - Patterns observed

**4. Proposals** (`proposals/`)
- `immediate-fix.md` - Quick mitigation strategy
- `long-term-fix.md` - Permanent solution
- `prevention.md` - How to prevent recurrence

**5. Screenshots** (`screenshots/`)
- Dashboard showing the issue
- Metrics that led to root cause
- Before/after if you implement fix

### Repository Structure
```
ce-lab-system-diagnosis-rca/
├── README.md
├── rca/
│   ├── root-cause-analysis.md
│   ├── investigation-notes.md
│   └── evidence/
│       ├── high-latency-chart.png
│       ├── cpu-spike.png
│       └── db-connections.png
├── analysis/
│   ├── red-method-analysis.md
│   ├── use-method-analysis.md
│   └── correlation-analysis.md
├── proposals/
│   ├── immediate-fix.md
│   ├── long-term-fix.md
│   └── prevention.md
└── screenshots/
    └── ...
```

## Grading: 100 points

- Systematic investigation: 25pts
- Root cause identification: 25pts
- RCA document quality: 30pts
- Proposals (immediate & long-term): 15pts
- Lessons learned: 5pts

## Detailed Instructions

### Part 1: Create the Degraded System Scenario (10 min)

**Option A: Use Provided Scenario**

We'll simulate a database connection pool exhaustion:

1. Deploy web application
2. Set database connection pool max to 20
3. Generate high traffic (50+ concurrent requests)
4. Connection pool exhausts
5. Requests queue and timeout
6. High latency appears

**Option B: Use Your Own System**

If you have a running system, you can:
1. Identify an existing performance issue
2. Or artificially create one (memory leak, CPU spike, etc.)

### Part 2: Investigation - RED Method (15 min)

**RED Framework:**
- **R**ate: Request volume
- **E**rrors: Error count/rate
- **D**uration: Latency

**Investigation Steps:**

**1. Check Rate:**
```bash
# Get request rate from CloudWatch
aws cloudwatch get-metric-statistics \
  --namespace AWS/ApplicationELB \
  --metric-name RequestCount \
  --dimensions Name=LoadBalancer,Value=app/your-alb/... \
  --start-time 2024-01-20T14:00:00Z \
  --end-time 2024-01-20T15:00:00Z \
  --period 300 \
  --statistics Sum

# Document findings
echo "Request Rate Analysis:" > analysis/red-method-analysis.md
echo "- Normal baseline: 500 requests/min" >> analysis/red-method-analysis.md
echo "- During incident: 1,500 requests/min (3x increase)" >> analysis/red-method-analysis.md
```

**2. Check Errors:**
```bash
# Get error rate
aws cloudwatch get-metric-statistics \
  --namespace AWS/ApplicationELB \
  --metric-name HTTPCode_Target_5XX_Count \
  --dimensions Name=LoadBalancer,Value=app/your-alb/... \
  --start-time 2024-01-20T14:00:00Z \
  --end-time 2024-01-20T15:00:00Z \
  --period 300 \
  --statistics Sum

# Calculate error rate
# (5XX count / Total requests) * 100

echo "Error Rate Analysis:" >> analysis/red-method-analysis.md
echo "- Normal: 0.1% error rate" >> analysis/red-method-analysis.md
echo "- During incident: 5.0% error rate (50x increase)" >> analysis/red-method-analysis.md
```

**3. Check Duration (Latency):**
```bash
# Get P95 latency
aws cloudwatch get-metric-statistics \
  --namespace AWS/ApplicationELB \
  --metric-name TargetResponseTime \
  --dimensions Name=LoadBalancer,Value=app/your-alb/... \
  --start-time 2024-01-20T14:00:00Z \
  --end-time 2024-01-20T15:00:00Z \
  --period 300 \
  --statistics p95

echo "Duration Analysis:" >> analysis/red-method-analysis.md
echo "- Normal P95: 300ms" >> analysis/red-method-analysis.md
echo "- During incident: 5,000ms (16x increase)" >> analysis/red-method-analysis.md
```

**RED Method Summary:**
```markdown
# RED Method Analysis

## Rate (Traffic)
- Normal: 500 req/min
- Incident: 1,500 req/min
- **Finding: 3x traffic increase**

## Errors
- Normal: 0.1%
- Incident: 5.0%
- **Finding: 50x error increase**

## Duration (Latency)
- Normal P95: 300ms
- Incident P95: 5,000ms
- **Finding: 16x latency increase**

## Conclusion
All three RED signals elevated. Significant performance degradation.
Primary symptom: High latency (16x increase)
```

### Part 3: Investigation - USE Method (15 min)

**USE Framework (for resources):**
- **U**tilization: % busy
- **S**aturation: Queued work
- **E**rrors: Error events

**Check EC2 Resources:**

```bash
# CPU Utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-xxxxx \
  --start-time 2024-01-20T14:00:00Z \
  --end-time 2024-01-20T15:00:00Z \
  --period 300 \
  --statistics Average,Maximum

echo "# USE Method Analysis" > analysis/use-method-analysis.md
echo "" >> analysis/use-method-analysis.md
echo "## CPU" >> analysis/use-method-analysis.md
echo "- Utilization: 45% average (normal)" >> analysis/use-method-analysis.md
echo "- Saturation: Load average 2.0 (normal)" >> analysis/use-method-analysis.md
echo "- Errors: None" >> analysis/use-method-analysis.md
echo "- **Status: OK**" >> analysis/use-method-analysis.md
```

**Check Memory:**
```bash
# Memory utilization
aws cloudwatch get-metric-statistics \
  --namespace CWAgent \
  --metric-name mem_used_percent \
  --dimensions Name=InstanceId,Value=i-xxxxx \
  --start-time 2024-01-20T14:00:00Z \
  --end-time 2024-01-20T15:00:00Z \
  --period 300 \
  --statistics Average

echo "" >> analysis/use-method-analysis.md
echo "## Memory" >> analysis/use-method-analysis.md
echo "- Utilization: 70% (normal)" >> analysis/use-method-analysis.md
echo "- Saturation: No swap usage" >> analysis/use-method-analysis.md
echo "- Errors: None" >> analysis/use-method-analysis.md
echo "- **Status: OK**" >> analysis/use-method-analysis.md
```

**Check Database Connections (Custom Metric):**
```bash
# Database connection pool usage
aws cloudwatch get-metric-statistics \
  --namespace Application \
  --metric-name DBConnectionPoolUsage \
  --start-time 2024-01-20T14:00:00Z \
  --end-time 2024-01-20T15:00:00Z \
  --period 300 \
  --statistics Average,Maximum

echo "" >> analysis/use-method-analysis.md
echo "## Database Connection Pool" >> analysis/use-method-analysis.md
echo "- Utilization: 100% (max 20 connections)" >> analysis/use-method-analysis.md
echo "- Saturation: Requests queuing for connections" >> analysis/use-method-analysis.md
echo "- Errors: Connection timeout errors in logs" >> analysis/use-method-analysis.md
echo "- **Status: ⚠️ EXHAUSTED**" >> analysis/use-method-analysis.md
```

**USE Method Summary:**
```markdown
# USE Method Summary

## CPU
- ✅ Utilization: 45% (normal)
- ✅ Saturation: Normal
- ✅ Errors: None

## Memory
- ✅ Utilization: 70% (normal)
- ✅ Saturation: No swapping
- ✅ Errors: None

## Database Connection Pool
- 🔴 Utilization: 100% (maxed out!)
- 🔴 Saturation: Requests queuing
- 🔴 Errors: Timeout errors

## Conclusion
**ROOT CAUSE IDENTIFIED: Database connection pool exhaustion**
```

### Part 4: Write RCA Document (15 min)

**Use This Template:**

```markdown
# Root Cause Analysis: High Latency Incident

## Executive Summary

**Incident:** API latency increased from 300ms (P95) to 5,000ms (P95)  
**Impact:** 5,000 users experienced slow page loads, 5% error rate  
**Duration:** 60 minutes (14:00-15:00 UTC, Jan 20, 2024)  
**Root Cause:** Database connection pool exhaustion  
**Status:** Resolved

## Timeline

| Time (UTC) | Event |
|------------|-------|
| 14:00 | CloudWatch alarm triggered: High latency detected |
| 14:05 | On-call engineer paged |
| 14:10 | Investigation started using RED method |
| 14:15 | Confirmed elevated error rate and latency |
| 14:20 | USE method applied to resources |
| 14:25 | Identified DB connection pool at 100% utilization |
| 14:30 | ROOT CAUSE: Connection pool exhausted (max 20) |
| 14:35 | Immediate fix: Increased pool size to 50 |
| 14:40 | Service began recovering |
| 14:50 | Latency returned to normal |
| 15:00 | Incident closed |

## Root Cause

**Problem:** Database connection pool exhausted

**Why it happened:**
1. Marketing campaign launched without engineering notification
2. Traffic increased 3x (500 → 1,500 requests/min)
3. Connection pool sized for normal load (20 connections)
4. No auto-scaling configured for connection pool
5. Requests queued waiting for connections
6. Timeouts and errors increased

**Evidence:**
- CloudWatch metric: DBConnectionPoolUsage at 100%
- Application logs: "connection timeout" errors
- Traffic spike correlates with latency spike
- CPU and memory remained normal (not a resource issue)

## Impact

**Users Affected:** ~5,000 users
**Duration:** 60 minutes
**Revenue Impact:** Estimated $2,500 (based on average order value)
**Error Rate:** 5% (250 failed requests)

**Severity:** P1 (Critical) - Service degraded but not completely down

## Resolution

### Immediate Fix (14:35)
```python
# Increased database connection pool
DB_CONNECTION_POOL_SIZE = 50  # Was: 20
```

### Verification
- Latency P95 dropped from 5,000ms to 400ms within 5 minutes
- Error rate decreased from 5% to 0.2%
- Connection pool utilization: 60% (healthy headroom)

## Lessons Learned

### What Went Well ✅
- Alerts triggered immediately (within 1 minute)
- On-call engineer responded quickly (5 minutes)
- Systematic RED/USE methodology led to root cause
- Clear monitoring data available
- Mitigation applied quickly (10 minutes to identify, 5 to fix)

### What Went Wrong ❌
- No capacity planning for marketing campaigns
- Connection pool not monitored (no alerts)
- No auto-scaling configured
- Lack of communication between marketing and engineering
- No load testing performed

## Action Items

### Immediate (Week 1)
- [x] Increase connection pool to 50 (DONE)
- [ ] Add CloudWatch alarm for connection pool > 80% (Owner: Sarah, Due: Jan 25)
- [ ] Document connection pool sizing (Owner: Mike, Due: Jan 22)

### Short-term (Month 1)
- [ ] Implement auto-scaling for connection pool (Owner: Mike, Due: Feb 5)
- [ ] Load test with 5x normal traffic (Owner: Team, Due: Feb 10)
- [ ] Create runbook for connection pool issues (Owner: Alex, Due: Jan 30)

### Long-term (Quarter 1)
- [ ] Establish engineering review process for marketing campaigns (Owner: Manager, Due: Feb 15)
- [ ] Implement capacity planning framework (Owner: Team, Due: Mar 1)
- [ ] Add auto-scaling dashboard widget (Owner: Sarah, Due: Feb 5)

## Prevention

**How to prevent recurrence:**

1. **Monitoring:** Alert on connection pool > 80%
2. **Auto-scaling:** Dynamically adjust pool size based on traffic
3. **Communication:** Require engineering sign-off on campaigns
4. **Load Testing:** Test at 3-5x normal traffic before campaigns
5. **Capacity Planning:** Quarterly capacity review meetings

## References

- CloudWatch Dashboard: [Link]
- Incident Slack Thread: [Link]
- Runbook: [Link]
- Related Incidents: None
```

### Part 5: Propose Fixes (10 min)

**Immediate Fix:**
```markdown
# Immediate Fix Proposal

## Problem
Database connection pool exhausted (20 connections)

## Solution
Increase connection pool size to 50

## Implementation
```python
# config/database.py
DB_CONNECTION_POOL_SIZE = 50  # Increased from 20
DB_CONNECTION_POOL_MAX_OVERFLOW = 10
DB_CONNECTION_TIMEOUT = 30
```

## Verification
- Monitor connection pool utilization
- Should be < 80% during peak traffic
- Test with load test

## Risk
Low - More connections = slightly more memory usage

## Rollback
Reduce back to 20 if memory issues appear
```

**Long-term Fix:**
```markdown
# Long-term Fix Proposal

## Problem
Fixed connection pool doesn't adapt to traffic

## Solution
Implement dynamic connection pool scaling

## Implementation
```python
# Auto-scaling connection pool
class DynamicConnectionPool:
    def __init__(self):
        self.min_connections = 20
        self.max_connections = 100
        self.current_connections = 20
    
    def scale_based_on_traffic(self, current_traffic):
        target = current_traffic / 20  # 20 requests per connection
        target = max(self.min_connections, min(target, self.max_connections))
        
        if target > self.current_connections:
            self.expand_pool(target)
        elif target < self.current_connections * 0.7:
            self.shrink_pool(target)
```

## Benefits
- Handles traffic spikes automatically
- Reduces cost during low traffic
- No manual intervention needed

## Monitoring
- Add CloudWatch metric: ConnectionPoolSize
- Alert if auto-scaling fails

## Testing Plan
1. Load test with varying traffic (100-2000 req/min)
2. Verify pool scales up/down correctly
3. Ensure no connection leaks
```

### Part 6: Document Lessons Learned (5 min)

```markdown
# Lessons Learned

## Technical Lessons

1. **Monitor Resource Pools:** Not just CPU/memory, but connection pools, thread pools, etc.
2. **Use RED/USE Methods:** Systematic frameworks lead to faster root cause identification
3. **Correlation is Key:** Traffic spike + latency spike + connection pool exhaustion = clear pattern

## Process Lessons

1. **Communication:** Marketing and engineering must coordinate on campaigns
2. **Capacity Planning:** Required for any significant traffic change
3. **Load Testing:** Test at 3-5x normal traffic, not just 2x

## Cultural Lessons

1. **Blameless RCAs:** Focus on systems, not individuals
2. **Document Everything:** Future you will thank current you
3. **Action Items Matter:** Track to completion, not just create

## What Would I Do Differently?

1. Set up connection pool monitoring earlier
2. Request notification of upcoming campaigns
3. Implement auto-scaling proactively
```

## Reflection Questions

Answer these in your README:

1. **Why use RED method first, then USE method?**
   - Hint: Services vs resources

2. **How did correlation help identify root cause?**
   - Hint: Traffic spike + connection pool exhaustion

3. **What's the difference between immediate and long-term fixes?**
   - Hint: Quick mitigation vs permanent solution

4. **Why write RCA even after issue is resolved?**
   - Hint: Learn and prevent recurrence

5. **How do you determine severity (P0, P1, P2)?**
   - Hint: Impact to users and revenue

## Bonus Challenges

**+5 points each:**
- [ ] Create a runbook for this type of incident
- [ ] Implement the immediate fix and show results
- [ ] Design monitoring dashboard for database metrics
- [ ] Write load testing plan
- [ ] Create capacity planning spreadsheet

## Resources

- [Google SRE Book - Postmortems](https://sre.google/sre-book/postmortem-culture/)
- [Atlassian Incident Postmortem Template](https://www.atlassian.com/incident-management/postmortem)
- [AWS Troubleshooting Guide](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/troubleshooting.html)

---

**Excellent investigation and RCA work!** 🔍
