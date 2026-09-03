# Test Plan: Migrate vSphere CI from IBM Cloud to AWS

> **DRAFT** — This test plan is a draft. Epic [SPLAT-2873](https://issues.redhat.com/browse/SPLAT-2873) is currently labelled **needs-refinement**. Target AWS topology, supported job inventory, migration window, rollback owner, and acceptance thresholds are **not yet specified**. All items marked **TBD** must be resolved before test execution begins.

| Field                | Value                                                     |
| ----------------------| -----------------------------------------------------------|
| **Epic**             | [SPLAT-2873](https://issues.redhat.com/browse/SPLAT-2873) |
| **Status**           | New / Unresolved                                          |
| **Document Version** | 0.1-draft                                                 |
| **Date**             | 2026-09-02                                                |
| **Standard**         | IEEE 829-2008                                             |

---

## 1. Test Plan Identifier

**TP-SPLAT-2873-v0.1**

This identifier uniquely references the test plan for the vSphere CI migration from IBM Cloud to AWS, corresponding to Jira epic [SPLAT-2873](https://issues.redhat.com/browse/SPLAT-2873).

---

## 2. Introduction

### 2.1 Purpose

This test plan defines the verification and validation strategy for migrating all vSphere CI workloads from the IBM Cloud-hosted build02 build farm to existing build farm clusters running on AWS, and for standing up the vSphere capacity manager on a dedicated AWS-hosted cluster. The plan ensures the migrated infrastructure meets functional, operational, and reliability requirements before the IBM Cloud environment is decommissioned.

### 2.2 Scope

The plan covers:

- Migration of all vSphere CI jobs from IBM Cloud build02 to existing build farm clusters.
- Installation and configuration of `vsphere-capacity-manager` and `vsphere-capacity-manager-vcenter-ctrl` on a dedicated AWS-hosted cluster.
- Updates to Boskos leases, cluster profiles, and `openshift/ci-tools` job dispatch logic.
- Validation that migrated jobs execute successfully on the new infrastructure.
- Decommissioning readiness of the IBM Cloud build02 environment.

### 2.3 Related Jira Issues

| Key | Title | Role |
|---|---|---|
| [SPLAT-2873](https://issues.redhat.com/browse/SPLAT-2873) | Migrate vSphere CI from IBM Cloud to AWS | Parent Epic |
| [SPLAT-2878](https://issues.redhat.com/browse/SPLAT-2878) | Migrate vSphere CI from IBM Cloud to AWS | Child — migration workstream |
| [SPLAT-2880](https://issues.redhat.com/browse/SPLAT-2880) | Install and configure vsphere-capacity-manager and vsphere-capacity-manager-vcenter-ctrl | Child — capacity manager |
| [SPLAT-2881](https://issues.redhat.com/browse/SPLAT-2881) | Research and update Boskos leases for new vSphere environment and cluster profile | Child — Boskos leases |
| [SPLAT-2882](https://issues.redhat.com/browse/SPLAT-2882) | Research changes and create new cluster profile as required for new vSphere environment | Child — cluster profile |
| [SPLAT-2883](https://issues.redhat.com/browse/SPLAT-2883) | Update all jobs to the new cluster profile | Child — job updates |
| [SPLAT-2884](https://issues.redhat.com/browse/SPLAT-2884) | Research and update openshift/ci-tools to dispatch vSphere jobs to existing build farm clusters | Child — job dispatch |

### 2.4 References

- IEEE 829-2008 Standard for Software and System Test Documentation
- OpenShift CI architecture documentation (openshift/release repository)
- vsphere-capacity-manager project documentation
- Boskos resource management documentation
- ci-tools job dispatch documentation (openshift/ci-tools repository)

### 2.5 Assumptions

1. The target AWS topology (regions, VPCs, cluster names) will be defined before test execution begins.
2. A complete inventory of vSphere CI jobs currently running on build02 will be available.
3. The existing build farm clusters have sufficient capacity for the migrated workloads, or capacity planning will be completed prior to migration.
4. vCenter credentials and network connectivity from the AWS-hosted cluster to vSphere infrastructure will be provisioned.
5. The vsphere-capacity-manager and vsphere-capacity-manager-vcenter-ctrl container images are available in the target registry.
6. Rollback to the IBM Cloud build02 environment is possible during the migration window.
7. CI job definitions, cluster profiles, and Boskos lease configurations are version-controlled in openshift/release.

---

## 3. Test Items

The following items are subject to testing:

| Item | Version / Ref | Source |
|---|---|---|
| vSphere CI jobs (migrated set) | TBD — full inventory required | openshift/release job definitions |
| `vsphere-capacity-manager` | TBD — target version | vsphere-capacity-manager repository |
| `vsphere-capacity-manager-vcenter-ctrl` | TBD — target version | vsphere-capacity-manager repository |
| Boskos lease configuration for new vSphere environment | TBD | openshift/release |
| New or updated cluster profile(s) | TBD | openshift/release |
| ci-tools job dispatch changes | TBD — PR/commit ref | openshift/ci-tools |
| AWS-hosted build farm cluster(s) | TBD — cluster identifiers | Infrastructure |
| AWS-hosted dedicated capacity-manager cluster | TBD — cluster identifier | Infrastructure |

---

## 4. Features to be Tested

### 4.1 Functional

- **FN-01**: vsphere-capacity-manager correctly provisions and reclaims vSphere resources on the new environment.
- **FN-02**: vsphere-capacity-manager-vcenter-ctrl communicates with vCenter and manages VMs on the new environment.
- **FN-03**: Boskos lease acquisition and release for vSphere resources works with the new lease definitions.
- **FN-04**: New or updated cluster profiles contain correct vSphere environment parameters.
- **FN-05**: ci-tools dispatches vSphere jobs to the correct build farm cluster(s) instead of build02.

### 4.2 Integration

- **IN-01**: End-to-end CI job execution using the new cluster profile, Boskos leases, and capacity manager.
- **IN-02**: Interaction between ci-tools dispatch logic and the target build farm cluster(s).
- **IN-03**: Capacity manager interaction with Boskos for resource lifecycle.

### 4.3 Operational

- **OP-01**: Observability — metrics, alerts, and dashboards for the new capacity manager and vSphere resources.
- **OP-02**: Logging — capacity manager and vCenter controller logs are collected and accessible.
- **OP-03**: Resource utilization under representative CI load.

### 4.4 Migration / Cutover

- **MG-01**: All identified vSphere jobs execute successfully on the new infrastructure.
- **MG-02**: No jobs remain dispatched to build02 after cutover.
- **MG-03**: Rollback procedure restores job dispatch to build02 if required.

### 4.5 Security / Access

- **SC-01**: vCenter credentials are stored securely (Kubernetes secrets or vault) on the new cluster.
- **SC-02**: RBAC on the capacity-manager cluster restricts access to authorized service accounts.
- **SC-03**: Network policies permit only required communication paths between the AWS cluster and vSphere infrastructure.

### 4.6 Decommissioning

- **DC-01**: IBM Cloud build02 environment can be safely decommissioned after successful migration validation.

---

## 5. Features Not to be Tested

The following are explicitly out of scope for this test plan:

- Performance benchmarking of vSphere CI jobs beyond pass/fail validation (performance optimization is a separate concern).
- Testing of CI jobs that do not target vSphere (e.g., AWS-native, GCP, bare-metal jobs).
- Upgrades to vSphere/vCenter infrastructure itself.
- Changes to the OpenShift CI platform unrelated to vSphere job dispatch or build farm migration.
- Disaster recovery of the AWS-hosted clusters (covered by separate infrastructure DR plans).
- Load testing or stress testing beyond representative job execution.

---

## 6. Approach

### 6.1 Test Strategy

Testing follows a phased approach aligned with the migration workstreams:

| Phase | Focus | Workstreams |
|---|---|---|
| **Phase 1 — Component Validation** | Validate individual components in isolation on the new infrastructure. | SPLAT-2880, SPLAT-2881, SPLAT-2882 |
| **Phase 2 — Integration Validation** | Validate end-to-end job execution with all components working together. | SPLAT-2883, SPLAT-2884 |
| **Phase 3 — Migration Execution** | Execute cutover; run representative jobs; verify rollback capability. | SPLAT-2878 |
| **Phase 4 — Decommissioning Readiness** | Confirm no residual dependencies on build02; validate decommission safety. | SPLAT-2873 (epic-level) |

### 6.2 Test Types

- **Smoke Tests**: Minimal set of representative vSphere CI jobs run on new infrastructure to confirm basic functionality.
- **Functional Tests**: Verify each component (capacity manager, Boskos leases, cluster profiles, job dispatch) behaves correctly.
- **Integration Tests**: End-to-end job execution through the full CI pipeline on new infrastructure.
- **Regression Tests**: Re-run the complete vSphere job inventory to confirm no regressions from migration.
- **Rollback Tests**: Validate that dispatch can be reverted to build02 and jobs resume successfully.
- **Security Validation**: Inspect credential storage, RBAC, and network policy configurations.

### 6.3 Test Execution Method

- Manual execution for infrastructure validation, security review, and decommissioning checks.
- Automated execution via Prow for CI job pass/fail validation (jobs are inherently self-testing).
- Comparison of job pass rates before and after migration using Sippy or equivalent CI analytics.

### 6.4 Test Data

- Test data consists of the existing vSphere CI job definitions and their required infrastructure resources.
- No synthetic test data is needed; the CI jobs themselves serve as the validation workload.

---

## 7. Item Pass/Fail Criteria

### 7.1 Overall Pass Criteria

The migration is considered successful when **all** of the following are true:

1. Every vSphere CI job identified in the migration inventory executes on the new infrastructure without infrastructure-related failures introduced by the migration.
2. The vsphere-capacity-manager correctly manages vSphere resource lifecycle (provision, monitor, reclaim) on the new cluster.
3. Boskos lease acquisition/release operates correctly with the new lease definitions.
4. ci-tools dispatches all vSphere jobs to the target build farm cluster(s) and none to build02.
5. Observability (metrics, logging, alerting) is functional for the new infrastructure.
6. Rollback procedure is validated and documented.
7. No residual hard dependencies on the IBM Cloud build02 environment remain.

### 7.2 Overall Fail Criteria

The migration is considered failed if **any** of the following are true:

1. A job that passed consistently on build02 fails repeatedly on the new infrastructure due to migration-introduced issues.
2. The capacity manager is unable to provision or reclaim vSphere resources.
3. Boskos leases cannot be acquired, resulting in job starvation.
4. Jobs are still dispatched to build02 after cutover.

### 7.3 Individual Test Pass/Fail

Each test scenario (Section 6.5) defines its own expected result. A test passes when the observed result matches the expected result. A test fails when the observed result deviates in a way attributable to the migration.

> **Note**: Quantitative pass-rate thresholds (e.g., "≥ X% of jobs must pass") are **TBD** — the epic does not specify acceptance thresholds. These must be defined during refinement before test execution.

---

## 8. Suspension Criteria and Resumption Requirements

### 8.1 Suspension Criteria

Testing shall be suspended if any of the following occur:

| ID | Condition |
|---|---|
| SUS-01 | The target AWS cluster(s) become unavailable or unhealthy. |
| SUS-02 | vCenter connectivity from the AWS cluster is lost. |
| SUS-03 | A critical defect in the capacity manager causes uncontrolled resource consumption or data loss. |
| SUS-04 | Security credentials are exposed or compromised during testing. |
| SUS-05 | More than TBD% of representative smoke-test jobs fail due to infrastructure issues. |
| SUS-06 | A blocking defect in ci-tools dispatch logic causes jobs to be routed to incorrect clusters. |

### 8.2 Resumption Requirements

Testing may resume when:

1. The root cause of the suspension is identified, documented, and either fixed or mitigated.
2. The environment is restored to a known-good state (clusters healthy, connectivity verified).
3. Any exposed credentials are rotated and secured.
4. The test lead (TBD) approves resumption.

---

## 9. Test Deliverables

| Deliverable | Format | Timing |
|---|---|---|
| This test plan | Markdown (this document) | Before test execution |
| Test scenario inventory | Traceability matrix (Section 16) | Before test execution |
| Component validation results (Phase 1) | Jira comments on child issues | During Phase 1 |
| Integration validation results (Phase 2) | CI job run links and Sippy reports | During Phase 2 |
| Migration execution report (Phase 3) | Summary document with job pass/fail data | After Phase 3 |
| Rollback test results | Documented procedure + execution evidence | During Phase 3 |
| Decommissioning readiness assessment (Phase 4) | Checklist with evidence | Before decommission |
| Defect reports | Jira issues linked to SPLAT-2873 | As discovered |
| Final test summary report | Markdown or Jira epic comment | After all phases |

---

## 10. Testing Tasks

| Task ID | Task | Dependencies | Phase |
|---|---|---|---|
| TT-01 | Obtain complete inventory of vSphere CI jobs on build02 | None | Pre-Phase 1 |
| TT-02 | Confirm AWS cluster(s) are provisioned and accessible | None | Pre-Phase 1 |
| TT-03 | Validate vsphere-capacity-manager installation (SPLAT-2880) | TT-02 | Phase 1 |
| TT-04 | Validate vsphere-capacity-manager-vcenter-ctrl installation (SPLAT-2880) | TT-02 | Phase 1 |
| TT-05 | Validate Boskos lease definitions (SPLAT-2881) | TT-03 | Phase 1 |
| TT-06 | Validate new cluster profile(s) (SPLAT-2882) | TT-02 | Phase 1 |
| TT-07 | Validate ci-tools dispatch changes (SPLAT-2884) | TT-02, TT-06 | Phase 1 |
| TT-08 | Run smoke-test subset of vSphere jobs end-to-end | TT-03 through TT-07 | Phase 2 |
| TT-09 | Run full vSphere job inventory on new infrastructure (SPLAT-2883) | TT-08 | Phase 2 |
| TT-10 | Compare pass rates (pre- vs. post-migration) | TT-09 | Phase 2 |
| TT-11 | Execute rollback procedure and validate | TT-08 | Phase 3 |
| TT-12 | Execute cutover — disable build02 dispatch | TT-09, TT-10 | Phase 3 |
| TT-13 | Verify no jobs routed to build02 post-cutover | TT-12 | Phase 3 |
| TT-14 | Validate observability (metrics, logs, alerts) | TT-03, TT-04 | Phase 2 |
| TT-15 | Validate security controls (credentials, RBAC, network) | TT-03, TT-04, TT-06 | Phase 2 |
| TT-16 | Decommissioning readiness assessment for build02 | TT-12, TT-13 | Phase 4 |

---

## 11. Environmental Needs

### 11.1 Infrastructure

| Component | Requirement | Status |
|---|---|---|
| AWS-hosted build farm cluster(s) | Existing clusters with capacity for vSphere CI jobs | TBD — clusters to be identified |
| AWS-hosted dedicated cluster for capacity manager | Cluster provisioned for vsphere-capacity-manager | TBD — to be provisioned |
| vSphere/vCenter environment | Accessible from AWS-hosted clusters; credentials provisioned | TBD |
| Network connectivity | AWS cluster ↔ vCenter, AWS cluster ↔ CI control plane | TBD |
| Container registry | Images for capacity manager components available | TBD |
| IBM Cloud build02 | Available for rollback testing during migration window | Existing |

### 11.2 Tools

| Tool | Purpose |
|---|---|
| Prow | CI job execution and reporting |
| Sippy (or equivalent) | Job pass-rate analytics and comparison |
| Boskos | vSphere resource lease management |
| `oc` / `kubectl` | Cluster inspection and validation |
| ci-tools | Job dispatch verification |

### 11.3 Access Requirements

- Cluster admin or equivalent access to the capacity-manager cluster (for installation validation).
- Read access to vSphere/vCenter (for resource verification).
- Access to Prow dashboards and job logs.
- Access to Sippy or equivalent for pass-rate comparison.
- Access to openshift/release and openshift/ci-tools repositories.

---

## 12. Responsibilities

| Role | Responsibility | Assignee |
|---|---|---|
| Test Lead | Owns test plan execution, reports status, manages defects | TBD |
| Migration Lead | Coordinates migration execution and cutover decisions | TBD |
| Infrastructure Engineer | Provisions and maintains AWS clusters, network, credentials | TBD |
| Capacity Manager Developer | Supports capacity-manager installation and troubleshooting | TBD |
| CI Platform Engineer | Supports ci-tools dispatch changes and Prow configuration | TBD |
| Rollback Owner | Executes rollback procedure if suspension criteria are met | TBD |
| Security Reviewer | Validates credential storage, RBAC, and network policies | TBD |

> **Note**: Assignees are TBD. The epic does not specify owners; these must be assigned during refinement.

---

## 13. Staffing and Training Needs

### 13.1 Staffing

- A minimum of one engineer with vSphere CI infrastructure experience is required for test execution.
- A minimum of one engineer with capacity-manager operational experience is required for Phase 1 validation.
- Additional staffing needs are TBD pending migration scope and schedule.

### 13.2 Training

| Topic | Audience | Status |
|---|---|---|
| vsphere-capacity-manager operations and troubleshooting | Test executors | TBD |
| New cluster profile structure and parameters | CI platform engineers | TBD |
| Rollback procedure execution | Rollback owner | TBD |
| AWS cluster access and tooling | All test participants | TBD |

---

## 14. Schedule

> **Note**: No dates are specified in the epic. The schedule below is a relative ordering; actual dates are **TBD**.

| Milestone | Relative Timing | Status |
|---|---|---|
| Test plan approval | Before Phase 1 | Draft |
| Job inventory complete (TT-01) | Before Phase 1 | TBD |
| AWS clusters provisioned (TT-02) | Before Phase 1 | TBD |
| Phase 1 — Component Validation complete | TBD | Not started |
| Phase 2 — Integration Validation complete | After Phase 1 | Not started |
| Phase 3 — Migration Execution / Cutover | After Phase 2 | Not started |
| Phase 4 — Decommissioning Readiness confirmed | After Phase 3 | Not started |
| IBM Cloud build02 decommissioned | After Phase 4 | Not started |

---

## 15. Risks and Contingencies

| ID | Risk | Likelihood | Impact | Mitigation / Contingency |
|---|---|---|---|---|
| R-01 | Target AWS cluster(s) have insufficient capacity for migrated vSphere CI workload | TBD | High | Perform capacity planning before Phase 2; monitor resource utilization during smoke tests |
| R-02 | Network latency or connectivity issues between AWS and vCenter degrade job performance or cause timeouts | TBD | High | Validate network path early in Phase 1; establish latency baselines |
| R-03 | Capacity manager has compatibility issues with the new vSphere/vCenter version or topology | TBD | High | Test capacity manager in isolation during Phase 1 before integration |
| R-04 | Incomplete job inventory causes some vSphere jobs to remain on build02 after cutover | TBD | Medium | Cross-reference Prow job definitions with build02 dispatch logs; verify no residual traffic post-cutover (TT-13) |
| R-05 | ci-tools dispatch changes introduce regressions for non-vSphere jobs | TBD | High | Ensure dispatch changes are scoped to vSphere jobs; run non-vSphere job regression checks |
| R-06 | Rollback to build02 is needed but build02 has been partially decommissioned | TBD | Critical | Do not begin build02 decommissioning until Phase 4 is complete; maintain rollback capability throughout migration window |
| R-07 | Boskos lease contention increases due to changed resource topology | TBD | Medium | Monitor lease acquisition times; compare with pre-migration baselines |
| R-08 | Credential rotation or expiry during migration window causes job failures | TBD | Medium | Document credential lifecycle; ensure credentials are valid for the entire migration window |
| R-09 | Migration window is too short to validate the full job inventory | TBD | Medium | Prioritize representative job subset for smoke testing; run full inventory in parallel if possible |
| R-10 | Epic is under-refined (needs-refinement label); acceptance criteria may change | High | Medium | Treat this plan as a draft; re-baseline after refinement |

---

## 16. Approvals

| Role | Name | Date | Signature |
|---|---|---|---|
| Test Lead | TBD | TBD | |
| Migration Lead | TBD | TBD | |
| Engineering Manager | TBD | TBD | |
| Product Owner | TBD | TBD | |

> **Note**: This document is a **draft** and has not been approved. Approval is required before test execution begins.

---

## Appendix A: Test Scenarios

Each scenario is identified by a unique ID and is traceable to a feature (Section 4) and workstream (Section 2.3).

---

### TS-CM-01: Capacity Manager Installation

| Field | Value |
|---|---|
| **ID** | TS-CM-01 |
| **Objective** | Verify that vsphere-capacity-manager installs and starts successfully on the dedicated AWS cluster. |
| **Traces To** | FN-01, SPLAT-2880 |
| **Setup / Data** | Dedicated AWS cluster provisioned; capacity-manager container image available in target registry; vCenter credentials stored as Kubernetes secret. |
| **Steps** | 1. Deploy vsphere-capacity-manager to the target cluster using the documented installation method. 2. Verify the deployment reaches Ready state. 3. Inspect pod logs for startup errors. 4. Confirm the manager connects to vCenter. |
| **Expected Result** | Deployment is Ready; pods are Running with no error-level log entries; vCenter connection is established. |
| **Evidence** | `oc get deployment`, `oc logs`, vCenter connection log entries. |

---

### TS-CM-02: vCenter Controller Installation

| Field | Value |
|---|---|
| **ID** | TS-CM-02 |
| **Objective** | Verify that vsphere-capacity-manager-vcenter-ctrl installs and starts successfully. |
| **Traces To** | FN-02, SPLAT-2880 |
| **Setup / Data** | Same as TS-CM-01. |
| **Steps** | 1. Deploy vsphere-capacity-manager-vcenter-ctrl to the target cluster. 2. Verify the deployment reaches Ready state. 3. Inspect pod logs for startup errors. 4. Confirm the controller communicates with vCenter. |
| **Expected Result** | Deployment is Ready; pods are Running; vCenter communication is functional. |
| **Evidence** | `oc get deployment`, `oc logs`, controller reconciliation logs. |

---

### TS-CM-03: Capacity Manager Resource Provisioning

| Field | Value |
|---|---|
| **ID** | TS-CM-03 |
| **Objective** | Verify that the capacity manager provisions vSphere resources on demand in the new environment. |
| **Traces To** | FN-01, FN-02, SPLAT-2880 |
| **Setup / Data** | Capacity manager and vCenter controller running (TS-CM-01, TS-CM-02 passed); Boskos lease definitions configured. |
| **Steps** | 1. Trigger a resource request (via Boskos lease acquisition or manual CR creation). 2. Observe the capacity manager creating the requested vSphere resources. 3. Verify the resources appear in vCenter. |
| **Expected Result** | Resources are provisioned in vCenter within the expected time frame; capacity manager logs confirm successful provisioning. |
| **Evidence** | Capacity manager logs, vCenter inventory, Boskos lease state. |

---

### TS-CM-04: Capacity Manager Resource Reclamation

| Field | Value |
|---|---|
| **ID** | TS-CM-04 |
| **Objective** | Verify that the capacity manager reclaims vSphere resources when released. |
| **Traces To** | FN-01, FN-02, SPLAT-2880 |
| **Setup / Data** | Resources provisioned via TS-CM-03. |
| **Steps** | 1. Release the Boskos lease (or delete the resource CR). 2. Observe the capacity manager cleaning up vSphere resources. 3. Verify the resources are removed from vCenter. |
| **Expected Result** | Resources are cleaned up from vCenter; capacity manager logs confirm successful reclamation. |
| **Evidence** | Capacity manager logs, vCenter inventory. |

---

### TS-BK-01: Boskos Lease Acquisition

| Field | Value |
|---|---|
| **ID** | TS-BK-01 |
| **Objective** | Verify that CI jobs can acquire Boskos leases for vSphere resources using the new lease definitions. |
| **Traces To** | FN-03, SPLAT-2881 |
| **Setup / Data** | New Boskos lease definitions deployed to the CI control plane; capacity manager running. |
| **Steps** | 1. Submit a CI job that requires a vSphere Boskos lease. 2. Observe the job acquiring a lease. 3. Verify the lease resource type and parameters match the new definitions. |
| **Expected Result** | Lease is acquired without timeout; resource parameters are correct for the new environment. |
| **Evidence** | Prow job logs showing lease acquisition, Boskos lease state. |

---

### TS-BK-02: Boskos Lease Release

| Field | Value |
|---|---|
| **ID** | TS-BK-02 |
| **Objective** | Verify that Boskos leases are correctly released after CI job completion. |
| **Traces To** | FN-03, SPLAT-2881 |
| **Setup / Data** | Lease acquired via TS-BK-01. |
| **Steps** | 1. Allow the CI job to complete (pass or fail). 2. Verify the Boskos lease is released. 3. Verify the underlying vSphere resources are reclaimed (via capacity manager). |
| **Expected Result** | Lease is released; resources are reclaimed; lease is available for reuse. |
| **Evidence** | Boskos lease state, capacity manager logs. |

---

### TS-CP-01: Cluster Profile Validation

| Field | Value |
|---|---|
| **ID** | TS-CP-01 |
| **Objective** | Verify that the new or updated cluster profile(s) contain correct parameters for the new vSphere environment. |
| **Traces To** | FN-04, SPLAT-2882 |
| **Setup / Data** | New cluster profile merged to openshift/release. |
| **Steps** | 1. Inspect the cluster profile configuration for correctness (vCenter URL, datacenter, datastore, network, credentials references). 2. Verify the profile is syntactically valid by running ci-operator config validation (if available). 3. Confirm the profile is referenced by at least one vSphere CI job definition. |
| **Expected Result** | Profile parameters match the new vSphere environment; validation passes; profile is referenced by jobs. |
| **Evidence** | Cluster profile file contents, validation command output, job definition references. |

---

### TS-JD-01: Job Dispatch to Target Cluster

| Field | Value |
|---|---|
| **ID** | TS-JD-01 |
| **Objective** | Verify that ci-tools dispatches vSphere jobs to the target build farm cluster(s) instead of build02. |
| **Traces To** | FN-05, SPLAT-2884 |
| **Setup / Data** | ci-tools dispatch changes deployed; target build farm cluster(s) configured. |
| **Steps** | 1. Submit a vSphere CI job via Prow. 2. Inspect the job's execution context to determine which build farm cluster it was dispatched to. 3. Confirm the job was NOT dispatched to build02. |
| **Expected Result** | Job executes on the target build farm cluster; no dispatch to build02. |
| **Evidence** | Prow job metadata (cluster field), ci-tools dispatch logs. |

---

### TS-JD-02: Non-vSphere Job Dispatch Unaffected

| Field | Value |
|---|---|
| **ID** | TS-JD-02 |
| **Objective** | Verify that dispatch changes do not affect non-vSphere CI jobs. |
| **Traces To** | FN-05, SPLAT-2884 |
| **Setup / Data** | ci-tools dispatch changes deployed. |
| **Steps** | 1. Submit a representative non-vSphere CI job (e.g., AWS or GCP platform job). 2. Verify the job is dispatched to its expected cluster (unchanged from pre-migration behavior). |
| **Expected Result** | Non-vSphere jobs continue to be dispatched to their pre-migration target clusters. |
| **Evidence** | Prow job metadata, comparison with pre-migration dispatch behavior. |

---

### TS-E2E-01: Smoke Test — Representative Job Execution

| Field | Value |
|---|---|
| **ID** | TS-E2E-01 |
| **Objective** | Verify that a small representative subset of vSphere CI jobs passes end-to-end on the new infrastructure. |
| **Traces To** | MG-01, IN-01, SPLAT-2878 |
| **Setup / Data** | All Phase 1 components validated; representative job subset identified (TBD). |
| **Steps** | 1. Identify a representative subset of vSphere CI jobs (TBD — selection criteria to be defined). 2. Trigger each job on the new infrastructure. 3. Verify each job completes successfully. 4. Inspect job logs for infrastructure-related warnings or errors. |
| **Expected Result** | All representative jobs pass; no infrastructure-related errors attributable to the migration. |
| **Evidence** | Prow job run links, pass/fail results, log excerpts. |

---

### TS-E2E-02: Full Job Inventory Execution

| Field | Value |
|---|---|
| **ID** | TS-E2E-02 |
| **Objective** | Verify that the complete inventory of vSphere CI jobs executes on the new infrastructure. |
| **Traces To** | MG-01, SPLAT-2883 |
| **Setup / Data** | Smoke tests passed (TS-E2E-01); all jobs updated to new cluster profile (SPLAT-2883). |
| **Steps** | 1. Run the complete set of vSphere CI jobs on the new infrastructure. 2. Compare pass rates with pre-migration baselines from Sippy or equivalent. 3. Investigate any new failures for migration-related root causes. |
| **Expected Result** | Pass rate is comparable to pre-migration baseline (threshold TBD); no new failures attributable to migration. |
| **Evidence** | Sippy reports (before and after), Prow job run links, failure investigation notes. |

---

### TS-RB-01: Rollback Procedure Validation

| Field | Value |
|---|---|
| **ID** | TS-RB-01 |
| **Objective** | Verify that vSphere CI jobs can be reverted to run on build02 if the migration fails. |
| **Traces To** | MG-03 |
| **Setup / Data** | Jobs currently dispatched to new infrastructure; build02 still operational. |
| **Steps** | 1. Execute the documented rollback procedure (TBD — procedure to be defined). 2. Verify ci-tools dispatches vSphere jobs back to build02. 3. Run a representative vSphere CI job and confirm it executes on build02. |
| **Expected Result** | Dispatch reverts to build02; representative job passes on build02. |
| **Evidence** | Prow job metadata showing build02 dispatch, job pass result. |

---

### TS-CUT-01: Cutover — No Residual Build02 Traffic

| Field | Value |
|---|---|
| **ID** | TS-CUT-01 |
| **Objective** | Verify that no vSphere CI jobs are dispatched to build02 after cutover. |
| **Traces To** | MG-02 |
| **Setup / Data** | Cutover executed (build02 dispatch disabled for vSphere jobs). |
| **Steps** | 1. Monitor Prow job dispatch for a defined observation period (TBD). 2. Query job metadata for any vSphere jobs dispatched to build02. 3. Confirm zero vSphere jobs on build02 during the observation period. |
| **Expected Result** | Zero vSphere CI jobs dispatched to build02 during the observation period. |
| **Evidence** | Prow/Sippy query results, dispatch logs. |

---

### TS-OBS-01: Observability Validation

| Field | Value |
|---|---|
| **ID** | TS-OBS-01 |
| **Objective** | Verify that metrics, logs, and alerts for the capacity manager and vSphere resources are functional. |
| **Traces To** | OP-01, OP-02 |
| **Setup / Data** | Capacity manager running; monitoring stack configured on the target cluster. |
| **Steps** | 1. Verify capacity manager exposes Prometheus metrics. 2. Verify metrics are scraped and visible in the monitoring dashboard. 3. Verify capacity manager and vCenter controller logs are collected by the cluster logging stack. 4. Trigger a test alert condition (if alerting rules are defined) and verify the alert fires. |
| **Expected Result** | Metrics visible in dashboards; logs collected and queryable; alerts fire when conditions are met. |
| **Evidence** | Dashboard screenshots or queries, log query results, alert notifications. |

---

### TS-SEC-01: Security Controls Validation

| Field | Value |
|---|---|
| **ID** | TS-SEC-01 |
| **Objective** | Verify that credentials, RBAC, and network policies are correctly configured on the capacity-manager cluster. |
| **Traces To** | SC-01, SC-02, SC-03 |
| **Setup / Data** | Capacity-manager cluster provisioned and configured. |
| **Steps** | 1. Verify vCenter credentials are stored as Kubernetes Secrets (not in plaintext ConfigMaps or environment variables). 2. Verify RBAC rules restrict capacity-manager namespace access to authorized service accounts. 3. Inspect NetworkPolicy resources for expected ingress/egress rules. 4. Attempt an unauthorized operation (e.g., access capacity-manager namespace from an unprivileged service account) and confirm it is denied. |
| **Expected Result** | Credentials are in Secrets; RBAC denies unauthorized access; NetworkPolicies enforce expected traffic rules. |
| **Evidence** | `oc get secret`, `oc get rolebinding`, `oc get networkpolicy`, unauthorized access attempt output. |

---

### TS-DC-01: Decommissioning Readiness

| Field | Value |
|---|---|
| **ID** | TS-DC-01 |
| **Objective** | Confirm that the IBM Cloud build02 environment has no residual dependencies and can be safely decommissioned. |
| **Traces To** | DC-01, SPLAT-2873 |
| **Setup / Data** | All migration phases complete; observation period (TBD) elapsed since cutover. |
| **Steps** | 1. Verify no vSphere CI jobs have been dispatched to build02 since cutover (TS-CUT-01). 2. Verify no other workloads depend on build02 infrastructure. 3. Verify all vSphere resources previously managed by build02 are now managed by the new capacity manager. 4. Verify DNS, credentials, and configuration references to build02 are removed or redirected. 5. Document decommissioning recommendation. |
| **Expected Result** | No dependencies on build02 remain; decommissioning is safe to proceed. |
| **Evidence** | Dependency audit results, configuration audit, decommissioning recommendation document. |

---

## Appendix B: Requirements-to-Test Traceability Matrix

| Requirement ID | Description | Test Scenarios | Workstream |
|---|---|---|---|
| FN-01 | Capacity manager provisions/reclaims vSphere resources | TS-CM-01, TS-CM-03, TS-CM-04 | SPLAT-2880 |
| FN-02 | vCenter controller manages VMs | TS-CM-02, TS-CM-03, TS-CM-04 | SPLAT-2880 |
| FN-03 | Boskos lease acquisition/release works | TS-BK-01, TS-BK-02 | SPLAT-2881 |
| FN-04 | Cluster profiles contain correct parameters | TS-CP-01 | SPLAT-2882 |
| FN-05 | ci-tools dispatches to correct clusters | TS-JD-01, TS-JD-02 | SPLAT-2884 |
| IN-01 | End-to-end job execution | TS-E2E-01, TS-E2E-02 | SPLAT-2883 |
| IN-02 | ci-tools ↔ build farm interaction | TS-JD-01 | SPLAT-2884 |
| IN-03 | Capacity manager ↔ Boskos interaction | TS-CM-03, TS-CM-04, TS-BK-01, TS-BK-02 | SPLAT-2880, SPLAT-2881 |
| OP-01 | Metrics and alerts functional | TS-OBS-01 | SPLAT-2880 |
| OP-02 | Logs collected and accessible | TS-OBS-01 | SPLAT-2880 |
| OP-03 | Resource utilization under load | TS-E2E-02 | SPLAT-2883 |
| MG-01 | All jobs execute on new infrastructure | TS-E2E-01, TS-E2E-02 | SPLAT-2878 |
| MG-02 | No jobs on build02 after cutover | TS-CUT-01 | SPLAT-2878 |
| MG-03 | Rollback procedure validated | TS-RB-01 | SPLAT-2878 |
| SC-01 | Credentials stored securely | TS-SEC-01 | SPLAT-2880 |
| SC-02 | RBAC restricts access | TS-SEC-01 | SPLAT-2880 |
| SC-03 | Network policies enforced | TS-SEC-01 | SPLAT-2880 |
| DC-01 | Build02 decommissioning safe | TS-DC-01 | SPLAT-2873 |

---

## Appendix C: Entry and Exit Criteria

### Entry Criteria (must be satisfied before test execution begins)

| ID | Criterion | Status |
|---|---|---|
| EC-01 | This test plan is approved (Section 16). | TBD |
| EC-02 | Complete inventory of vSphere CI jobs on build02 is available. | TBD |
| EC-03 | Target AWS cluster(s) are provisioned, accessible, and healthy. | TBD |
| EC-04 | vCenter connectivity from AWS cluster(s) is verified. | TBD |
| EC-05 | Capacity manager and vCenter controller images are available. | TBD |
| EC-06 | Boskos lease definitions, cluster profiles, and ci-tools changes are code-reviewed and ready to merge. | TBD |
| EC-07 | Rollback procedure is documented. | TBD |
| EC-08 | Pre-migration job pass-rate baseline is captured from Sippy or equivalent. | TBD |
| EC-09 | All roles in Section 12 are assigned. | TBD |
| EC-10 | Acceptance thresholds (pass-rate, observation period) are defined. | TBD |

### Exit Criteria (must be satisfied before declaring migration complete)

| ID | Criterion | Status |
|---|---|---|
| EX-01 | All test scenarios in Appendix A have been executed and passed. | TBD |
| EX-02 | Post-migration job pass rate meets the defined acceptance threshold. | TBD |
| EX-03 | Rollback procedure has been validated (TS-RB-01). | TBD |
| EX-04 | No vSphere jobs dispatched to build02 during post-cutover observation period (TS-CUT-01). | TBD |
| EX-05 | Decommissioning readiness confirmed (TS-DC-01). | TBD |
| EX-06 | All critical and high-severity defects are resolved or have approved waivers. | TBD |
| EX-07 | Final test summary report is published. | TBD |

---

## Appendix D: Glossary

| Term | Definition |
|---|---|
| **build02** | The IBM Cloud-hosted build farm cluster currently running vSphere CI jobs. |
| **Boskos** | Resource management system used by OpenShift CI to manage leases on shared infrastructure resources. |
| **Capacity Manager** | The vsphere-capacity-manager component that provisions and reclaims vSphere resources. |
| **ci-tools** | The openshift/ci-tools codebase containing CI job dispatch logic. |
| **Cluster Profile** | A set of configuration parameters in openshift/release defining how CI jobs interact with a target platform (e.g., vSphere). |
| **Prow** | Kubernetes-based CI/CD system used by OpenShift CI. |
| **Sippy** | CI analytics platform for OpenShift job pass-rate tracking. |
| **vCenter** | VMware vCenter Server, the management platform for vSphere. |

---

*End of test plan TP-SPLAT-2873-v0.1*
