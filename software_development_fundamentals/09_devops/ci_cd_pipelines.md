# CI/CD Pipelines

## Introduction

Continuous Integration (CI) and Continuous Delivery/Deployment (CD) form the
backbone of modern software delivery. CI ensures that code changes are
automatically validated through building and testing. CD extends this by
automating the release process so that validated changes can be deployed to
production reliably and repeatedly.

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                     CI/CD PIPELINE OVERVIEW                      │
  │                                                                  │
  │  ┌────────┐   ┌───────┐   ┌──────┐   ┌─────────┐   ┌────────┐ │
  │  │ Commit │──▶│ Build │──▶│ Test │──▶│ Package │──▶│ Deploy │ │
  │  └────────┘   └───────┘   └──────┘   └─────────┘   └────────┘ │
  │       │                                                  │      │
  │       │            Continuous Integration                 │      │
  │       │◄────────────────────────────────────────────────▶│      │
  │       │                                                  │      │
  │       │            Continuous Delivery                    │      │
  │       │◄────────────────────────────────────────────────▶│      │
  │       │                                     ┌────────┐   │      │
  │       │            Continuous Deployment     │Auto    │   │      │
  │       │◄───────────────────────────────────▶│Release │──▶│      │
  │       │                                     └────────┘   │      │
  └──────────────────────────────────────────────────────────────────┘
```

**Continuous Delivery** means every change *can* be deployed at any time but a
human makes the final decision. **Continuous Deployment** means every validated
change is *automatically* released to production without human intervention.

---

## Pipeline Stages

A typical pipeline moves code through a series of sequential gates. Each stage
must pass before the next begins.

```
  ┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐
  │  SOURCE   │───▶│   BUILD   │───▶│   TEST    │───▶│  PACKAGE  │───▶│  DEPLOY   │
  │           │    │           │    │           │    │           │    │           │
  │ - Checkout│    │ - Compile │    │ - Unit    │    │ - Bundle  │    │ - Stage   │
  │ - Lint    │    │ - Resolve │    │ - Integr. │    │ - Version │    │ - Smoke   │
  │ - Validate│    │   deps    │    │ - E2E     │    │ - Sign    │    │ - Release │
  └───────────┘    └───────────┘    └───────────┘    └───────────┘    └───────────┘
       │                │                │                │                │
       ▼                ▼                ▼                ▼                ▼
   [Fail fast]     [Fail fast]     [Fail fast]     [Fail fast]     [Rollback]
```

### Pipeline-as-Code

Defining your pipeline in a version-controlled file ensures reproducibility and
auditability. Below is a language-agnostic pseudocode representation:

```
PIPELINE "main-pipeline"
  TRIGGER ON push TO branches ["main", "release/*"]
  TRIGGER ON pull_request TO branches ["main"]

  ENVIRONMENT
    SET build_timeout = 600 SECONDS
    SET test_timeout = 1200 SECONDS
    SET artifact_retention = 30 DAYS

  STAGE "source"
    STEP "checkout"
      CHECKOUT repository AT current_commit
    END STEP
    STEP "lint"
      RUN static_analysis ON source_directory
      IF lint_errors > 0 THEN
        FAIL WITH "Linting errors found"
      END IF
    END STEP
  END STAGE

  STAGE "build"
    DEPENDS ON "source"
    STEP "compile"
      RUN build_command WITH options [optimized, warnings_as_errors]
      IF exit_code != 0 THEN
        FAIL WITH "Build failed"
      END IF
    END STEP
    STEP "resolve-dependencies"
      RUN dependency_resolver WITH lockfile
      VERIFY checksums MATCH lockfile
    END STEP
  END STAGE

  STAGE "test"
    DEPENDS ON "build"
    PARALLEL
      STEP "unit-tests"
        RUN test_runner WITH scope "unit"
        REQUIRE coverage >= 80 PERCENT
      END STEP
      STEP "integration-tests"
        RUN test_runner WITH scope "integration"
      END STEP
    END PARALLEL
  END STAGE

  STAGE "package"
    DEPENDS ON "test"
    STEP "create-artifact"
      SET version = GENERATE_VERSION(commit_hash, timestamp)
      RUN packager WITH version
      SIGN artifact WITH signing_key
      UPLOAD artifact TO artifact_store
    END STEP
  END STAGE

  STAGE "deploy"
    DEPENDS ON "package"
    REQUIRES approval FROM "release-team" IF branch == "main"
    STEP "deploy-staging"
      DEPLOY artifact TO environment "staging"
      RUN smoke_tests AGAINST "staging"
    END STEP
    STEP "deploy-production"
      DEPLOY artifact TO environment "production"
      RUN smoke_tests AGAINST "production"
      IF smoke_tests FAIL THEN
        ROLLBACK TO previous_version
      END IF
    END STEP
  END STAGE
END PIPELINE
```

---

## Branch Strategies

### Trunk-Based Development

All developers commit to a single main branch. Short-lived feature branches
(lasting hours, not days) may be used but are merged quickly.

```
  main  ─────●────●────●────●────●────●────●────▶
              │         ▲    │         ▲
              │         │    │         │
  feat/a      └──●──●──┘    └──●──●──┘  feat/b
              (hours)        (hours)
```

**Advantages:** Fewer merge conflicts, faster feedback, simpler pipeline.
**Trade-off:** Requires feature flags for incomplete work.

### Feature Branch Strategy

Each feature or fix lives in a long-lived branch that is merged through a pull
request after review and passing CI.

```
  main     ─────●─────────────────●──────────────────●────▶
                 │                 ▲                  ▲
                 │                 │                  │
  feature/x      └──●──●──●──●──┘                   │
                    (days/weeks)                      │
  feature/y              └──●──●──●──●──●──●────────┘
                            (days/weeks)
```

**Advantages:** Isolation for complex features, natural review point.
**Trade-off:** Merge conflicts grow with branch lifetime.

---

## Artifact Management

An artifact is any output of a build process: compiled binaries, packaged
archives, container images, documentation bundles.

```
  ┌─────────────────────────────────────────────────┐
  │              ARTIFACT LIFECYCLE                   │
  │                                                   │
  │  Build ──▶ Version ──▶ Sign ──▶ Store ──▶ Deploy │
  │              │                    │                │
  │              ▼                    ▼                │
  │        Semantic Tag         Verification          │
  │        (1.4.2-rc1)         at deploy time         │
  └─────────────────────────────────────────────────┘
```

### Versioning Pseudocode

```
FUNCTION generate_version(branch, commit_hash, build_number)
  SET major = READ_FROM config "version.major"
  SET minor = READ_FROM config "version.minor"
  SET patch = READ_FROM config "version.patch"

  IF branch == "main" THEN
    RETURN FORMAT("{major}.{minor}.{patch}")
  ELSE IF branch STARTS WITH "release/" THEN
    RETURN FORMAT("{major}.{minor}.{patch}-rc{build_number}")
  ELSE
    SET short_hash = FIRST 8 CHARACTERS OF commit_hash
    RETURN FORMAT("{major}.{minor}.{patch}-dev+{short_hash}")
  END IF
END FUNCTION
```

**Principles:**
- Artifacts are immutable once created; never overwrite a published version.
- Store artifacts in a dedicated repository, not in source control.
- Retain artifacts according to a defined retention policy.
- Sign artifacts cryptographically and verify signatures before deployment.

---

## Deployment Strategies

### Blue-Green Deployment

Two identical environments exist. One ("blue") serves live traffic while the
other ("green") receives the new release. Traffic is switched atomically.

```
  BEFORE SWITCH                        AFTER SWITCH
  ┌──────────┐                         ┌──────────┐
  │  ROUTER  │                         │  ROUTER  │
  └────┬─────┘                         └────┬─────┘
       │                                    │
  ┌────▼─────┐  ┌──────────┐         ┌─────┴────┐  ┌──────────┐
  │  BLUE    │  │  GREEN   │         │  BLUE    │  │  GREEN   │
  │  v1.3   │  │  v1.4    │         │  v1.3   │  │  v1.4    │
  │ (live)  │  │ (staged) │         │ (idle)  │  │ (live)   │
  └──────────┘  └──────────┘         └──────────┘  └──────────┘
```

**Advantage:** Instant rollback by switching traffic back.
**Trade-off:** Requires double the infrastructure.

### Canary Deployment

A small percentage of traffic is routed to the new version. If metrics remain
healthy, traffic is gradually shifted until the new version receives 100%.

```
  Step 1: 5% canary        Step 2: 25% canary       Step 3: 100% new
  ┌──────────┐             ┌──────────┐             ┌──────────┐
  │  ROUTER  │             │  ROUTER  │             │  ROUTER  │
  └────┬─────┘             └────┬─────┘             └────┬─────┘
    95%│  5%                75% │  25%                    │100%
  ┌────▼──┐ ┌──▼───┐     ┌────▼──┐ ┌──▼───┐         ┌──▼─────┐
  │ v1.3  │ │ v1.4 │     │ v1.3  │ │ v1.4 │         │  v1.4  │
  └───────┘ └──────┘     └───────┘ └──────┘         └────────┘
```

**Advantage:** Limits blast radius of faulty releases.
**Trade-off:** Requires traffic-splitting capability and robust metrics.

### Rolling Deployment

Instances are updated one at a time (or in small batches). At any moment, both
old and new versions coexist.

```
  Time ──▶

  Instance 1:  [  v1.3  ][ updating ][   v1.4   ]
  Instance 2:  [  v1.3           ][ updating ][   v1.4   ]
  Instance 3:  [  v1.3                    ][ updating ][   v1.4   ]
  Instance 4:  [  v1.3                             ][ updating ][  v1.4  ]
```

**Advantage:** No extra infrastructure needed.
**Trade-off:** Both versions run simultaneously; must handle compatibility.

---

## Rollback Procedures

```
FUNCTION rollback(environment, target_version)
  SET current = GET_DEPLOYED_VERSION(environment)
  LOG "Rolling back {environment} from {current} to {target_version}"

  SET artifact = FETCH_ARTIFACT(target_version)
  IF artifact IS NOT FOUND THEN
    FAIL WITH "Artifact for {target_version} not found in store"
  END IF

  VERIFY_SIGNATURE(artifact)

  DEPLOY artifact TO environment
  RUN smoke_tests AGAINST environment

  IF smoke_tests PASS THEN
    LOG "Rollback to {target_version} successful"
    NOTIFY operations_channel "Rollback complete"
  ELSE
    LOG "Rollback smoke tests failed -- escalating"
    ALERT on_call_team WITH severity "critical"
  END IF
END FUNCTION
```

**Rollback best practices:**
- Always keep at least the last N stable artifacts available.
- Automate rollback triggers based on health-check failures.
- Practice rollbacks regularly; an untested rollback is not a rollback.
- Ensure database migrations are backward-compatible (expand-contract pattern).

---

## Pipeline Security

### Secrets Management

```
  ┌───────────────────────────────────────────────┐
  │          SECRETS IN PIPELINES                  │
  │                                                │
  │  ┌──────────┐    ┌───────────────┐            │
  │  │ Pipeline │───▶│ Secrets Vault │            │
  │  │  Config  │    │  (encrypted)  │            │
  │  └──────────┘    └───────┬───────┘            │
  │                          │                     │
  │                 Injected at runtime            │
  │                 as environment vars             │
  │                          │                     │
  │                   ┌──────▼──────┐              │
  │                   │ Build Step  │              │
  │                   │ (ephemeral) │              │
  │                   └─────────────┘              │
  │                                                │
  │  RULES:                                        │
  │  - Never store secrets in source control       │
  │  - Rotate secrets on a schedule                │
  │  - Audit access to secret values               │
  │  - Scope secrets to minimum required stages    │
  └───────────────────────────────────────────────┘
```

### Signed Artifacts

```
FUNCTION sign_artifact(artifact_path, signing_key)
  SET hash = COMPUTE_HASH(artifact_path, algorithm = "SHA-256")
  SET signature = SIGN(hash, signing_key)
  STORE signature AT artifact_path + ".sig"
  RETURN signature
END FUNCTION

FUNCTION verify_artifact(artifact_path, public_key)
  SET expected_hash = COMPUTE_HASH(artifact_path, algorithm = "SHA-256")
  SET signature = READ(artifact_path + ".sig")
  SET verified = VERIFY(expected_hash, signature, public_key)

  IF NOT verified THEN
    FAIL WITH "Artifact signature verification failed"
  END IF
  RETURN TRUE
END FUNCTION
```

### Additional Security Practices

- **Least privilege:** Pipeline service accounts should have only the
  permissions they need for their specific stage.
- **Ephemeral environments:** Build agents should start clean and be destroyed
  after each run to prevent cross-build contamination.
- **Dependency scanning:** Automatically scan dependencies for known
  vulnerabilities before packaging.
- **Audit trail:** Every pipeline run should produce an immutable log of what
  was built, tested, and deployed.

---

## Practice Problems

1. **Pipeline Design:** Sketch a pipeline-as-code definition for a project
   that has three components: a backend service, a frontend application, and a
   shared library. The shared library must be built first, then the backend and
   frontend can build in parallel.

2. **Branch Strategy Trade-offs:** Your team of fifteen developers ships a
   web application weekly. Feature branches average five days. Merge conflicts
   occur on roughly 30% of pull requests. Propose a branch strategy that
   reduces this conflict rate and explain your reasoning.

3. **Deployment Strategy Selection:** You operate a service handling
   financial transactions. Downtime is unacceptable, and any bug in a new
   release could cause monetary loss. Which deployment strategy would you
   choose and why? Describe the rollback procedure.

4. **Artifact Integrity:** Write pseudocode for a pipeline step that
   verifies an artifact's signature, checks its checksum against a manifest,
   and rejects deployment if either check fails.

5. **Security Audit:** A pipeline currently stores database credentials in
   its pipeline-as-code file (in plain text). Describe the steps you would
   take to remediate this, including how credentials would be injected at
   runtime without exposing them in logs.

6. **Canary Analysis:** Design pseudocode for an automated canary analysis
   function. It should compare error rates and latency between the canary
   and baseline populations over a configurable window, and automatically
   roll back if metrics exceed thresholds.
