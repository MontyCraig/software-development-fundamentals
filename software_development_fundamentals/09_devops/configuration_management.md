# Configuration Management

## Introduction

Configuration management is the practice of handling application settings,
environment-specific values, feature toggles, and secrets in a systematic,
auditable, and secure way. Proper configuration management ensures that the
same application artifact can run correctly across development, staging, and
production environments without modification.

```
  ┌────────────────────────────────────────────────────────────┐
  │             CONFIGURATION MANAGEMENT OVERVIEW               │
  │                                                             │
  │  ┌──────────┐   ┌──────────┐   ┌──────────┐              │
  │  │  Source   │   │  Source   │   │  Source   │              │
  │  │  Code     │   │  Config   │   │  Secrets  │              │
  │  │  (repo)   │   │  (repo)   │   │  (vault)  │              │
  │  └─────┬────┘   └─────┬────┘   └─────┬────┘              │
  │        │               │               │                    │
  │        └───────────────┼───────────────┘                    │
  │                        ▼                                    │
  │               ┌─────────────────┐                          │
  │               │ Merged at deploy│                          │
  │               │ or at runtime   │                          │
  │               └────────┬────────┘                          │
  │                        │                                    │
  │        ┌───────────────┼───────────────┐                   │
  │        ▼               ▼               ▼                    │
  │   ┌─────────┐    ┌─────────┐    ┌───────────┐            │
  │   │   DEV   │    │ STAGING │    │PRODUCTION │            │
  │   │         │    │         │    │           │            │
  │   │ debug=T │    │ debug=F │    │ debug=F   │            │
  │   │ db=local│    │ db=stg  │    │ db=prod   │            │
  │   │ log=DBG │    │ log=INF │    │ log=WARN  │            │
  │   └─────────┘    └─────────┘    └───────────┘            │
  │                                                             │
  │  PRINCIPLE: Same artifact, different configuration          │
  └────────────────────────────────────────────────────────────┘
```

---

## Environment-Specific Configuration Patterns

### Pattern 1: Environment Variables

Configuration is injected through environment variables at runtime. The
application reads variables on startup.

```
FUNCTION load_config_from_environment()
  SET config = {}

  SET config.database_host = READ_ENV("DATABASE_HOST", default = "localhost")
  SET config.database_port = READ_ENV("DATABASE_PORT", default = 5432)
  SET config.log_level = READ_ENV("LOG_LEVEL", default = "INFO")
  SET config.cache_ttl = READ_ENV("CACHE_TTL_SECONDS", default = 300)
  SET config.max_connections = READ_ENV("MAX_CONNECTIONS", default = 100)

  RETURN config
END FUNCTION
```

### Pattern 2: Configuration Files with Overrides

A base configuration file is overridden by environment-specific files.

```
FUNCTION load_config_with_overrides(environment)
  SET base_config = READ_CONFIG_FILE("config/base.conf")
  SET env_config = READ_CONFIG_FILE("config/{environment}.conf")

  // Environment-specific values override base values
  SET merged = DEEP_MERGE(base_config, env_config)

  // Environment variables override file values (highest priority)
  FOR EACH key IN merged DO
    SET env_value = READ_ENV(UPPER_CASE(key))
    IF env_value IS SET THEN
      SET merged[key] = env_value
    END IF
  END FOR

  RETURN merged
END FUNCTION
```

```
  PRIORITY (highest to lowest):

  ┌───────────────────────────┐
  │ 1. Environment variables  │  ◀── Runtime override
  ├───────────────────────────┤
  │ 2. Environment config file│  ◀── Environment-specific
  ├───────────────────────────┤
  │ 3. Base config file       │  ◀── Defaults
  └───────────────────────────┘
```

### Pattern 3: Remote Configuration Service

Configuration is fetched from a centralized service at startup or
refreshed periodically.

```
FUNCTION load_config_from_service(service_url, app_name, environment)
  SET response = HTTP_GET("{service_url}/config/{app_name}/{environment}")

  IF response.status != 200 THEN
    LOG "Config service unavailable -- using cached config"
    RETURN READ_CACHED_CONFIG()
  END IF

  SET config = PARSE(response.body)
  CACHE_CONFIG(config)

  RETURN config
END FUNCTION

FUNCTION watch_config_changes(service_url, app_name, environment, callback)
  SET last_version = 0

  EVERY 30 SECONDS DO
    SET response = HTTP_GET(
      "{service_url}/config/{app_name}/{environment}?since={last_version}"
    )

    IF response.has_changes THEN
      SET last_version = response.version
      SET new_config = PARSE(response.body)
      CALL callback(new_config)
      LOG "Configuration updated to version {last_version}"
    END IF
  END EVERY
END FUNCTION
```

---

## Feature Flags

Feature flags (also called feature toggles) decouple deployment from release.
Code is deployed but features are activated or deactivated through
configuration rather than code changes.

### Toggle Types

```
  ┌─────────────────────────────────────────────────────────┐
  │                  FEATURE FLAG TYPES                      │
  │                                                          │
  │  TYPE         LIFETIME    PURPOSE                        │
  │  ──────────── ────────── ──────────────────────────────  │
  │  Release      Short      Gate incomplete features        │
  │                          Remove after full rollout       │
  │                                                          │
  │  Experiment   Medium     A/B testing and gradual         │
  │                          rollout to percentage of users  │
  │                                                          │
  │  Ops          Permanent  Circuit breakers, graceful      │
  │                          degradation, kill switches      │
  │                                                          │
  │  Permission   Permanent  Entitlements, premium features, │
  │                          role-based access               │
  └─────────────────────────────────────────────────────────┘
```

### Feature Flag Pseudocode

```
FUNCTION create_flag_system()
  SET flags = LOAD_FLAGS_FROM_CONFIG()
  SET system = NEW FlagSystem

  FOR EACH flag IN flags DO
    REGISTER flag IN system
  END FOR

  RETURN system
END FUNCTION

FUNCTION is_enabled(system, flag_name, context)
  SET flag = system.flags[flag_name]

  IF flag IS NOT FOUND THEN
    LOG "Unknown flag: {flag_name} -- returning default FALSE"
    RETURN FALSE
  END IF

  // Check flag type and evaluate
  IF flag.type == "release" THEN
    RETURN flag.enabled

  ELSE IF flag.type == "experiment" THEN
    // Percentage rollout based on consistent hashing
    SET bucket = HASH(context.user_id + flag_name) MODULO 100
    RETURN bucket < flag.percentage

  ELSE IF flag.type == "ops" THEN
    // Check current system conditions
    IF flag.condition == "circuit_breaker" THEN
      SET error_rate = GET_METRIC("error_rate", flag.target_service)
      RETURN error_rate < flag.threshold
    END IF
    RETURN flag.enabled

  ELSE IF flag.type == "permission" THEN
    RETURN context.user_role IN flag.allowed_roles
      OR context.user_id IN flag.allowed_users
      OR context.organization IN flag.allowed_orgs

  END IF

  RETURN FALSE
END FUNCTION

// Usage:
SET flags = create_flag_system()
SET user_context = {user_id: "u-1234", user_role: "premium", organization: "acme"}

IF is_enabled(flags, "new_checkout_flow", user_context) THEN
  EXECUTE new_checkout()
ELSE
  EXECUTE legacy_checkout()
END IF

IF is_enabled(flags, "expensive_computation", user_context) THEN
  EXECUTE full_computation()
ELSE
  LOG "Feature disabled -- returning cached result"
  RETURN cached_result()
END IF
```

### Flag Lifecycle Management

```
FUNCTION audit_stale_flags(system, max_age_days)
  SET stale_flags = EMPTY LIST

  FOR EACH flag IN system.flags DO
    SET age = DAYS_SINCE(flag.created_date)

    IF flag.type == "release" AND age > max_age_days THEN
      APPEND stale_flags WITH {
        name: flag.name,
        age: age,
        recommendation: "Remove flag and dead code path"
      }
    END IF

    IF flag.type == "experiment" AND flag.status == "concluded" THEN
      APPEND stale_flags WITH {
        name: flag.name,
        result: flag.winner,
        recommendation: "Promote winner, remove flag"
      }
    END IF
  END FOR

  IF stale_flags.count > 0 THEN
    LOG "Found {stale_flags.count} stale flags -- cleanup needed"
    NOTIFY engineering_team WITH stale_flags
  END IF

  RETURN stale_flags
END FUNCTION
```

---

## Secrets Management

Secrets (credentials, API keys, certificates, encryption keys) require special
handling distinct from regular configuration.

### Envelope Encryption

```
  ┌──────────────────────────────────────────────────────────┐
  │              ENVELOPE ENCRYPTION                          │
  │                                                           │
  │  ┌─────────────┐                                         │
  │  │ Master Key  │  Stored in hardware security module      │
  │  │ (KEK)       │  or managed key service                  │
  │  └──────┬──────┘                                         │
  │         │ encrypts                                        │
  │         ▼                                                 │
  │  ┌─────────────┐                                         │
  │  │ Data Key    │  Generated per secret or per             │
  │  │ (DEK)       │  batch of secrets                        │
  │  └──────┬──────┘                                         │
  │         │ encrypts                                        │
  │         ▼                                                 │
  │  ┌─────────────┐                                         │
  │  │ Secret      │  Database password, API token, etc.      │
  │  │ (plaintext) │                                          │
  │  └─────────────┘                                         │
  │                                                           │
  │  STORED:                                                  │
  │  ┌───────────────────────────────────┐                   │
  │  │ Encrypted DEK + Encrypted Secret  │                   │
  │  │ (both stored together, master key │                   │
  │  │  is needed to decrypt the DEK,    │                   │
  │  │  which then decrypts the secret)  │                   │
  │  └───────────────────────────────────┘                   │
  └──────────────────────────────────────────────────────────┘
```

### Secret Rotation Pseudocode

```
FUNCTION rotate_secret(secret_name, generator_function)
  // Generate new secret value
  SET new_value = CALL generator_function()

  // Store new version (old version still active)
  SET new_version = STORE_SECRET(secret_name, new_value, status = "pending")
  LOG "New version {new_version} created for {secret_name}"

  // Update consumers to use new version
  SET consumers = GET_SECRET_CONSUMERS(secret_name)
  FOR EACH consumer IN consumers DO
    NOTIFY consumer TO reload_secret(secret_name, new_version)
    WAIT UNTIL consumer.confirmed_version == new_version
      TIMEOUT 5 MINUTES
    IF NOT consumer.confirmed THEN
      LOG "Consumer {consumer.name} did not confirm -- rolling back"
      REVOKE_SECRET(secret_name, new_version)
      RETURN {status: "FAILED", reason: "consumer_timeout"}
    END IF
  END FOR

  // Mark old versions as deprecated
  SET old_versions = GET_VERSIONS(secret_name, status = "active")
  FOR EACH version IN old_versions DO
    IF version.id != new_version THEN
      SET version.status = "deprecated"
      SCHEDULE deletion OF version IN 24 HOURS
    END IF
  END FOR

  SET new_version.status = "active"
  LOG "Secret {secret_name} rotated successfully to version {new_version}"
  RETURN {status: "SUCCESS", version: new_version}
END FUNCTION
```

---

## Configuration Validation

Configuration errors are a leading cause of outages. Validate configuration
before it reaches production.

```
FUNCTION validate_config(config, schema)
  SET errors = EMPTY LIST

  // Type checking
  FOR EACH field IN schema.fields DO
    SET value = config[field.name]

    IF field.required AND value IS NOT SET THEN
      APPEND errors WITH "Missing required field: {field.name}"
      CONTINUE
    END IF

    IF value IS SET AND TYPE_OF(value) != field.type THEN
      APPEND errors WITH "Field {field.name}: expected {field.type}, got {TYPE_OF(value)}"
    END IF

    IF field.min IS SET AND value < field.min THEN
      APPEND errors WITH "Field {field.name}: value {value} below minimum {field.min}"
    END IF

    IF field.max IS SET AND value > field.max THEN
      APPEND errors WITH "Field {field.name}: value {value} above maximum {field.max}"
    END IF

    IF field.pattern IS SET AND NOT MATCHES(value, field.pattern) THEN
      APPEND errors WITH "Field {field.name}: value does not match pattern {field.pattern}"
    END IF

    IF field.one_of IS SET AND value NOT IN field.one_of THEN
      APPEND errors WITH "Field {field.name}: value must be one of {field.one_of}"
    END IF
  END FOR

  // Cross-field validation
  IF config.max_connections IS SET AND config.connection_pool_size IS SET THEN
    IF config.connection_pool_size > config.max_connections THEN
      APPEND errors WITH "connection_pool_size cannot exceed max_connections"
    END IF
  END IF

  IF errors.count > 0 THEN
    LOG "Configuration validation failed: {errors.count} errors"
    FOR EACH error IN errors DO
      LOG "  - {error}"
    END FOR
    RETURN {valid: FALSE, errors: errors}
  END IF

  RETURN {valid: TRUE, errors: EMPTY}
END FUNCTION

// Pre-deployment config check
FUNCTION validate_before_deploy(config, environment)
  SET schema = LOAD_SCHEMA("config_schema.def")
  SET result = validate_config(config, schema)

  IF NOT result.valid THEN
    FAIL WITH "Configuration invalid for {environment}: {result.errors}"
  END IF

  // Environment-specific sanity checks
  IF environment == "production" THEN
    IF config.debug == TRUE THEN
      FAIL WITH "Debug mode must be disabled in production"
    END IF
    IF config.log_level == "TRACE" OR config.log_level == "DEBUG" THEN
      FAIL WITH "Verbose logging not permitted in production"
    END IF
  END IF

  LOG "Configuration validated for {environment}"
  RETURN TRUE
END FUNCTION
```

---

## Twelve-Factor App Configuration Principles

The twelve-factor methodology defines configuration as "everything that is
likely to vary between deploys." These principles apply to configuration:

```
  ┌──────────────────────────────────────────────────────────┐
  │         TWELVE-FACTOR CONFIGURATION PRINCIPLES            │
  │                                                           │
  │  1. STRICT SEPARATION                                     │
  │     Config is NOT code. Never store config values         │
  │     in the codebase. Config varies by deploy;             │
  │     code does not.                                        │
  │                                                           │
  │  2. STORE IN THE ENVIRONMENT                              │
  │     Use environment variables for configuration.          │
  │     They are language-agnostic, OS-agnostic, and         │
  │     cannot accidentally be checked into source control.   │
  │                                                           │
  │  3. NO GROUPING BY ENVIRONMENT NAME                       │
  │     Avoid "development", "staging", "production"          │
  │     bundles in code. Instead, each config value is        │
  │     independently managed. This scales to many            │
  │     deploys without combinatorial explosion.              │
  │                                                           │
  │  4. THE LITMUS TEST                                       │
  │     Could the codebase be open-sourced at any             │
  │     moment without compromising credentials?              │
  │     If yes, config is properly separated.                 │
  │                                                           │
  │  5. BACKING SERVICES AS ATTACHED RESOURCES                │
  │     Database URLs, cache addresses, and message           │
  │     queues are configuration -- swappable without         │
  │     code changes.                                         │
  └──────────────────────────────────────────────────────────┘
```

---

## Configuration Drift and Reconciliation

Configuration drift occurs when the actual configuration of a running system
diverges from the intended configuration stored in the source of truth.

```
  ┌───────────────────────────────────────────────────────┐
  │          CONFIGURATION DRIFT DETECTION                 │
  │                                                        │
  │  Source of Truth          Running System               │
  │  ┌───────────────┐       ┌───────────────┐            │
  │  │ log_level=INFO│       │ log_level=DBG │  ◀── DRIFT │
  │  │ cache_ttl=300 │       │ cache_ttl=300 │  ◀── OK    │
  │  │ max_conn=100  │       │ max_conn=50   │  ◀── DRIFT │
  │  │ debug=false   │       │ debug=true    │  ◀── DRIFT │
  │  └───────────────┘       └───────────────┘            │
  │                                                        │
  │  3 of 4 values have drifted                           │
  └───────────────────────────────────────────────────────┘
```

### Drift Reconciliation Pseudocode

```
FUNCTION detect_config_drift(intended_config, running_instances)
  SET drift_report = EMPTY LIST

  FOR EACH instance IN running_instances DO
    SET actual_config = FETCH_RUNTIME_CONFIG(instance)

    FOR EACH key IN intended_config DO
      IF actual_config[key] != intended_config[key] THEN
        APPEND drift_report WITH {
          instance: instance.id,
          key: key,
          intended: intended_config[key],
          actual: actual_config[key],
          severity: classify_drift_severity(key)
        }
      END IF
    END FOR

    // Check for unexpected configuration keys
    FOR EACH key IN actual_config DO
      IF key NOT IN intended_config THEN
        APPEND drift_report WITH {
          instance: instance.id,
          key: key,
          intended: NOT_SET,
          actual: actual_config[key],
          severity: "WARNING"
        }
      END IF
    END FOR
  END FOR

  RETURN drift_report
END FUNCTION

FUNCTION reconcile_drift(drift_report, strategy)
  FOR EACH drift IN drift_report DO
    IF strategy == "enforce" THEN
      // Force intended value
      SET_RUNTIME_CONFIG(drift.instance, drift.key, drift.intended)
      LOG "Enforced {drift.key} = {drift.intended} on {drift.instance}"

    ELSE IF strategy == "report" THEN
      // Only report, do not change
      LOG "Drift detected: {drift.instance} {drift.key} "
        + "expected={drift.intended} actual={drift.actual}"

    ELSE IF strategy == "selective" THEN
      // Enforce critical, report non-critical
      IF drift.severity == "CRITICAL" THEN
        SET_RUNTIME_CONFIG(drift.instance, drift.key, drift.intended)
        LOG "Enforced critical config: {drift.key} on {drift.instance}"
      ELSE
        LOG "Non-critical drift: {drift.key} on {drift.instance}"
      END IF
    END IF
  END FOR
END FUNCTION
```

---

## Practice Problems

1. **Configuration Loading:** Design a configuration loading function that
   supports three layers of precedence (default values, configuration file,
   environment variables) and validates all values against a schema before
   the application starts. Include error handling for missing required values
   and type mismatches.

2. **Feature Flag Design:** A team wants to gradually roll out a new search
   algorithm to users. Design a feature flag system that: (a) rolls out to
   5% of users initially, (b) increases to 25%, 50%, and 100% based on
   success metrics, (c) automatically rolls back to 0% if error rate exceeds
   a threshold, and (d) ensures a given user always sees the same variant
   (sticky assignment).

3. **Secret Rotation:** Write pseudocode for rotating a database credential
   where the database supports two active passwords simultaneously. The
   rotation should: create a new password, update the database to accept both
   old and new, update all application instances to use the new password,
   verify all instances have switched, then revoke the old password.

4. **Drift Prevention:** Your team discovers that operators frequently change
   configuration on running instances via a management console, causing drift.
   Propose a system design that (a) detects drift within minutes, (b) alerts
   the team, and (c) optionally auto-remediate. Consider how to handle
   legitimate emergency changes.

5. **Twelve-Factor Audit:** Review the following configuration practices and
   identify which ones violate twelve-factor principles. For each violation,
   explain the risk and propose a correction:
   a. Database URL is hardcoded in a source file with comments like
      "change this for production"
   b. API keys are stored in environment variables on the deployment server
   c. Configuration file has sections labeled "dev", "staging", "prod"
   d. A secret is stored in an encrypted file committed to the repository

6. **Flag Cleanup:** Write pseudocode for a tool that scans the codebase for
   references to feature flags, compares them against the flag configuration
   service, and reports: (a) flags referenced in code but not in the service,
   (b) flags in the service but not referenced in code, and (c) release flags
   older than a configurable maximum age.
