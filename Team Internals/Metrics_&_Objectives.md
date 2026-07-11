## SOC Performance Metrics

### Core SOC Goal
Protect CIA (Confidentiality, Integrity, Availability) by reliably detecting and reporting True Positives to L2.

---

### Key Metrics

| Metric | Formula | Measures |
|--------|---------|---------|
| **Alerts Count** | AC = Total alerts received | Overall analyst workload |
| **False Positive Rate** | FPR = False Positives / Total Alerts | Noise level in detection |
| **Alert Escalation Rate** | AER = Escalated Alerts / Total Alerts | L1 analyst experience/independence |
| **Threat Detection Rate** | TDR = Detected Threats / Total Threats | SOC reliability |

---

### Alerts Count

**Ideal range:** 5–30 alerts/day per L1 analyst

| Scenario | Problem |
|----------|---------|
| 80+ alerts/shift | Overwhelming — real threats hide in noise |
| 0 alerts for a week | Concerning — SIEM may be broken or visibility gap exists |

---

### False Positive Rate

**Target:** As low as possible
**Problem threshold:** 80%+ = serious issue

```
75 FP out of 80 alerts = 94% FPR → analysts become desensitized
"Yet another spam" mindset → real threats get missed
```

**Fix:** Detection rule tuning — "False Positive Remediation"

---

### Alert Escalation Rate

**Target:** Below 50% (ideally below 20%)

| Rate | Interpretation |
|------|---------------|
| Very high (>50%) | L1 lacks confidence or experience — escalating too much |
| Very low (<5%) | L1 may be overconfident — closing alerts they shouldn't |
| ~10-20% | Healthy — L1 handles most, escalates genuine threats |

---

### Threat Detection Rate

**Target:** 100% — no exceptions

```
Example:
6 attacks in 2025
4 detected + stopped     ✓
1 missed — broken rule   ✗
1 missed — L1 misclassified as FP ✗

TDR = 4/6 = 67% → UNACCEPTABLE
```

**Why 100% is the only acceptable target:**
- Every missed threat = potential ransomware, data breach, or worse
- Unlike FPR or AER, there's no "acceptable" miss rate for real attacks

---

### Metrics Summary

| Metric | Good | Bad | Fix |
|--------|------|-----|-----|
| Alerts Count | 5-30/day | <5 or >80 | Tune rules / check SIEM coverage |
| FPR | <30% | >80% | FP remediation / rule tuning |
| AER | <20% | >50% | L1 training + workbook improvement |
| TDR | 100% | Anything less | Rule fixes + analyst retraining |

---

## SOC SLA Metrics — MTTD, MTTA, MTTR

### What is an SLA?
Service Level Agreement — formal document defining required SOC performance standards.
Signed between: SOC team ↔ company management, or MSSP ↔ customer.

---

### Threat Timeline

<img width="1222" height="377" alt="1" src="https://github.com/user-attachments/assets/6dd16818-ddb8-4533-a81b-64fbdfc8373c" />

```
Attack happens
    ↓ (MTTD measured here)
SOC tool detects → alert generated
    ↓ (MTTA measured here)
L1 analyst acknowledges → starts triage
    ↓ (MTTR measured here)
Threat contained → device isolated / account secured
```

---

### SLA Metrics Reference

| Metric | Common SLA Target | What it measures |
|--------|------------------|-----------------|
| **SOC Availability** | 24/7 (or 8/5) | Working schedule of the SOC team |
| **MTTD** (Mean Time to Detect) | 5 minutes | Time between attack occurring and SOC tool detecting it |
| **MTTA** (Mean Time to Acknowledge) | 10 minutes | Time for L1 to pick up and start triaging the alert |
| **MTTR** (Mean Time to Respond) | 60 minutes | Time for SOC to actually stop the breach from spreading |

---

### Why Each Metric Matters

| Metric | If too high | Consequence |
|--------|------------|-------------|
| MTTD | Detection rules are weak or SIEM has gaps | Attacker operates undetected longer |
| MTTA | Analysts are slow or understaffed | Alert sits in queue while attack progresses |
| MTTR | Response procedures are slow or unclear | Breach spreads — more systems compromised |

---

### Key Insight
```
Attack to Detection (MTTD):     5 min target
Detection to Acknowledgment:   10 min target
Acknowledgment to Response:    60 min target

Total acceptable breach window: ~75 minutes
```

---

## Improving SOC Metrics — L1 Recommendations

### Why Metrics Matter for L1
- Make SOC more efficient → attacks less successful
- Used to evaluate your personal performance
- Good metrics → career growth → promotion to L2

---

### Fixes by Metric

#### False Positive Rate > 80%
Too much noise — analysts become desensitized.
```
1. Exclude known trusted activity from detection rules
   (e.g., system updates, scheduled IT tasks, known admin tools)
2. Automate triage for most common/repetitive FP alerts
   using SOAR or custom scripts
```

#### MTTD > 30 minutes
Threats detected too slowly.
```
1. Ask SOC engineers to optimize detection rules
   (run faster, higher frequency checks)
2. Verify SIEM log collection is real-time
   (a 10-min ingestion delay = 10-min blind spot)
```

#### MTTA > 30 minutes
Analysts slow to pick up alerts.
```
1. Ensure real-time notifications when new alerts arrive
   (don't rely on manual dashboard refresh)
2. Evenly distribute alert queue across all analysts on shift
   (no single analyst buried while others are idle)
```

#### MTTR > 4 hours
Breach not contained in time.
```
1. As L1: escalate True Positives to L2 as fast as possible
   (don't over-investigate what's above your scope)
2. Ensure documented response procedures exist for common scenarios
   (no improvising during an active breach)
```

---

### Quick Reference

| Metric | Bad Threshold | L1 Fix |
|--------|--------------|--------|
| FPR | > 80% | Rule tuning + SOAR automation |
| MTTD | > 30 min | Rule optimization + real-time log ingestion |
| MTTA | > 30 min | Real-time alerts + balanced queue distribution |
| MTTR | > 4 hours | Fast escalation + documented response playbooks |

---

### Career Takeaway
> As L1, you can't fix SIEM architecture or rewrite detection rules alone.
> But you CAN escalate fast, triage accurately, reduce your personal MTTA,
> and flag FP patterns to engineers.
> Your individual metrics are visible. Make them reflect your best work.
