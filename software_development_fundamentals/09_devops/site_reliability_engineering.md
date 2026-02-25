# Site Reliability Engineering

## Introduction

Site Reliability Engineering (SRE) applies software engineering principles to
operations problems. Instead of treating operations as a separate discipline,
SRE treats infrastructure and reliability as software problems that can be
solved through automation, measurement, and engineering rigor.

```
  ┌──────────────────────────────────────────────────────────┐
  │                    SRE PRINCIPLES                         │
  │                                                           │
  │  1. Embrace risk         Perfection is the enemy of       │
  │     quantitatively       velocity. Use error budgets.     │
  │                                                           │
  │  2. Eliminate toil       Automate repetitive operational  │
  │                          work to free engineering time.   │
  │                                                           │
  │  3. Monitor everything   If you cannot measure it, you    │
  │                          cannot improve it.               │
  │                                                           │
  │  4. Automate responses   Humans should solve novel        │
  │                          problems, not routine ones.      │
  │                                                           │
  │  5. Release engineering  Make releases boring through     │
  │                          automation and small batches.    │
  │                                                           │
  │  6. Simplicity           A system is reliable only if     │
  │                          it is understandable.            │
  └──────────────────────────────────────────────────────────┘
```

---

## Error Budgets

An error budget quantifies how much unreliability a service can tolerate. It
creates a shared framework for balancing reliability and feature velocity.

```
  ┌──────────────────────────────────────────────────────┐
  │             ERROR BUDGET LIFECYCLE                    │
  │                                                       │
  │  SLO: 99.9% availability (30-day window)             │
  │  Budget: 0.1% = 43.2 minutes of downtime             │
  │                                                       │
  │  Budget Healthy (> 50% remaining)                     │
  │  ├── Ship features freely                             │
  │  ├── Run experiments                                  │
  │  └── Accept moderate risk                             │
  │                                                       │
  │  Budget Caution (25-50% remaining)                    │
  │  ├── Require extra review for changes                 │
  │  ├── Prioritize reliability work                      │
  │  └── Reduce batch sizes                               │
  │                                                       │
  │  Budget Critical (< 25% remaining)                    │
  │  ├── Freeze non-critical deployments                  │
  │  ├── All engineering focus on reliability              │
  │  └── Postmortems for all incidents                    │
  │                                                       │
  │  Budget Exhausted (0% remaining)                      │
  │  ├── Full deployment freeze                           │
  │  ├── Only reliability fixes permitted                 │
  │  └── Leadership review and escalation                 │
  └──────────────────────────────────────────────────────┘
```

### Error Budget Pseudocode

```
FUNCTION manage_error_budget(slo_target, window_days, current_failures_minutes)
  SET total_minutes = window_days * 24 * 60
  SET budget_total = total_minutes * (1 - slo_target)
  SET budget_remaining = budget_total - current_failures_minutes
  SET budget_percentage = (budget_remaining / budget_total) * 100

  SET policy = {}

  IF budget_percentage > 50 THEN
    SET policy.deployment_status = "OPEN"
    SET policy.risk_tolerance = "NORMAL"
    SET policy.review_required = FALSE
    SET policy.message = "Error budget healthy -- proceed normally"

  ELSE IF budget_percentage > 25 THEN
    SET policy.deployment_status = "CAUTIOUS"
    SET policy.risk_tolerance = "LOW"
    SET policy.review_required = TRUE
    SET policy.message = "Error budget below 50% -- extra review required"

  ELSE IF budget_percentage > 0 THEN
    SET policy.deployment_status = "FROZEN"
    SET policy.risk_tolerance = "MINIMAL"
    SET policy.review_required = TRUE
    SET policy.allowed_changes = ["reliability_fixes", "rollbacks"]
    SET policy.message = "Error budget critical -- non-critical deploys frozen"

  ELSE
    SET policy.deployment_status = "EMERGENCY"
    SET policy.risk_tolerance = "NONE"
    SET policy.allowed_changes = ["reliability_fixes"]
    SET policy.message = "Error budget exhausted -- emergency mode"
    NOTIFY leadership WITH policy.message
  END IF

  LOG "Error budget: {budget_remaining}/{budget_total} min ({budget_percentage}%)"
  RETURN policy
END FUNCTION
```

---

## Toil Identification and Reduction

Toil is the kind of work tied to running a production service that is manual,
repetitive, automatable, tactical, lacking enduring value, and grows linearly
with service size.

```
  ┌──────────────────────────────────────────────────────┐
  │                TOIL vs ENGINEERING                    │
  │                                                       │
  │  TOIL (reduce this)         ENGINEERING (increase)    │
  │  ─────────────────          ───────────────────────   │
  │  Manual restarts            Self-healing automation   │
  │  Ticket-driven scaling      Autoscaling policies      │
  │  Copy-paste deployments     Pipeline automation       │
  │  Hand-editing configs       Configuration management  │
  │  Reading logs manually      Automated alerting        │
  │  Running reports by hand    Scheduled report jobs     │
  │                                                       │
  │  GOAL: Keep toil below 50% of an SRE team's time     │
  │        Invest the rest in engineering that reduces     │
  │        future toil.                                   │
  └──────────────────────────────────────────────────────┘
```

### Toil Assessment Pseudocode

```
FUNCTION assess_toil(tasks_log, period)
  SET toil_tasks = EMPTY LIST
  SET engineering_tasks = EMPTY LIST

  FOR EACH task IN tasks_log WHERE task.date WITHIN period DO
    SET is_toil = TRUE

    IF task.requires_creativity THEN SET is_toil = FALSE END IF
    IF task.has_lasting_value THEN SET is_toil = FALSE END IF
    IF task.reduces_future_work THEN SET is_toil = FALSE END IF
    IF NOT task.is_repetitive THEN SET is_toil = FALSE END IF

    IF is_toil THEN
      APPEND toil_tasks WITH task
    ELSE
      APPEND engineering_tasks WITH task
    END IF
  END FOR

  SET toil_hours = SUM(toil_tasks, field = "hours")
  SET total_hours = SUM(tasks_log, field = "hours")
  SET toil_ratio = toil_hours / total_hours

  RETURN {
    toil_ratio: toil_ratio,
    toil_hours: toil_hours,
    top_toil_categories: GROUP_AND_RANK(toil_tasks, by = "category"),
    recommendation: IF toil_ratio > 0.5
      THEN "Toil exceeds 50% -- prioritize automation"
      ELSE "Toil within acceptable range"
  }
END FUNCTION
```

---

## Incident Management Lifecycle

```
  ┌────────────────────────────────────────────────────────────────┐
  │              INCIDENT MANAGEMENT LIFECYCLE                      │
  │                                                                 │
  │  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐  │
  │  │ DETECT   │──▶│ RESPOND  │──▶│ MITIGATE │──▶│ RESOLVE  │  │
  │  └──────────┘   └──────────┘   └──────────┘   └──────────┘  │
  │       │              │              │              │           │
  │  Alert fires    Page on-call    Stop bleeding    Fix root     │
  │  or user        Assemble team   Restore service  cause        │
  │  reports        Declare severity                              │
  │                                                                │
  │                          │                                     │
  │                          ▼                                     │
  │                   ┌──────────┐                                 │
  │                   │POSTMORTEM│                                 │
  │                   └──────────┘                                 │
  │                   Document,                                    │
  │                   learn,                                       │
  │                   prevent                                      │
  │                   recurrence                                   │
  │                                                                │
  │  SEVERITY LEVELS:                                              │
  │  ┌──────┬───────────────────────┬─────────────────────┐       │
  │  │ Sev  │ Impact                │ Response            │       │
  │  ├──────┼───────────────────────┼─────────────────────┤       │
  │  │  1   │ Complete outage       │ All hands, exec     │       │
  │  │  2   │ Major feature broken  │ On-call + backups   │       │
  │  │  3   │ Minor degradation     │ On-call only        │       │
  │  │  4   │ Cosmetic / low impact │ Next business day   │       │
  │  └──────┴───────────────────────┴─────────────────────┘       │
  └────────────────────────────────────────────────────────────────┘
```

### Incident Response Pseudocode

```
FUNCTION handle_incident(alert)
  // DETECT
  SET incident = CREATE_INCIDENT(
    title = alert.message,
    severity = classify_severity(alert),
    start_time = CURRENT_TIME(),
    status = "OPEN"
  )

  // RESPOND
  SET on_call = GET_ON_CALL_ENGINEER(alert.service)
  NOTIFY on_call VIA pager WITH incident.summary

  IF incident.severity <= 2 THEN
    CREATE communication_channel FOR incident
    ASSIGN incident_commander
    NOTIFY stakeholders WITH initial_status
    BEGIN status_updates EVERY 15 MINUTES
  END IF

  // MITIGATE
  LOG_TIMELINE incident "Investigation started by {on_call}"

  SET runbook = FIND_RUNBOOK(alert.type)
  IF runbook IS FOUND THEN
    EXECUTE runbook STEPS WITH logging
  END IF

  // Resolution is manual from here -- engineer investigates
  RETURN incident
END FUNCTION

FUNCTION close_incident(incident, resolution_notes)
  SET incident.status = "RESOLVED"
  SET incident.end_time = CURRENT_TIME()
  SET incident.duration = incident.end_time - incident.start_time
  SET incident.resolution = resolution_notes

  NOTIFY stakeholders WITH final_status
  SCHEDULE postmortem FOR incident WITHIN 48 HOURS

  UPDATE error_budget WITH incident.duration
  RETURN incident
END FUNCTION
```

---

## On-Call Practices

```
  ┌────────────────────────────────────────────────────────┐
  │              ON-CALL BEST PRACTICES                     │
  │                                                         │
  │  ROTATION                                               │
  │  ├── Minimum 2 people in rotation                       │
  │  ├── Primary and secondary on-call                      │
  │  ├── Rotate weekly (or bi-weekly for small teams)       │
  │  ├── No more than 25% of time on-call per person        │
  │  └── Handoff includes status of ongoing issues          │
  │                                                         │
  │  ALERTING                                               │
  │  ├── Only page for actionable, user-impacting issues    │
  │  ├── Include runbook link in every alert                 │
  │  ├── Escalation path: primary → secondary → manager     │
  │  ├── Auto-escalate after 15 minutes without ack         │
  │  └── Track pages per shift and alert-to-noise ratio     │
  │                                                         │
  │  SUSTAINABILITY                                         │
  │  ├── Compensate on-call time (time off or pay)          │
  │  ├── Postmortem action items reduce future pages         │
  │  ├── Target: ≤ 2 pages per 12-hour shift               │
  │  └── Review on-call load monthly                        │
  └────────────────────────────────────────────────────────┘
```

---

## Blameless Postmortem Template

```
POSTMORTEM TEMPLATE
═══════════════════

TITLE:        [Brief description of the incident]
DATE:         [When the incident occurred]
SEVERITY:     [1-4]
DURATION:     [Total time from detection to resolution]
AUTHOR:       [Person writing the postmortem]
PARTICIPANTS: [People involved in the response]

─── SUMMARY ───
[2-3 sentence description of what happened and the impact]

─── TIMELINE ───
[Chronological list of events]
  HH:MM  Alert fired: [description]
  HH:MM  On-call acknowledged
  HH:MM  [Investigation step]
  HH:MM  [Mitigation applied]
  HH:MM  [Service restored]
  HH:MM  Incident closed

─── IMPACT ───
  Users affected:     [number or percentage]
  Duration of impact: [minutes]
  Revenue impact:     [if applicable]
  Error budget consumed: [minutes / percentage]

─── ROOT CAUSE ───
[Detailed technical explanation. No blame on individuals.]

─── CONTRIBUTING FACTORS ───
[Other conditions that made the incident more likely or severe]
  - [Factor 1]
  - [Factor 2]

─── WHAT WENT WELL ───
  - [Detection was fast because...]
  - [Runbook was accurate for...]

─── WHAT WENT POORLY ───
  - [Alert was noisy because...]
  - [Rollback took too long because...]

─── ACTION ITEMS ───
  [ID] [Description]              [Owner]   [Priority] [Due Date]
  AI-1 [Specific remediation]     [person]  P1         [date]
  AI-2 [Preventive measure]       [person]  P2         [date]
  AI-3 [Detection improvement]    [person]  P2         [date]

─── LESSONS LEARNED ───
[Key takeaways for the broader organization]
```

---

## Chaos Engineering

Chaos engineering is the discipline of experimenting on a system to build
confidence in its ability to withstand turbulent conditions in production.

```
  ┌────────────────────────────────────────────────────────┐
  │            CHAOS ENGINEERING PROCESS                    │
  │                                                         │
  │  1. HYPOTHESIZE                                         │
  │     "We believe the system will [behavior]              │
  │      when [condition]"                                  │
  │                                                         │
  │  2. DEFINE STEADY STATE                                 │
  │     Identify metrics that indicate normal behavior      │
  │     (throughput, error rate, latency)                   │
  │                                                         │
  │  3. INTRODUCE FAILURE                                   │
  │     ┌─────────────────────────────────────────┐        │
  │     │  Common experiments:                     │        │
  │     │  - Kill a service instance               │        │
  │     │  - Introduce network latency             │        │
  │     │  - Fill a disk                           │        │
  │     │  - Exhaust CPU on a host                 │        │
  │     │  - Block network between services        │        │
  │     │  - Corrupt a cache                       │        │
  │     └─────────────────────────────────────────┘        │
  │                                                         │
  │  4. OBSERVE                                             │
  │     Compare actual behavior against hypothesis          │
  │                                                         │
  │  5. LEARN AND FIX                                       │
  │     If the system did not behave as expected,           │
  │     improve resilience and repeat                       │
  └────────────────────────────────────────────────────────┘
```

### Chaos Experiment Pseudocode

```
FUNCTION run_chaos_experiment(experiment)
  // Record steady state
  SET baseline = CAPTURE_METRICS(
    metrics = experiment.steady_state_metrics,
    duration = experiment.baseline_window
  )

  LOG "Baseline captured: {baseline}"

  // Safety check
  IF NOT system_healthy(baseline) THEN
    LOG "System not in steady state -- aborting experiment"
    RETURN {status: "ABORTED", reason: "unhealthy_baseline"}
  END IF

  // Inject failure
  LOG "Injecting failure: {experiment.fault_type}"
  SET fault = INJECT_FAULT(experiment.fault_type, experiment.parameters)

  // Observe
  SET observation = CAPTURE_METRICS(
    metrics = experiment.steady_state_metrics,
    duration = experiment.observation_window
  )

  // Compare
  SET deviation = COMPARE(baseline, observation)

  // Automatic abort if impact exceeds safety threshold
  IF deviation.error_rate > experiment.abort_threshold THEN
    REMOVE_FAULT(fault)
    LOG "Experiment aborted -- safety threshold exceeded"
    RETURN {status: "ABORTED", reason: "threshold_exceeded", data: deviation}
  END IF

  // Clean up
  REMOVE_FAULT(fault)
  WAIT UNTIL system_healthy(experiment.steady_state_metrics)

  // Analyze
  SET result = {
    status: "COMPLETED",
    hypothesis_confirmed: deviation WITHIN experiment.tolerance,
    baseline: baseline,
    observation: observation,
    deviation: deviation
  }

  LOG "Experiment complete: hypothesis {result.hypothesis_confirmed}"
  RETURN result
END FUNCTION
```

---

## Capacity Planning

```
  ┌──────────────────────────────────────────────────────────┐
  │              CAPACITY PLANNING PROCESS                    │
  │                                                           │
  │  ┌───────────┐   ┌───────────┐   ┌───────────┐          │
  │  │  MEASURE  │──▶│  MODEL    │──▶│   PLAN    │          │
  │  │           │   │           │   │           │          │
  │  │ Current   │   │ Growth    │   │ Provision │          │
  │  │ usage &   │   │ forecast  │   │ ahead of  │          │
  │  │ headroom  │   │ & trend   │   │ demand    │          │
  │  └───────────┘   └───────────┘   └───────────┘          │
  │                                                           │
  │  KEY METRICS:                                             │
  │  - CPU utilization (target: < 70% sustained)              │
  │  - Memory usage (target: < 80%)                           │
  │  - Storage growth rate                                    │
  │  - Network bandwidth utilization                          │
  │  - Request throughput vs capacity ceiling                  │
  └──────────────────────────────────────────────────────────┘
```

```
FUNCTION forecast_capacity(historical_usage, forecast_months)
  SET growth_rate = CALCULATE_TREND(historical_usage, method = "linear_regression")
  SET current_usage = LATEST(historical_usage)
  SET current_capacity = GET_PROVISIONED_CAPACITY()

  SET forecasted_usage = current_usage * (1 + growth_rate) ^ forecast_months
  SET headroom = current_capacity - forecasted_usage
  SET months_until_full = LOG(current_capacity / current_usage) / LOG(1 + growth_rate)

  IF months_until_full < 3 THEN
    ALERT capacity_team WITH "Less than 3 months of capacity remaining"
  END IF

  RETURN {
    current_usage: current_usage,
    current_capacity: current_capacity,
    growth_rate_monthly: growth_rate,
    forecasted_usage: forecasted_usage,
    headroom: headroom,
    months_until_full: months_until_full
  }
END FUNCTION
```

---

## Runbooks

A runbook is a step-by-step procedure for diagnosing and resolving a specific
type of operational issue. Good runbooks are specific, testable, and
maintained alongside the systems they describe.

### Runbook Pseudocode

```
RUNBOOK "high_error_rate"
  TRIGGER: alert "error_rate_above_threshold"
  SEVERITY: 2
  ESTIMATED_TIME: 15 minutes

  STEP 1 "Assess scope"
    RUN query "error_rate BY endpoint LAST 15 MINUTES"
    IF errors concentrated ON single endpoint THEN
      GOTO STEP 3  // Likely application bug
    ELSE IF errors spread ACROSS all endpoints THEN
      GOTO STEP 2  // Likely infrastructure issue
    END IF
  END STEP

  STEP 2 "Check infrastructure"
    RUN health_check ON database
    RUN health_check ON cache_layer
    RUN health_check ON upstream_dependencies

    IF database.health == UNHEALTHY THEN
      EXECUTE runbook "database_recovery"
      STOP
    END IF

    IF cache_layer.health == UNHEALTHY THEN
      RUN cache_restart WITH graceful = TRUE
      WAIT 2 MINUTES
      RUN query "error_rate LAST 2 MINUTES"
      IF error_rate < threshold THEN
        LOG "Resolved by cache restart"
        STOP
      END IF
    END IF

    ESCALATE TO secondary_on_call WITH context
  END STEP

  STEP 3 "Check recent deployments"
    SET last_deploy = GET_LAST_DEPLOYMENT()
    IF last_deploy.time WITHIN LAST 60 MINUTES THEN
      LOG "Recent deployment detected: {last_deploy.version}"
      CONFIRM WITH deployer "Should we rollback?"
      IF confirmed THEN
        EXECUTE rollback TO last_deploy.previous_version
        VERIFY error_rate RETURNS TO normal WITHIN 5 MINUTES
      END IF
    ELSE
      LOG "No recent deployments -- investigate application logs"
      RUN query "error_logs BY exception_type LAST 15 MINUTES"
      ESCALATE WITH log analysis TO development_team
    END IF
  END STEP
END RUNBOOK
```

---

## Practice Problems

1. **Error Budget Calculation:** A service has an SLO of 99.95% availability
   over a 30-day window. It has experienced two incidents: one lasting 8
   minutes and another lasting 12 minutes. Calculate the total error budget,
   the amount consumed, the remaining budget, and whether the team should
   modify their deployment practices.

2. **Toil Analysis:** Categorize the following tasks as toil or engineering.
   For each toil item, propose an automation that would eliminate it:
   a. Restarting a service every Monday morning because of a memory leak
   b. Writing a script that automatically restarts the service when memory
      exceeds a threshold
   c. Manually updating configuration files across 20 servers
   d. Reviewing and merging pull requests

3. **Incident Timeline:** Write a complete incident timeline for the following
   scenario: A deployment at 14:00 introduces a database query that runs 100x
   slower than intended. Error rates climb gradually. An alert fires at 14:22.
   The on-call investigates, identifies the query at 14:45, and rolls back the
   deployment at 14:52. Service recovers by 14:58.

4. **Chaos Experiment Design:** Design a chaos experiment to test whether your
   system can handle the loss of an entire availability zone (data center).
   Include: hypothesis, steady-state metrics, fault injection method, abort
   conditions, and expected outcome.

5. **Runbook Creation:** Write a runbook for "disk space exceeding 90% on a
   production host." Include diagnostic steps, remediation actions (identify
   large files, clean logs, expand storage), escalation criteria, and
   verification steps.

6. **Capacity Planning:** Your service currently handles 10,000 requests per
   second at 60% CPU utilization. Traffic is growing at 15% per month. At what
   month will you exceed 80% CPU utilization? Write the pseudocode to compute
   this and determine when to provision additional capacity.
