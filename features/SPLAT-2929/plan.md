# Test Plan for CAPI Support for Installing External Platform Clusters (Day 0)

**Plan Identifier:** TP-SPLAT-2929-001
**Plan Status:** Draft — Planning phase; version 5.1 is scoped to research and planning, not a committed implementation release.
**Feature Epic:** [SPLAT-2929](https://redhat.atlassian.net/browse/SPLAT-2929) — [Day 0] Add Cluster API (CAPI) support for installing external platform clusters
**Related Strategy:** [OCPSTRAT-3724](https://redhat.atlassian.net/browse/OCPSTRAT-3724) — Strategic feature for external-platform CAPI support
**Research Spike:** [SPLAT-2902](https://redhat.atlassian.net/browse/SPLAT-2902) — Self-managed CAPI feasibility with external profile / local control plane (in-progress)
**Date:** 2026-09-03
**Revision:** 1.0

---

## Overview

- [References](#references) — Jira tickets, strategy items, related documents
- [Introduction](#introduction) — What the feature is and the problem it solves
- [Test Items](#test-items) — Software components under test
- [Features to Be Tested](#features-to-be-tested) — In-scope functional areas
- [Features Not to Be Tested](#features-not-to-be-tested) — Explicitly excluded scope
- [Approach](#approach) — Test strategy and methodology
- [Test Scope — Scenario Catalogue](#test-scope--scenario-catalogue) — Proposed test scenarios
- [Traceability Matrix](#traceability-matrix) — Scenario-to-requirement mapping
- [Item Pass/Fail Criteria](#item-passfail-criteria) — Exit criteria for test items
- [Suspension and Resumption Criteria](#suspension-and-resumption-criteria)
- [Test Deliverables](#test-deliverables) — Tangible outputs
- [Testing Tasks](#testing-tasks) — Work breakdown
- [Environmental Needs](#environmental-needs) — Infrastructure and tooling
- [Responsibilities](#responsibilities) — Roles and ownership
- [Staffing and Training Needs](#staffing-and-training-needs)
- [Schedule](#schedule) — Milestones and dependencies
- [Risks and Contingencies](#risks-and-contingencies)
- [Approvals](#approvals)
- [References (Full List)](#references-full-list)
- [Glossary](#glossary)

---

## References

| Type | Link | Notes |
|------|------|-------|
| Feature Epic | [SPLAT-2929](https://redhat.atlassian.net/browse/SPLAT-2929) | Day 0 CAPI for external platform |
| Research Spike | [SPLAT-2902](https://redhat.atlassian.net/browse/SPLAT-2902) | Self-managed CAPI feasibility; local control plane; in-progress |
| Strategic Feature | [OCPSTRAT-3724](https://redhat.atlassian.net/browse/OCPSTRAT-3724) | Broader CAPI external-platform strategy |
| Day-2 Strategy | [OCPSTRAT-1322](https://redhat.atlassian.net/browse/OCPSTRAT-1322) | Day-2 autoscaling / machine-health-check; excluded from this plan |
| Docs Tracker | [OSDOCS-21999](https://redhat.atlassian.net/browse/OSDOCS-21999) | Documentation linked from OCPSTRAT-3724 |
| OPCT External/HCP | [OPCT-314](https://redhat.atlassian.net/browse/OPCT-314) | External platform on HCP currently unsupported |
| Plan Template | [kueue-operator feature-test-plan-template](https://github.com/openshift/kueue-operator/blob/main/docs/plans/templates/feature-test-plan-template.md) | Style reference |
| Plan Example | [kueue-operator hierarchical-cohorts plan](https://github.com/openshift/kueue-operator/blob/main/docs/plans/features/hierarchical-cohorts.md) | Content reference |
| IEEE 829 | IEEE 829-2008 / ISO/IEC/IEEE 29119 | Standard for software test documentation |

---

## Introduction

### Problem Statement

OpenShift `platform=external` currently has no installer-native infrastructure provisioning path. Partners and customers who want to install OpenShift on non-integrated platforms — with Oracle Cloud Infrastructure (OCI) as the concrete initial driver — must use Assisted Installer or User-Provisioned Infrastructure (UPI). Infrastructure lifecycle is entirely outside installer control, and no machine-management layer is wired from day 0.

### Desired Outcome

The installer provisions infrastructure for an external-platform cluster using a **partner-supplied Cluster API (CAPI) infrastructure provider**, delivering an IPI-like install experience without requiring in-tree platform integration for each partner cloud. The install-time bootstrap control plane invokes the partner CAPI provider to create machines, and credentials are handled without a partner fork of the installer.

### Scope of This Plan

This test plan covers the **day-0 install path only**: installer bootstrap → CAPI provider invocation → machine provisioning → cluster ready. It explicitly **excludes** day-2 machine management (autoscaling, MachineHealthCheck), which is tracked under [OCPSTRAT-1322](https://redhat.atlassian.net/browse/OCPSTRAT-1322), and HCP / managed-service topologies, which are currently unsupported per [OPCT-314](https://redhat.atlassian.net/browse/OPCT-314).

> **Planning caveat:** Version 5.1 is scoped to research and planning. Many design decisions (delivery model, credential handling mechanism, CAPI provider packaging) are pending the outcome of [SPLAT-2902](https://redhat.atlassian.net/browse/SPLAT-2902). Sections below mark these unknowns explicitly.

---

## Test Items

The following components are expected to be under test once implementation begins. Exact versions, image references, and binary names are **TBD** pending design decisions from SPLAT-2902.

| Item | Description | Version / Status |
|------|-------------|------------------|
| `openshift-install` binary | Installer with CAPI external-platform support | TBD — not yet implemented |
| Partner CAPI infrastructure provider | Out-of-tree CAPI provider (OCI as initial target) | TBD — provider version and packaging undecided |
| CAPI core controllers | Cluster API core (cluster-api-controller-manager) | TBD — version aligned with OCP release |
| Install-config manifest | `install-config.yaml` with `platform: external` + CAPI stanza | TBD — schema not yet defined |
| Bootstrap control plane | Temporary control plane hosting CAPI controllers during install | TBD — local vs. ephemeral design pending |
| Credential injection mechanism | Partner cloud credentials supplied at install time | TBD — mechanism pending design |
| Partner integration contract (documentation) | Documented interface for partner CAPI provider onboarding | TBD |

---

## Features to Be Tested

| ID | Feature Area | Description |
|----|-------------|-------------|
| F-01 | End-to-end install | Complete install using `openshift-install create cluster` with an external-platform CAPI provider (OCI target) |
| F-02 | Install-config validation | Schema validation for external-platform CAPI fields in `install-config.yaml` |
| F-03 | Credential handling | Installer accepts and propagates partner cloud credentials without partner-specific installer fork |
| F-04 | CAPI provider lifecycle | CAPI infrastructure provider is deployed, invoked, and torn down correctly during bootstrap |
| F-05 | Bootstrap pivot | Control plane pivots from bootstrap to permanent cluster; CAPI resources migrate correctly |
| F-06 | Failure / error handling | Graceful handling of provider errors, timeouts, invalid credentials, missing prerequisites |
| F-07 | Idempotency / re-entrant install | Install can be retried after transient failures without orphaned cloud resources |
| F-08 | Observability | Install logs emit actionable diagnostics for CAPI provider operations |
| F-09 | Destroy workflow | `openshift-install destroy cluster` cleans up CAPI-provisioned infrastructure |
| F-10 | Security boundary | Credentials are not leaked to logs, configmaps, or unauthenticated endpoints during install |
| F-11 | Non-regression | Existing in-tree platform installs (AWS, GCP, Azure, etc.) are unaffected by CAPI external-platform changes |

---

## Features Not to Be Tested

| Feature | Reason for Exclusion |
|---------|----------------------|
| Day-2 autoscaling / MachineHealthCheck | Tracked under [OCPSTRAT-1322](https://redhat.atlassian.net/browse/OCPSTRAT-1322); not in SPLAT-2929 scope |
| HCP / Hosted Control Plane topology | Currently unsupported for external platform per [OPCT-314](https://redhat.atlassian.net/browse/OPCT-314); must be N/A or an explicitly gated future item |
| Managed-service (ROSA, ARO) topologies | Not in scope; additional research required per SPLAT-2902 |
| Day-2 node scaling / Machine API operations | Day-0 only; day-2 surface deferred |
| In-tree platform provider behavior (AWS, GCP, Azure, vSphere, etc.) | Tested by existing platform CI; this plan covers **non-regression** only |
| Partner CAPI provider internal correctness | Partner responsibility; this plan validates the integration contract, not the provider's internal logic |
| Assisted Installer or UPI install paths | Existing paths; not changed by this feature |
| Upgrade from CAPI-installed cluster | Day-2 concern; not in scope for day-0 plan |

### Scenarios Considered and Excluded

| Scenario | Reason |
|----------|--------|
| Multi-provider install (two CAPI providers simultaneously) | Not a stated requirement; no design support |
| In-payload bundling of partner provider images | Delivery model undecided; cannot test until resolved |
| Windows node provisioning via CAPI | Not in scope for external platform day 0 |
| HCP-based external platform install | [OPCT-314](https://redhat.atlassian.net/browse/OPCT-314) explicitly states this is unsupported |

---

## Approach

### Test Strategy

The testing approach is **three-tiered**, pending design finalization:

1. **Unit Tests** — Installer code: install-config validation, credential propagation, CAPI manifest generation. These run in the installer repository CI.
2. **Integration Tests** — CAPI provider lifecycle in a simulated or local environment (e.g., `envtest`, local bootstrap). Validate the orchestration contract without requiring real cloud resources.
3. **End-to-End (E2E) Tests** — Full install on OCI (or equivalent external platform) using a real partner CAPI provider. This is the primary signal for "does the feature work?"

### Upstream vs. Downstream

- **Upstream (CAPI):** Upstream Cluster API has its own test suites. This plan does **not** contribute tests to `kubernetes-sigs/cluster-api`. Instead, we rely on upstream CAPI conformance and focus on the **OpenShift installer integration**.
- **Downstream (OpenShift installer):** All scenarios below are downstream tests contributed to the `openshift/installer` repository (or a testing repository TBD). CI job configuration is pending.

### Existing vs. Proposed Tests

- **Existing:** The OpenShift installer has E2E CI jobs for in-tree platforms (`e2e-aws`, `e2e-gcp`, `e2e-azure`, etc.) and `platform=external` UPI paths. These provide the **non-regression baseline** but do not cover CAPI-based installs.
- **Proposed:** All scenarios in this plan (P-xx, N-xx, S-xx, etc.) are **new proposed tests**. None exist today.

### Test Categories

Each scenario is categorized as:

| Category | Code | Description |
|----------|------|-------------|
| Positive / Happy Path | P | Feature works as intended |
| Negative / Error | N | Correct behavior under invalid input or failure |
| Security / Credential | S | Credential handling, secret isolation |
| Failure Recovery | FR | Retry, idempotency, cleanup after failure |
| Compatibility | C | Cross-version, cross-architecture checks |
| Observability | O | Logging, events, diagnostics |
| Non-Regression | NR | Existing functionality unaffected |
| Idempotency | I | Re-entrant operations produce consistent results |

---

## Test Scope — Scenario Catalogue

> **Status note:** All scenarios below are **proposed**. None are implemented. Implementation is blocked on design decisions from [SPLAT-2902](https://redhat.atlassian.net/browse/SPLAT-2902).

### Positive / Happy Path Scenarios

| ID | Scenario | Preconditions | Steps (High-Level) | Expected Result | Feature Ref |
|----|----------|---------------|---------------------|-----------------|-------------|
| P-01 | End-to-end CAPI install on OCI | Valid OCI credentials; partner CAPI provider available; install-config with `platform: external` + CAPI stanza | Run `openshift-install create cluster` | Cluster reaches `Ready`; all nodes provisioned via CAPI; console accessible | F-01 |
| P-02 | Install-config with valid CAPI external fields | Valid install-config YAML | Run `openshift-install create install-config` validation | Validation passes; no errors | F-02 |
| P-03 | Credential propagation to CAPI provider | Valid partner credentials in expected format | Install proceeds; CAPI provider authenticates to partner cloud | Provider creates machines without auth failure | F-03 |
| P-04 | CAPI provider deployment during bootstrap | Bootstrap cluster running | Observe bootstrap control plane pods | Partner CAPI provider controller is running and healthy | F-04 |
| P-05 | Bootstrap pivot completes | Install reaches pivot phase | Observe cluster after pivot | CAPI resources present on permanent cluster; bootstrap resources removed | F-05 |
| P-06 | Cluster destroy via installer | Running CAPI-installed cluster | Run `openshift-install destroy cluster` | All CAPI-provisioned infrastructure removed from partner cloud; no orphaned resources | F-09 |
| P-07 | Install with custom machine configuration | Install-config with custom machine types / sizes | Run install | Machines provisioned with requested configuration | F-01, F-02 |

### Negative / Error Scenarios

| ID | Scenario | Preconditions | Expected Result | Feature Ref |
|----|----------|---------------|-----------------|-------------|
| N-01 | Invalid install-config (missing CAPI fields) | Malformed or incomplete CAPI stanza | Installer returns clear validation error before provisioning | F-02 |
| N-02 | Invalid partner cloud credentials | Wrong or expired credentials | Installer fails with actionable credential error; no partial infrastructure left behind | F-03, F-06 |
| N-03 | CAPI provider image unavailable | Provider image pull fails | Installer fails with clear error identifying the missing provider; bootstrap cleans up | F-04, F-06 |
| N-04 | CAPI provider timeout during machine creation | Simulated network partition or cloud API unavailability | Installer times out with diagnostic message; partial resources documented or cleaned up | F-06, F-08 |
| N-05 | Unsupported platform value with CAPI stanza | `platform: aws` combined with CAPI external stanza | Installer rejects config with clear error | F-02 |
| N-06 | Destroy with already-deleted infrastructure | Cloud resources manually removed before destroy | Destroy completes without error (idempotent) or returns clear status | F-09 |

### Security / Credential Scenarios

| ID | Scenario | Preconditions | Expected Result | Feature Ref |
|----|----------|---------------|-----------------|-------------|
| S-01 | Credentials not leaked in install logs | Normal install with verbose logging | Grep install logs for credential values; none found | F-10 |
| S-02 | Credentials not stored in cluster ConfigMaps | Completed install | Inspect ConfigMaps and Secrets in `openshift-*` namespaces; partner credentials not in ConfigMaps | F-10 |
| S-03 | Credentials not exposed via unauthenticated API | Completed install | Query unauthenticated API endpoints; no credential material returned | F-10 |
| S-04 | Credential format validation | Various credential formats (JSON, env-var, file-path) | Installer validates and accepts documented formats; rejects undocumented ones | F-03 |

### Failure Recovery / Idempotency Scenarios

| ID | Scenario | Preconditions | Expected Result | Feature Ref |
|----|----------|---------------|-----------------|-------------|
| FR-01 | Retry install after transient cloud failure | First install attempt failed mid-provisioning | Second `create cluster` run succeeds; no duplicate machines or orphaned resources | F-07 |
| FR-02 | Retry install after bootstrap timeout | Bootstrap timed out | Re-run succeeds from clean state or resumes correctly | F-07 |
| FR-03 | Destroy after partial install failure | Install failed leaving partial infrastructure | `destroy cluster` removes all created resources | F-09, F-07 |
| I-01 | Repeated install-config validation | Same config validated multiple times | Identical result each time; no side effects | F-02 |

### Compatibility Scenarios

| ID | Scenario | Preconditions | Expected Result | Feature Ref |
|----|----------|---------------|-----------------|-------------|
| C-01 | Install on x86_64 architecture | x86_64 target infrastructure | Install succeeds | F-01 |
| C-02 | Install on aarch64 architecture | aarch64 target infrastructure (if supported by partner provider) | Install succeeds or is clearly unsupported | F-01 |
| C-03 | FIPS-enabled install | `fips: true` in install-config | Install succeeds with FIPS-mode cluster | F-01 |
| C-04 | Disconnected / restricted-network install | Air-gapped environment with mirror registry | Install succeeds using mirrored images for CAPI provider and OCP | F-01 |
| C-05 | Proxy-configured install | HTTP/HTTPS proxy configured | Install succeeds; CAPI provider respects proxy settings | F-01 |

### Observability Scenarios

| ID | Scenario | Preconditions | Expected Result | Feature Ref |
|----|----------|---------------|-----------------|-------------|
| O-01 | Install log contains CAPI provider phase transitions | Normal install | Logs show provider deployment, machine creation, pivot phases with timestamps | F-08 |
| O-02 | Error logs contain actionable diagnostics | Install failure scenario (any N-xx) | Error output identifies root cause and suggests remediation | F-08 |
| O-03 | Must-gather includes CAPI resources | Completed or failed install | `oc adm must-gather` collects CAPI CRs, provider logs, events | F-08 |

### Non-Regression Scenarios

| ID | Scenario | Preconditions | Expected Result | Feature Ref |
|----|----------|---------------|-----------------|-------------|
| NR-01 | AWS IPI install unaffected | Standard AWS install-config (no CAPI stanza) | Install succeeds; no behavioral change | F-11 |
| NR-02 | GCP IPI install unaffected | Standard GCP install-config | Install succeeds; no behavioral change | F-11 |
| NR-03 | Azure IPI install unaffected | Standard Azure install-config | Install succeeds; no behavioral change | F-11 |
| NR-04 | Existing `platform: external` UPI unaffected | External platform without CAPI stanza | Install proceeds via existing UPI path; no regression | F-11 |
| NR-05 | vSphere IPI install unaffected | Standard vSphere install-config | Install succeeds; no behavioral change | F-11 |

---

## Traceability Matrix

| Requirement / Success Criterion | Scenario IDs |
|---------------------------------|-------------|
| End-to-end use of partner-supplied CAPI infrastructure provider | P-01, P-04, P-05, P-06 |
| Agreed delivery model (in-payload vs out-of-payload) | _Decision pending — no scenario assignable yet_ |
| Versioning and Red Hat/partner support boundary | _Decision pending — covered by partner integration contract review (testing task, not automated scenario)_ |
| Install-time credential handling without partner fork | P-03, S-01, S-02, S-03, S-04, N-02 |
| CI coverage for at least one external-platform CAPI install path | P-01 (proposed CI job TBD) |
| Oracle validation on OCI | P-01 (OCI as target) |
| Install-config validation | P-02, N-01, N-05, I-01 |
| CAPI provider lifecycle (deploy, invoke, teardown) | P-04, P-05, N-03, N-04 |
| Failure recovery and idempotency | FR-01, FR-02, FR-03, N-06 |
| Credential security | S-01, S-02, S-03, S-04 |
| Observability and diagnostics | O-01, O-02, O-03 |
| Non-regression of existing platforms | NR-01 through NR-05 |
| Day-2 machine management (autoscaling, MHC) | **Excluded** — [OCPSTRAT-1322](https://redhat.atlassian.net/browse/OCPSTRAT-1322) |
| HCP / managed-service topology | **Excluded** — [OPCT-314](https://redhat.atlassian.net/browse/OPCT-314) unsupported |

---

## Item Pass/Fail Criteria

> **Note:** Quantitative thresholds (pass rates, defect counts) are **TBD** pending implementation and CI setup. The criteria below are qualitative targets.

| Criterion | Pass | Fail |
|-----------|------|------|
| E2E install on OCI | P-01 passes on target OCP version | P-01 fails or cannot execute |
| Install-config validation | P-02, N-01, N-05 all pass | Any validation scenario fails |
| Credential security | All S-xx scenarios pass | Any credential leakage detected |
| Non-regression | All NR-xx scenarios pass in CI | Any existing platform install regresses |
| Destroy cleanup | P-06, FR-03, N-06 pass; no orphaned resources | Cloud resources remain after destroy |
| Critical / blocker defects | Zero open critical or blocker defects against SPLAT-2929 | Any critical/blocker open at exit |
| CI job stability | TBD — proposed: ≥N% pass rate over M runs (values TBD after CI setup) | Below threshold |

---

## Suspension and Resumption Criteria

### Suspension Criteria

- **Blocker from SPLAT-2902:** Research spike does not reach a design recommendation → testing cannot begin.
- **Partner CAPI provider unavailable:** OCI CAPI provider not available in a usable form → E2E testing blocked.
- **Critical installer regression:** Installer build is broken for all platforms → all testing suspended.
- **Infrastructure outage:** OCI test environment unavailable → E2E testing paused.

### Resumption Criteria

- Blocking design decisions are documented and communicated.
- Partner CAPI provider is available and functional in a test environment.
- Installer builds are green on at least one in-tree platform.
- Test infrastructure is restored and accessible.

---

## Test Deliverables

| Deliverable | Description | Status |
|-------------|-------------|--------|
| This test plan | IEEE 829-aligned test plan for SPLAT-2929 | Draft |
| Unit tests (installer) | Validation, manifest generation, credential propagation | Proposed — not yet authored |
| Integration tests | CAPI provider lifecycle orchestration | Proposed — not yet authored |
| E2E test suite | Full install/destroy on OCI | Proposed — not yet authored |
| CI job definition | Prow job(s) for external-platform CAPI E2E | Proposed — job name(s) TBD |
| Test execution report | Results from E2E test runs | Future — after implementation |
| Partner integration contract review | Manual review of documented partner contract | Future |
| Defect reports | Bugs filed during test execution | Future |

---

## Testing Tasks

| Task | Description | Jira Story | Status |
|------|-------------|-----------|--------|
| T-TASK-01 | Complete SPLAT-2902 research spike; produce design recommendation | [SPLAT-2902](https://redhat.atlassian.net/browse/SPLAT-2902) | In Progress |
| T-TASK-02 | Finalize test plan based on design decisions | TBD | Blocked on T-TASK-01 |
| T-TASK-03 | Author unit tests for install-config CAPI validation | TBD | Not started |
| T-TASK-04 | Author integration tests for CAPI provider lifecycle | TBD | Not started |
| T-TASK-05 | Author E2E test for OCI CAPI install (P-01) | TBD | Not started |
| T-TASK-06 | Configure Prow CI job for external-platform CAPI E2E | TBD | Not started |
| T-TASK-07 | Author credential security test scenarios (S-xx) | TBD | Not started |
| T-TASK-08 | Author destroy / cleanup test scenarios | TBD | Not started |
| T-TASK-09 | Validate non-regression on existing platforms | TBD | Not started — may leverage existing CI |
| T-TASK-10 | Manual exploratory testing of partner integration contract | TBD | Not started |
| T-TASK-11 | Oracle partner validation on OCI | TBD | Not started |

---

## Environmental Needs

### Hardware / Infrastructure

| Need | Description | Status |
|------|-------------|--------|
| OCI tenancy | Oracle Cloud Infrastructure account with sufficient quota for OpenShift cluster | TBD — partner coordination required |
| x86_64 compute | Machines for control plane and workers | TBD |
| aarch64 compute (optional) | For C-02 architecture compatibility | TBD — depends on OCI CAPI provider support |
| Mirror registry | For disconnected install testing (C-04) | TBD |
| Proxy environment | For proxy-configured install testing (C-05) | TBD |

### Software

| Need | Description | Status |
|------|-------------|--------|
| OpenShift installer build | With CAPI external-platform support | TBD — not yet implemented |
| Partner CAPI provider image | OCI infrastructure provider container image | TBD — version and source undecided |
| CAPI core controllers | `cluster-api-controller-manager` compatible with OCP | TBD |
| `oc` CLI | For post-install validation | Available |
| `openshift-install` CLI | Modified binary | TBD |
| Prow CI | For automated test execution | Available — job config TBD |

### Credentials / Accounts

| Need | Description | Status |
|------|-------------|--------|
| OCI API credentials | For CAPI provider to provision infrastructure | TBD — secure storage mechanism undecided |
| Pull secret | OCP pull secret for image access | Available via standard process |
| CI service account | For automated Prow job execution | TBD |

---

## Responsibilities

> **Note:** Named individuals are not assigned. Roles are described generically pending team allocation.

| Role | Responsibility |
|------|---------------|
| Feature Owner (SPLAT team) | Design decisions, implementation, unit tests |
| QE Lead (TBD) | Test plan review, E2E test authoring, CI job setup |
| Partner Engineering (Oracle) | OCI CAPI provider availability, partner validation (T-TASK-11) |
| CI/Infrastructure (TBD) | Prow job configuration, OCI test environment provisioning |
| Documentation (TBD) | Partner integration contract documentation review |
| Security (TBD) | Review of credential handling design (S-xx scenarios) |

---

## Staffing and Training Needs

| Need | Description | Status |
|------|-------------|--------|
| CAPI fundamentals | QE team needs familiarity with Cluster API architecture, CRDs, and provider model | TBD — training or knowledge transfer needed |
| OCI platform knowledge | Understanding of OCI compute, networking, and IAM for test environment setup | TBD |
| External-platform installer internals | Understanding of `platform=external` code path in `openshift/installer` | TBD — design not finalized |
| Partner CAPI provider specifics | Provider CRDs, configuration, and failure modes | TBD — depends on provider availability |

---

## Schedule

> **Note:** No committed dates exist. Version 5.1 is scoped to **research and planning**, not implementation. The milestones below are **relative dependencies**, not calendar dates.

| Milestone | Dependency | Status |
|-----------|-----------|--------|
| SPLAT-2902 research spike complete | None | In Progress |
| Design decisions documented (delivery model, credential mechanism, provider packaging) | SPLAT-2902 | Blocked |
| Test plan finalized (this document updated) | Design decisions | Blocked |
| Partner CAPI provider available for testing | Partner (Oracle) delivery | TBD |
| Unit / integration tests authored | Design decisions + implementation started | Not started |
| E2E test authored | Implementation + provider available | Not started |
| CI job configured and green | E2E test + CI infrastructure | Not started |
| Oracle partner validation | CI job + OCI environment | Not started |
| Test exit criteria met | All above | Not started |

---

## Risks and Contingencies

| ID | Risk | Likelihood | Impact | Mitigation / Contingency |
|----|------|-----------|--------|--------------------------|
| R-01 | SPLAT-2902 research spike does not converge on a design | Medium | High — all testing blocked | Timebox spike; escalate design decisions to architecture review |
| R-02 | Partner CAPI provider not available or unstable | Medium | High — E2E testing blocked | Identify a mock/stub provider for integration tests; accept reduced E2E coverage initially |
| R-03 | Delivery model (in-payload vs out-of-payload) undecided | High | Medium — affects packaging, CI, and disconnected testing | Document both paths in test plan; test whichever is chosen |
| R-04 | OCI test infrastructure unavailable or quota-limited | Medium | Medium — E2E testing delayed | Pre-arrange OCI tenancy and quotas; have fallback environment plan |
| R-05 | Credential handling mechanism changes late in development | Medium | Medium — security scenarios need rework | Decouple credential-format tests from mechanism; use interface-based testing |
| R-06 | CAPI provider API breaking changes between development and release | Low | High — E2E tests break | Pin provider version in CI; monitor upstream CAPI releases |
| R-07 | Non-regression CI jobs (NR-xx) flaky due to unrelated failures | Medium | Low — false signals | Use existing platform CI as baseline; only flag regressions correlated with CAPI changes |
| R-08 | HCP / managed-service scope creep into day-0 plan | Low | Medium — scope expansion | Hard boundary: HCP is N/A per OPCT-314; any HCP work requires separate plan and Jira epic |
| R-09 | Insufficient QE staffing or CAPI expertise | Medium | Medium — test development delayed | Invest in training (see Staffing section); consider cross-team pairing |

---

## Approvals

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Feature Owner | _TBD_ | _TBD_ | _TBD_ |
| QE Lead | _TBD_ | _TBD_ | _TBD_ |
| Engineering Manager | _TBD_ | _TBD_ | _TBD_ |
| Partner Engineering | _TBD_ | _TBD_ | _TBD_ |

> Approvals are pending. This plan is in **Draft** status.

---

## References (Full List)

1. [SPLAT-2929](https://redhat.atlassian.net/browse/SPLAT-2929) — [Day 0] Add Cluster API (CAPI) support for installing external platform clusters
2. [SPLAT-2902](https://redhat.atlassian.net/browse/SPLAT-2902) — Self-managed CAPI feasibility with external profile / local control plane
3. [OCPSTRAT-3724](https://redhat.atlassian.net/browse/OCPSTRAT-3724) — Strategic feature for external-platform CAPI support
4. [OCPSTRAT-1322](https://redhat.atlassian.net/browse/OCPSTRAT-1322) — Day-2 autoscaling / machine-health-check (excluded from this plan)
5. [OSDOCS-21999](https://redhat.atlassian.net/browse/OSDOCS-21999) — Documentation tracker linked from OCPSTRAT-3724
6. [OPCT-314](https://redhat.atlassian.net/browse/OPCT-314) — External platform on HCP currently unsupported
7. [kueue-operator feature-test-plan-template](https://github.com/openshift/kueue-operator/blob/main/docs/plans/templates/feature-test-plan-template.md) — OpenShift test plan template (style reference)
8. [kueue-operator hierarchical-cohorts plan](https://github.com/openshift/kueue-operator/blob/main/docs/plans/features/hierarchical-cohorts.md) — Example feature test plan
9. IEEE 829-2008 — IEEE Standard for Software and System Test Documentation
10. ISO/IEC/IEEE 29119 — Software and systems engineering — Software testing

---

## Glossary

| Term | Definition |
|------|-----------|
| CAPI | Cluster API — a Kubernetes sub-project providing declarative APIs for cluster creation, configuration, and management |
| CAPI Infrastructure Provider | A CAPI-compliant controller that provisions infrastructure on a specific cloud or platform (e.g., OCI) |
| Day 0 | Install-time operations: cluster provisioning, bootstrap, initial configuration |
| Day 2 | Post-install operations: scaling, upgrades, machine health checks, autoscaling |
| External Platform | OpenShift `platform=external` — a platform type where infrastructure is not managed by an in-tree provider |
| HCP | Hosted Control Plane — a topology where the control plane runs outside the data-plane cluster (e.g., HyperShift) |
| IPI | Installer-Provisioned Infrastructure — install mode where the installer manages cloud resource lifecycle |
| OCI | Oracle Cloud Infrastructure — the initial target partner cloud for this feature |
| OPCT | OpenShift Provider Certification Tool |
| Pivot | The bootstrap phase where control plane components transfer from the temporary bootstrap machine to the permanent cluster |
| UPI | User-Provisioned Infrastructure — install mode where the user pre-creates cloud resources |
| MHC | MachineHealthCheck — a Kubernetes/CAPI resource for automated machine remediation (day-2, excluded) |

---

_End of Test Plan TP-SPLAT-2929-001_
