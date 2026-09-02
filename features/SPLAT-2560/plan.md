# Test Plan: Cluster API for OpenShift Installations on vSphere

## IEEE 829 Test Plan Document

---

| Field            | Value                  |
| ------------------| ------------------------|
| **Test Plan ID** | TP-SPLAT-2560-001      |
| **Version**      | 0.1 (Draft)            |
| **Status**       | Draft — Pending Review |
| **Epic**         | SPLAT-2560             |
| **Priority**     | Critical               |
| **Created**      | 2026-09-02             |
| **Last Updated** | 2026-09-02             |
| **Author**       | N/A                    |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [References](#2-references)
3. [Test Items](#3-test-items)
4. [Features to Be Tested](#4-features-to-be-tested)
5. [Features Not to Be Tested](#5-features-not-to-be-tested)
6. [Test Approach and Levels](#6-test-approach-and-levels)
7. [Requirements Traceability Matrix](#7-requirements-traceability-matrix)
8. [Test Scenarios and Cases](#8-test-scenarios-and-cases)
9. [Suspension and Resumption Criteria](#9-suspension-and-resumption-criteria)
10. [Test Deliverables](#10-test-deliverables)
11. [Test Tasks](#11-test-tasks)
12. [Environment Needs](#12-environment-needs)
13. [Responsibilities](#13-responsibilities)
14. [Staffing and Training](#14-staffing-and-training)
15. [Schedule](#15-schedule)
16. [Risks and Contingencies](#16-risks-and-contingencies)
17. [Assumptions and Open Questions](#17-assumptions-and-open-questions)
18. [Approvals](#18-approvals)

---

## 1. Introduction

### 1.1 Purpose

This document defines the test plan for **SPLAT-2560: [dev preview] Introduce Cluster API for OpenShift Installations on vSphere**. It describes the scope, approach, resources, and schedule of testing activities required to validate the replacement of Machine API (MAPI) provisioning with Cluster API (CAPI) using the official Cluster API Provider for vSphere (CAPV).

This plan covers OCP clusters on supported VMware vSphere and VMware Cloud Foundation (VCF) platforms, spanning both day-0 installation and day-2 lifecycle management.

### 1.2 Objectives

- Validate that new CAPI-based OCP cluster installations on vSphere/VCF function correctly end-to-end.
- Verify a reliable upgrade path from existing MAPI-based clusters to CAPI-managed clusters.
- Confirm functional parity between the new CAPI-based solution and the existing MAPI solution.
- Measure and compare provision, scale, and upgrade performance against the existing MAPI baseline.
- Validate that reliability meets or exceeds the current MAPI solution.
- Confirm removal of the Terraform dependency and alignment with upstream CAPI.
- Ensure no unnecessary user-facing complexity is introduced.
- Validate related capabilities: Control Plane Machine Set, Static IP, and Multi-NIC.
- Verify migration rollback capabilities when MAPI-to-CAPI upgrade fails.

### 1.3 Scope

**In scope:**

- Day-0 CAPI-based installation on vSphere and VCF
- Day-2 lifecycle management (scale, upgrade, remediation)
- MAPI-to-CAPI migration/upgrade path
- Migration rollback
- Control Plane Machine Set integration
- Static IP support
- Multi-NIC support
- Performance benchmarking vs. MAPI baseline
- Reliability validation

**Out of scope:**

- Items explicitly marked TBD in the SPLAT-2560 epic (out-of-scope list pending)
- Non-vSphere/VCF platforms (AWS, Azure, bare metal, etc.)
- CAPV target version — TBD per epic

> **Note:** The SPLAT-2560 epic records that work is currently on hold / dropped in priority. This test plan will require revision once work resumes and scope is finalized.

---

## 2. References

### 2.1 Jira

| ID | Title | Link |
|----|-------|------|
| SPLAT-2560 | [dev preview] Introduce Cluster API for OpenShift Installations on vSphere | [SPLAT-2560](https://redhat.atlassian.net/browse/SPLAT-2560) |
| SPLAT-2410 | MAPI-to-CAPI Upgrade Story | [SPLAT-2410](https://redhat.atlassian.net/browse/SPLAT-2410) |

### 2.2 GitHub Pull Requests

| Repository | PR | Link |
|------------|----|------|
| openshift/cluster-capi-operator | #455 | [PR 455](https://github.com/openshift/cluster-capi-operator/pull/455) |
| openshift/cluster-capi-operator | #465 | [PR 465](https://github.com/openshift/cluster-capi-operator/pull/465) |
| openshift/release | #75003 | [PR 75003](https://github.com/openshift/release/pull/75003) |
| openshift/cluster-api-provider-vsphere | #83 | [PR 83](https://github.com/openshift/cluster-api-provider-vsphere/pull/83) |
| openshift/api | #2724 | [PR 2724](https://github.com/openshift/api/pull/2724) |

### 2.3 External Documentation

| Document | Link |
|----------|------|
| Cluster API Quick Start Guide | [cluster-api.sigs.k8s.io](https://cluster-api.sigs.k8s.io/user/quick-start.html) |

### 2.4 Standards

| Standard | Description |
|----------|-------------|
| IEEE 829-2008 | Standard for Software and System Test Documentation |

---

## 3. Test Items

The following components and artifacts are subject to testing:

| Item ID | Component | Description | Source |
|---------|-----------|-------------|--------|
| TI-01 | cluster-capi-operator | Operator managing CAPI lifecycle within OCP clusters | [cluster-capi-operator](https://github.com/openshift/cluster-capi-operator) |
| TI-02 | cluster-api-provider-vsphere (CAPV) | Official Cluster API provider for vSphere infrastructure | [cluster-api-provider-vsphere](https://github.com/openshift/cluster-api-provider-vsphere) |
| TI-03 | OpenShift API changes | API additions/modifications to support CAPI integration | [openshift/api PR #2724](https://github.com/openshift/api/pull/2724) |
| TI-04 | OpenShift Installer | Installation workflow changes for CAPI-based provisioning on vSphere | TBD |
| TI-05 | CI/Release configuration | CI job definitions and release gating for CAPI vSphere | [openshift/release PR #75003](https://github.com/openshift/release/pull/75003) |
| TI-06 | MAPI-to-CAPI migration tooling | Migration/upgrade path components and procedures | Per SPLAT-2410 |
| TI-07 | Control Plane Machine Set (CPMS) | Integration of CPMS with CAPI-managed control plane | TBD |

---

## 4. Features to Be Tested

| Feature ID | Feature | Description | Source Requirement |
|------------|---------|-------------|--------------------|
| FT-01 | New CAPI-based installation (day-0) | Fresh OCP cluster installation on vSphere/VCF using CAPI/CAPV instead of MAPI/Terraform | SPLAT-2560 |
| FT-02 | MAPI-to-CAPI upgrade | Migration of existing MAPI-provisioned clusters to CAPI management | SPLAT-2560, SPLAT-2410 |
| FT-03 | Migration rollback | Ability to revert a failed MAPI-to-CAPI migration | SPLAT-2410 |
| FT-04 | Day-2 lifecycle — node scaling | Scale worker and infrastructure nodes up/down via CAPI | SPLAT-2560 |
| FT-05 | Day-2 lifecycle — cluster upgrade | OCP version upgrade on CAPI-managed clusters | SPLAT-2560 |
| FT-06 | Day-2 lifecycle — node remediation | Automated and manual node remediation under CAPI | SPLAT-2560 |
| FT-07 | Functional parity with MAPI | All MAPI-supported operations available via CAPI | SPLAT-2560 |
| FT-08 | Control Plane Machine Set | CPMS functionality on CAPI-managed clusters | SPLAT-2560 |
| FT-09 | Static IP support | Static IP assignment for nodes during CAPI provisioning | SPLAT-2560 |
| FT-10 | Multi-NIC support | Multiple network interface support for CAPI-provisioned nodes | SPLAT-2560 |
| FT-11 | Terraform dependency removal | No Terraform binaries or providers required in the install workflow | SPLAT-2560 |
| FT-12 | Performance — provision/scale/upgrade | Performance comparable to or better than MAPI baseline | SPLAT-2560 |
| FT-13 | Reliability | Reliability at least equal to current MAPI solution | SPLAT-2560 |
| FT-14 | User complexity | No unnecessary user-facing complexity introduced | SPLAT-2560 |
| FT-15 | Zero-downtime migration | No cluster downtime during MAPI-to-CAPI migration | SPLAT-2410 |

---

## 5. Features Not to Be Tested

| Feature | Reason |
|---------|--------|
| Non-vSphere/VCF platforms (AWS, Azure, GCP, bare metal, etc.) | Out of scope — SPLAT-2560 targets vSphere/VCF only |
| MAPI provisioning regression | Covered by existing MAPI test suites; not part of this plan |
| CAPV internals / upstream CAPI conformance | Upstream project responsibility; OCP testing validates integration only |
| Specific CAPV version validation | CAPV target version is TBD per SPLAT-2560 |
| Out-of-scope items per SPLAT-2560 | Explicitly TBD in the epic; this section will be updated when defined |

---

## 6. Test Approach and Levels

### 6.1 Test Levels

| Level | Description | Applicability |
|-------|-------------|---------------|
| Unit | Automated tests within each component repository | TI-01, TI-02, TI-03 |
| Integration | Component interaction tests (operator + CAPV + vSphere API) | TI-01, TI-02, TI-04 |
| System / E2E | Full cluster lifecycle tests on real vSphere/VCF infrastructure | All test items |
| Performance | Timed benchmarks comparing CAPI vs. MAPI baseline | FT-12 |
| Migration | MAPI-to-CAPI upgrade and rollback on live clusters | FT-02, FT-03, FT-15 |

### 6.2 Test Approach

- **Automation-first:** All repeatable scenarios will be automated in CI (OpenShift CI / Prow). Manual testing is reserved for exploratory and UX-complexity validation.
- **Baseline comparison:** Performance and reliability metrics will be captured for MAPI-based operations first, then compared against equivalent CAPI operations.
- **Dev preview gate:** Since SPLAT-2560 targets dev preview, test failures in non-blocking scenarios may be accepted with documented waivers, but blocking scenarios (installation, upgrade, rollback) must pass.
- **Negative testing:** Deliberate fault injection (invalid credentials, unreachable vCenter, interrupted migrations) to validate error handling and recovery.
- **Regression:** Existing MAPI-based e2e test suites will be adapted to run against CAPI-managed clusters to verify functional parity.

### 6.3 Pass/Fail Criteria (Global)

| Criterion | Threshold |
|-----------|-----------|
| All blocking scenarios pass | 100% |
| Non-blocking scenario pass rate | >= 90% (`Proposed`) |
| No P0/P1 defects open at exit | Required |
| Performance within baseline tolerance | <= 110% of MAPI baseline time (`Proposed`) |
| Zero data-loss or corruption incidents | Required |

---

## 7. Requirements Traceability Matrix

Each requirement derived from SPLAT-2560 and SPLAT-2410 is mapped to one or more test scenarios defined in [Section 8](#8-test-scenarios-and-cases).

| Req ID | Requirement | Source | Test Scenario(s) |
|--------|-------------|--------|-------------------|
| REQ-01 | New CAPI-based installation on vSphere/VCF (day-0) | SPLAT-2560 | TC-INST-01, TC-INST-02, TC-INST-03, TC-INST-04 |
| REQ-02 | Upgrade path from existing MAPI-based clusters to CAPI | SPLAT-2560, SPLAT-2410 | TC-MIG-01, TC-MIG-02, TC-MIG-03 |
| REQ-03 | Functional parity with existing MAPI solution | SPLAT-2560 | TC-PAR-01, TC-PAR-02, TC-PAR-03 |
| REQ-04 | Comparable-or-better provision/scale/upgrade performance | SPLAT-2560 | TC-PERF-01, TC-PERF-02, TC-PERF-03 |
| REQ-05 | At-least-current reliability | SPLAT-2560 | TC-REL-01, TC-REL-02 |
| REQ-06 | Removal of Terraform dependency / upstream CAPI alignment | SPLAT-2560 | TC-INST-01, TC-DEP-01 |
| REQ-07 | No unnecessary user complexity | SPLAT-2560 | TC-UX-01, TC-UX-02 |
| REQ-08 | Day-2 lifecycle management | SPLAT-2560 | TC-D2-01, TC-D2-02, TC-D2-03, TC-D2-04 |
| REQ-09 | Control Plane Machine Set support | SPLAT-2560 | TC-PAR-03 |
| REQ-10 | Static IP support | SPLAT-2560 | TC-INST-03 |
| REQ-11 | Multi-NIC support | SPLAT-2560 | TC-INST-04 |
| REQ-12 | Existing MAPI clusters can be upgraded to CAPI | SPLAT-2410 | TC-MIG-01, TC-MIG-02 |
| REQ-13 | Migration process is reliable and documented | SPLAT-2410 | TC-MIG-01, TC-OBS-02 |
| REQ-14 | No cluster downtime during migration | SPLAT-2410 | TC-MIG-03 |
| REQ-15 | Rollback available if migration fails | SPLAT-2410 | TC-RB-01, TC-RB-02 |

---

## 8. Test Scenarios and Cases

### 8.1 New Installation (Day-0)

#### TC-INST-01: Fresh CAPI-based OCP installation on vSphere

| Field | Value |
|-------|-------|
| **ID** | TC-INST-01 |
| **Title** | Fresh CAPI-based OCP installation on vSphere |
| **Priority** | P0 — Blocking |
| **Traces to** | REQ-01, REQ-06 |
| **Prerequisites** | vSphere environment with supported version (TBD); valid OCP pull secret; DNS/DHCP configured; `openshift-install` binary with CAPI support |
| **Test Data** | Standard `install-config.yaml` targeting vSphere with CAPI platform type; cluster name, base domain, vCenter credentials |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Generate install manifests using `openshift-install create manifests` with CAPI-enabled vSphere config | Manifests generated; no Terraform-related files present |
| 2 | Run `openshift-install create cluster` | Installation proceeds using CAPI/CAPV controllers |
| 3 | Monitor bootstrap and control plane provisioning | Bootstrap VM created via CAPV; control plane machines created as CAPI Machine objects |
| 4 | Wait for install completion | Cluster reports `install-complete`; `oc get nodes` shows all expected nodes in `Ready` state |
| 5 | Verify CAPI resources | `oc get clusters,machines,vspherevm -A` returns expected CAPI/CAPV objects |
| 6 | Verify no Terraform artifacts | No Terraform state files, binaries, or provider plugins present in installer output or cluster |
| 7 | Run OpenShift conformance suite | Conformance tests pass at baseline rate |

**Pass/Fail Criteria:** Cluster installs successfully; all nodes `Ready`; CAPI resources healthy; no Terraform dependency; conformance pass rate meets baseline.

---

#### TC-INST-02: Fresh CAPI-based OCP installation on VCF

| Field | Value |
|-------|-------|
| **ID** | TC-INST-02 |
| **Title** | Fresh CAPI-based OCP installation on VMware Cloud Foundation |
| **Priority** | P0 — Blocking |
| **Traces to** | REQ-01 |
| **Prerequisites** | VCF environment with supported version (TBD); valid OCP pull secret; DNS/DHCP configured |
| **Test Data** | `install-config.yaml` targeting VCF with CAPI platform type |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Configure `install-config.yaml` for VCF-specific parameters | Config accepted without errors |
| 2 | Run `openshift-install create cluster` | Installation completes using CAPI/CAPV on VCF infrastructure |
| 3 | Verify node and CAPI resource health | All nodes `Ready`; CAPI Machine objects reconciled |
| 4 | Validate VCF-specific integration | VMs correctly placed in VCF workload domain; storage and networking aligned with VCF policies |

**Pass/Fail Criteria:** Cluster installs successfully on VCF; all nodes and CAPI resources healthy.

---

#### TC-INST-03: Installation with Static IP configuration

| Field | Value |
|-------|-------|
| **ID** | TC-INST-03 |
| **Title** | CAPI-based installation with Static IP addresses |
| **Priority** | P1 |
| **Traces to** | REQ-01, REQ-10 |
| **Prerequisites** | vSphere environment; IP address pool reserved; no DHCP for cluster network segment |
| **Test Data** | `install-config.yaml` with static IP definitions for control plane and worker nodes |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Configure static IP parameters in install config | Config validates successfully |
| 2 | Run `openshift-install create cluster` | Installation completes; all nodes receive configured static IPs |
| 3 | Verify node IP assignments | `oc get nodes -o wide` shows expected static IPs; no DHCP leases consumed |
| 4 | Reboot a node | Node retains its static IP after reboot |

**Pass/Fail Criteria:** All nodes receive and retain configured static IPs; cluster fully operational without DHCP.

---

#### TC-INST-04: Installation with Multi-NIC configuration

| Field | Value |
|-------|-------|
| **ID** | TC-INST-04 |
| **Title** | CAPI-based installation with multiple network interfaces |
| **Priority** | P1 |
| **Traces to** | REQ-01, REQ-11 |
| **Prerequisites** | vSphere environment with multiple port groups/networks configured; additional NICs defined in templates |
| **Test Data** | `install-config.yaml` with multi-NIC configuration; network attachment definitions |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Configure multi-NIC parameters in install config | Config validates successfully |
| 2 | Run `openshift-install create cluster` | Installation completes; VMs have multiple NICs attached |
| 3 | Verify network interfaces on nodes | `ip addr` on each node shows expected additional interfaces |
| 4 | Validate traffic on secondary interfaces | Pods scheduled with secondary network attachment can communicate over additional NICs |

**Pass/Fail Criteria:** All nodes provisioned with correct NIC count; secondary networks functional.

---

### 8.2 MAPI-to-CAPI Migration / Upgrade

#### TC-MIG-01: Successful MAPI-to-CAPI migration

| Field | Value |
|-------|-------|
| **ID** | TC-MIG-01 |
| **Title** | Complete MAPI-to-CAPI migration on a running cluster |
| **Priority** | P0 — Blocking |
| **Traces to** | REQ-02, REQ-12, REQ-13 |
| **Prerequisites** | Running OCP cluster on vSphere provisioned via MAPI; cluster healthy; migration tooling installed |
| **Test Data** | Existing MAPI cluster with 3 control plane + 3 worker nodes |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Record pre-migration state (node count, Machine objects, workload health) | Baseline captured |
| 2 | Initiate MAPI-to-CAPI migration procedure per documentation | Migration process begins; progress observable via CLI/events |
| 3 | Monitor migration progress | MAPI Machine objects gradually replaced by CAPI Machine objects |
| 4 | Wait for migration completion | All machines managed by CAPI; MAPI controllers scaled down or removed |
| 5 | Verify cluster health post-migration | All nodes `Ready`; all ClusterOperators available; workloads running |
| 6 | Verify CAPI resource integrity | `oc get clusters,machines,vspherevm -A` shows correct CAPI objects |
| 7 | Run smoke test suite | Standard e2e smoke tests pass |

**Pass/Fail Criteria:** Migration completes without errors; all nodes remain `Ready`; workloads uninterrupted; CAPI resources reconciled.

---

#### TC-MIG-02: Migration of large cluster

| Field | Value |
|-------|-------|
| **ID** | TC-MIG-02 |
| **Title** | MAPI-to-CAPI migration on a cluster with many nodes |
| **Priority** | P1 |
| **Traces to** | REQ-02, REQ-12 |
| **Prerequisites** | Running OCP cluster on vSphere with 3 control plane + 50+ worker nodes (`Proposed` scale) |
| **Test Data** | Large cluster with diverse MachineSet configurations |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Record pre-migration state and baseline metrics | Baseline captured |
| 2 | Initiate migration | Migration proceeds across all nodes |
| 3 | Monitor node-by-node migration progress | Each node transitions without disruption to others |
| 4 | Validate completion | All 50+ nodes under CAPI management; no data loss |

**Pass/Fail Criteria:** All nodes migrate successfully; migration time scales linearly or better with node count (`Proposed`).

---

#### TC-MIG-03: Zero-downtime validation during migration

| Field | Value |
|-------|-------|
| **ID** | TC-MIG-03 |
| **Title** | Verify zero cluster downtime during MAPI-to-CAPI migration |
| **Priority** | P0 — Blocking |
| **Traces to** | REQ-02, REQ-14 |
| **Prerequisites** | Running OCP cluster on vSphere provisioned via MAPI; synthetic workload deployed (HTTP service + continuous client) |
| **Test Data** | Continuous health-check probe running throughout migration; PodDisruptionBudgets configured |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Deploy synthetic workload with liveness/readiness probes and continuous client | Workload running; client recording request success/failure |
| 2 | Deploy cluster monitoring with API availability probes | Monitoring active |
| 3 | Initiate MAPI-to-CAPI migration | Migration begins |
| 4 | Continuously monitor API server availability | API server accessible throughout migration |
| 5 | Continuously monitor workload availability | Zero failed requests from continuous client |
| 6 | Validate final state after migration | Cluster healthy; zero-downtime confirmed via monitoring data |

**Pass/Fail Criteria:** API server and workload availability remain 100% during migration; no request failures recorded.

---

### 8.3 Rollback

#### TC-RB-01: Rollback after failed migration

| Field | Value |
|-------|-------|
| **ID** | TC-RB-01 |
| **Title** | Rollback MAPI-to-CAPI migration after failure |
| **Priority** | P0 — Blocking |
| **Traces to** | REQ-15 |
| **Prerequisites** | Running OCP cluster on vSphere provisioned via MAPI; migration initiated but not completed |
| **Test Data** | Cluster with partial migration state (some nodes CAPI, some MAPI) |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Initiate MAPI-to-CAPI migration | Migration begins |
| 2 | Simulate migration failure midway (e.g., revoke vCenter permissions during migration) | Migration encounters error and halts |
| 3 | Initiate rollback procedure per documentation | Rollback process begins |
| 4 | Monitor rollback progress | Partially migrated nodes return to MAPI management |
| 5 | Verify cluster health post-rollback | All nodes `Ready` under MAPI; cluster fully operational |
| 6 | Verify no CAPI resource residue | No orphaned CAPI Machine or VSphereMachine objects |

**Pass/Fail Criteria:** Rollback completes successfully; cluster returns to pre-migration MAPI state; no orphaned resources.

---

#### TC-RB-02: Rollback with workload validation

| Field | Value |
|-------|-------|
| **ID** | TC-RB-02 |
| **Title** | Verify workload integrity after migration rollback |
| **Priority** | P1 |
| **Traces to** | REQ-15 |
| **Prerequisites** | Same as TC-RB-01 plus deployed stateful and stateless workloads |
| **Test Data** | Stateful workload (PVC-backed) + stateless workload; continuous health probes |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Deploy workloads and record state (PV data, pod counts) | Baseline captured |
| 2 | Initiate and then roll back migration (as in TC-RB-01) | Rollback completes |
| 3 | Verify stateless workload | Pods running; service endpoints responding |
| 4 | Verify stateful workload | PV data intact; no data corruption |
| 5 | Run e2e smoke tests | All pass |

**Pass/Fail Criteria:** All workloads operational post-rollback; no data loss or corruption.

---

### 8.4 Interruption and Recovery

#### TC-IR-01: Installation interrupted by vCenter disconnect

| Field | Value |
|-------|-------|
| **ID** | TC-IR-01 |
| **Title** | CAPI installation recovery after temporary vCenter unavailability |
| **Priority** | P1 |
| **Traces to** | REQ-01, REQ-05 |
| **Prerequisites** | vSphere environment; CAPI installation in progress |
| **Test Data** | Standard install config |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Begin `openshift-install create cluster` | Installation starts |
| 2 | During bootstrap phase, block network access to vCenter for 5 minutes | CAPV controller logs connection errors; retries |
| 3 | Restore network access to vCenter | CAPV controller reconnects |
| 4 | Wait for installation to complete | Installation recovers and completes successfully |

**Pass/Fail Criteria:** Installation recovers after transient vCenter outage; no manual intervention required.

---

#### TC-IR-02: Node failure during day-2 operations

| Field | Value |
|-------|-------|
| **ID** | TC-IR-02 |
| **Title** | CAPI-managed node recovery after unexpected node failure |
| **Priority** | P1 |
| **Traces to** | REQ-08, REQ-05 |
| **Prerequisites** | Running CAPI-managed OCP cluster on vSphere; MachineHealthCheck configured |
| **Test Data** | Healthy cluster with 3 workers |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Identify a worker node and record its CAPI Machine object | Baseline captured |
| 2 | Force-delete the worker VM in vCenter (simulating hardware failure) | Node becomes `NotReady` |
| 3 | Wait for MachineHealthCheck to detect failure | MHC marks Machine as unhealthy |
| 4 | Wait for CAPI to remediate | New VM provisioned; node joins cluster as `Ready` |
| 5 | Verify cluster returns to desired node count | `oc get nodes` shows expected count; all `Ready` |

**Pass/Fail Criteria:** CAPI auto-remediates failed node; cluster returns to desired state without manual intervention.

---

#### TC-IR-03: Migration interrupted by operator restart

| Field | Value |
|-------|-------|
| **ID** | TC-IR-03 |
| **Title** | MAPI-to-CAPI migration recovery after operator pod restart |
| **Priority** | P1 |
| **Traces to** | REQ-02, REQ-05 |
| **Prerequisites** | Running MAPI cluster; migration in progress |
| **Test Data** | Cluster mid-migration |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Begin MAPI-to-CAPI migration | Migration in progress |
| 2 | Delete the cluster-capi-operator pod during migration | Pod restarts via Deployment |
| 3 | Monitor migration state | Operator resumes migration from last checkpoint |
| 4 | Wait for migration completion | Migration completes successfully |

**Pass/Fail Criteria:** Migration resumes and completes after operator restart; no data loss or inconsistency.

---

### 8.5 Day-2 Lifecycle

#### TC-D2-01: Scale worker nodes up

| Field | Value |
|-------|-------|
| **ID** | TC-D2-01 |
| **Title** | Scale up worker MachineSet on CAPI-managed cluster |
| **Priority** | P0 — Blocking |
| **Traces to** | REQ-08, REQ-03 |
| **Prerequisites** | Running CAPI-managed OCP cluster on vSphere |
| **Test Data** | Initial 3-worker cluster |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Record current worker count and MachineSet replicas | Baseline: 3 workers |
| 2 | Scale MachineSet replicas from 3 to 5 | CAPI creates 2 new Machine objects |
| 3 | Wait for new VMs and nodes | 2 new VMs created in vSphere; 2 new nodes join cluster |
| 4 | Verify all 5 workers are `Ready` | `oc get nodes` shows 5 Ready workers |
| 5 | Verify workload scheduling on new nodes | Pods can be scheduled on new nodes |

**Pass/Fail Criteria:** Scale-up completes; new nodes Ready and schedulable.

---

#### TC-D2-02: Scale worker nodes down

| Field | Value |
|-------|-------|
| **ID** | TC-D2-02 |
| **Title** | Scale down worker MachineSet on CAPI-managed cluster |
| **Priority** | P1 |
| **Traces to** | REQ-08, REQ-03 |
| **Prerequisites** | Running CAPI-managed OCP cluster with 5 workers (post TC-D2-01) |
| **Test Data** | 5-worker cluster |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Scale MachineSet replicas from 5 to 3 | CAPI marks 2 Machines for deletion |
| 2 | Wait for node drain and VM deletion | Pods drained; VMs deleted from vSphere |
| 3 | Verify 3 workers remain `Ready` | `oc get nodes` shows 3 Ready workers |
| 4 | Verify no orphaned VMs in vSphere | vCenter shows only 3 worker VMs |

**Pass/Fail Criteria:** Scale-down completes gracefully; workloads rescheduled; no orphaned resources.

---

#### TC-D2-03: OCP cluster upgrade on CAPI-managed cluster

| Field | Value |
|-------|-------|
| **ID** | TC-D2-03 |
| **Title** | OCP version upgrade on CAPI-managed vSphere cluster |
| **Priority** | P0 — Blocking |
| **Traces to** | REQ-08, REQ-03, REQ-04 |
| **Prerequisites** | Running CAPI-managed OCP cluster at version N; version N+1 available |
| **Test Data** | OCP versions N and N+1 (specific versions TBD) |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Record pre-upgrade state (version, node status, operator status) | Baseline captured |
| 2 | Initiate cluster upgrade to version N+1 | CVO begins upgrade |
| 3 | Monitor upgrade progress | Control plane nodes upgraded; worker nodes upgraded |
| 4 | Verify upgrade completion | `oc get clusterversion` shows N+1; all nodes and operators healthy |
| 5 | Measure upgrade duration | Record and compare against MAPI baseline |

**Pass/Fail Criteria:** Upgrade completes successfully; all nodes and operators healthy at new version.

---

#### TC-D2-04: Node remediation via MachineHealthCheck

| Field | Value |
|-------|-------|
| **ID** | TC-D2-04 |
| **Title** | Automated node remediation via MachineHealthCheck on CAPI cluster |
| **Priority** | P1 |
| **Traces to** | REQ-08, REQ-03 |
| **Prerequisites** | Running CAPI-managed OCP cluster; MachineHealthCheck configured |
| **Test Data** | Healthy 3-worker cluster |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Cordon and taint a worker node to simulate failure condition | Node marked unschedulable |
| 2 | Stop kubelet on the worker node | Node transitions to `NotReady` |
| 3 | Wait for MachineHealthCheck timeout | MHC detects unhealthy node; annotates Machine for remediation |
| 4 | Observe CAPI remediation | Old Machine deleted; new Machine provisioned; new node joins |
| 5 | Verify cluster health | All nodes `Ready`; desired count restored |

**Pass/Fail Criteria:** MachineHealthCheck triggers remediation; new node replaces failed node automatically.

---

### 8.6 Feature Parity with MAPI

#### TC-PAR-01: MachineSets with various configurations

| Field | Value |
|-------|-------|
| **ID** | TC-PAR-01 |
| **Title** | CAPI MachineSet feature parity with MAPI MachineSet |
| **Priority** | P1 |
| **Traces to** | REQ-03 |
| **Prerequisites** | Running CAPI-managed OCP cluster on vSphere |
| **Test Data** | MachineSets with varied CPU, memory, disk, and storage policy configurations |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Create MachineSet with custom CPU/memory | Machine created with correct resource allocation in vSphere |
| 2 | Create MachineSet with custom disk size | VM disk provisioned at specified size |
| 3 | Create MachineSet with vSphere storage policy | VM uses correct storage policy |
| 4 | Create MachineSet in specific resource pool/folder | VM placed in correct vSphere resource pool and folder |
| 5 | Delete a Machine manually | Replacement Machine created by MachineSet controller |

**Pass/Fail Criteria:** All MAPI-supported MachineSet configuration options work equivalently under CAPI.

---

#### TC-PAR-02: Machine annotations and labels

| Field | Value |
|-------|-------|
| **ID** | TC-PAR-02 |
| **Title** | Verify CAPI Machine metadata propagation |
| **Priority** | P2 |
| **Traces to** | REQ-03 |
| **Prerequisites** | Running CAPI-managed OCP cluster |
| **Test Data** | MachineSet with labels and annotations |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Create MachineSet with node labels and annotations | Machines created with specified metadata |
| 2 | Verify labels propagate to Nodes | `oc get node -L <label>` shows expected values |
| 3 | Update MachineSet labels | Existing Machines updated (or new Machines reflect changes, per CAPI semantics) |

**Pass/Fail Criteria:** Labels and annotations propagate correctly from MachineSet to Machine to Node.

---

#### TC-PAR-03: Control Plane Machine Set (CPMS) operations

| Field | Value |
|-------|-------|
| **ID** | TC-PAR-03 |
| **Title** | CPMS functionality on CAPI-managed cluster |
| **Priority** | P1 |
| **Traces to** | REQ-03, REQ-09 |
| **Prerequisites** | Running CAPI-managed OCP cluster with CPMS enabled |
| **Test Data** | 3-node control plane |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Verify CPMS object exists and is managing control plane Machines | CPMS reports 3 replicas, all updated |
| 2 | Update CPMS spec (e.g., increase control plane VM memory) | CPMS initiates rolling replacement of control plane machines |
| 3 | Monitor rolling update | One control plane machine replaced at a time; cluster remains available |
| 4 | Verify completion | All 3 control plane machines updated; cluster healthy |

**Pass/Fail Criteria:** CPMS manages control plane under CAPI; rolling updates work without cluster disruption.

---

### 8.7 Performance

#### TC-PERF-01: Cluster provisioning time — CAPI vs. MAPI

| Field | Value |
|-------|-------|
| **ID** | TC-PERF-01 |
| **Title** | Compare CAPI cluster provisioning time against MAPI baseline |
| **Priority** | P1 |
| **Traces to** | REQ-04 |
| **Prerequisites** | Identical vSphere environment for both CAPI and MAPI installations |
| **Test Data** | Standard 3 control-plane + 3 worker installation; same hardware specs |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Record MAPI baseline: install cluster end-to-end, capture wall-clock time | Baseline time recorded |
| 2 | Install identical cluster via CAPI, capture wall-clock time | CAPI time recorded |
| 3 | Compare times | CAPI time <= 110% of MAPI baseline (`Proposed` threshold) |
| 4 | Repeat 3 times for statistical significance | Standard deviation within acceptable range |

**Pass/Fail Criteria:** CAPI provisioning time comparable to or better than MAPI (within `Proposed` 110% tolerance).

---

#### TC-PERF-02: Node scale-out time — CAPI vs. MAPI

| Field | Value |
|-------|-------|
| **ID** | TC-PERF-02 |
| **Title** | Compare CAPI node scale-out time against MAPI baseline |
| **Priority** | P1 |
| **Traces to** | REQ-04 |
| **Prerequisites** | Running clusters — one MAPI, one CAPI — on identical vSphere infrastructure |
| **Test Data** | Scale from 3 to 10 workers |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | On MAPI cluster: scale MachineSet to 10, record time to all nodes Ready | MAPI baseline captured |
| 2 | On CAPI cluster: scale MachineSet to 10, record time to all nodes Ready | CAPI time captured |
| 3 | Compare times | CAPI time <= 110% of MAPI baseline (`Proposed` threshold) |

**Pass/Fail Criteria:** CAPI scale-out time comparable to or better than MAPI.

---

#### TC-PERF-03: Cluster upgrade time — CAPI vs. MAPI

| Field | Value |
|-------|-------|
| **ID** | TC-PERF-03 |
| **Title** | Compare OCP upgrade time on CAPI-managed vs. MAPI-managed cluster |
| **Priority** | P2 |
| **Traces to** | REQ-04 |
| **Prerequisites** | Running clusters at version N — one MAPI, one CAPI |
| **Test Data** | Upgrade from version N to N+1 |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Upgrade MAPI cluster, record wall-clock time | MAPI baseline captured |
| 2 | Upgrade CAPI cluster, record wall-clock time | CAPI time captured |
| 3 | Compare times | CAPI upgrade time <= 110% of MAPI baseline (`Proposed` threshold) |

**Pass/Fail Criteria:** CAPI upgrade time comparable to or better than MAPI.

---

### 8.8 Reliability

#### TC-REL-01: Repeated installation reliability

| Field | Value |
|-------|-------|
| **ID** | TC-REL-01 |
| **Title** | CAPI installation reliability over repeated runs |
| **Priority** | P1 |
| **Traces to** | REQ-05 |
| **Prerequisites** | vSphere environment |
| **Test Data** | Standard install config |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Run CAPI-based installation 10 times (`Proposed` count) on same infrastructure | Record pass/fail per run |
| 2 | Calculate success rate | Success rate >= MAPI baseline success rate |
| 3 | For any failures, capture logs and categorize root cause | Failures categorized (infra vs. product bug) |

**Pass/Fail Criteria:** CAPI installation success rate >= MAPI baseline; no new product-bug failure modes.

---

#### TC-REL-02: Long-running cluster stability

| Field | Value |
|-------|-------|
| **ID** | TC-REL-02 |
| **Title** | CAPI-managed cluster stability under sustained operation |
| **Priority** | P2 |
| **Traces to** | REQ-05 |
| **Prerequisites** | Running CAPI-managed OCP cluster with representative workloads |
| **Test Data** | 72-hour soak test (`Proposed` duration) |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Deploy mixed workloads (stateful, stateless, batch) | Workloads running |
| 2 | Run cluster for 72 hours with periodic health checks | No unexpected node failures, operator degradation, or CAPI controller crashes |
| 3 | Perform periodic scale operations during soak | Scale operations complete successfully |
| 4 | Collect resource utilization of CAPI controllers | CPU/memory within reasonable bounds; no memory leaks |

**Pass/Fail Criteria:** No unexpected failures during soak period; CAPI controller resource usage stable.

---

### 8.9 Negative / Error-Handling Scenarios

#### TC-NEG-01: Installation with invalid vCenter credentials

| Field | Value |
|-------|-------|
| **ID** | TC-NEG-01 |
| **Title** | CAPI installation fails gracefully with invalid vCenter credentials |
| **Priority** | P2 |
| **Traces to** | REQ-07 (user complexity — clear errors) |
| **Prerequisites** | vSphere environment |
| **Test Data** | `install-config.yaml` with incorrect vCenter username/password |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Run `openshift-install create cluster` with invalid credentials | Installation fails |
| 2 | Check error output | Clear, actionable error message indicating authentication failure |
| 3 | Verify no partial resources left behind | No orphaned VMs or CAPI objects in vSphere |

**Pass/Fail Criteria:** Clear error message; no orphaned resources; user can fix and retry.

---

#### TC-NEG-02: Installation with insufficient vSphere permissions

| Field | Value |
|-------|-------|
| **ID** | TC-NEG-02 |
| **Title** | CAPI installation fails gracefully with insufficient vSphere permissions |
| **Priority** | P2 |
| **Traces to** | REQ-07 |
| **Prerequisites** | vSphere environment with a restricted-privilege vCenter account |
| **Test Data** | `install-config.yaml` with limited-privilege credentials |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Run `openshift-install create cluster` with restricted account | Installation fails at VM creation |
| 2 | Check error output | Error identifies missing permissions |
| 3 | Verify cleanup | No orphaned resources |

**Pass/Fail Criteria:** Actionable error message; graceful cleanup.

---

#### TC-NEG-03: Scale operation with resource exhaustion

| Field | Value |
|-------|-------|
| **ID** | TC-NEG-03 |
| **Title** | CAPI scale-up behavior when vSphere resources are exhausted |
| **Priority** | P2 |
| **Traces to** | REQ-08, REQ-07 |
| **Prerequisites** | Running CAPI-managed cluster; vSphere resource pool with tight CPU/memory limits |
| **Test Data** | MachineSet scale request exceeding available resources |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Scale MachineSet to request more VMs than resources allow | CAPV attempts provisioning |
| 2 | Observe Machine status | Machines show clear error condition (e.g., `ProvisioningFailed`) |
| 3 | Verify existing nodes are unaffected | Running nodes remain `Ready` |
| 4 | Free resources and verify recovery | Pending Machines eventually provisioned |

**Pass/Fail Criteria:** Clear error reporting; existing cluster unaffected; self-healing when resources become available.

---

### 8.10 Terraform Dependency Removal

#### TC-DEP-01: Verify no Terraform dependency in install workflow

| Field | Value |
|-------|-------|
| **ID** | TC-DEP-01 |
| **Title** | Confirm Terraform is not required or used during CAPI installation |
| **Priority** | P1 |
| **Traces to** | REQ-06 |
| **Prerequisites** | Environment with no Terraform binary installed |
| **Test Data** | Standard CAPI install config |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Remove/confirm absence of Terraform binary from install host | `which terraform` returns not found |
| 2 | Run `openshift-install create cluster` | Installation completes successfully |
| 3 | Inspect installer output directory | No `.terraform`, `terraform.tfstate`, or `.tf` files |
| 4 | Inspect cluster resources | No Terraform-related ConfigMaps or Secrets |

**Pass/Fail Criteria:** Installation succeeds without Terraform; no Terraform artifacts anywhere in the workflow.

---

### 8.11 Observability and Documentation

#### TC-OBS-01: CAPI resource observability

| Field | Value |
|-------|-------|
| **ID** | TC-OBS-01 |
| **Title** | Verify CAPI resources are observable via standard OCP tooling |
| **Priority** | P2 |
| **Traces to** | REQ-07 |
| **Prerequisites** | Running CAPI-managed OCP cluster |
| **Test Data** | N/A |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | List CAPI resources via `oc get` | Cluster, Machine, MachineSet, VSphereMachine resources visible |
| 2 | Describe a Machine object | Status conditions, provider ID, and phase clearly shown |
| 3 | Check events | CAPI lifecycle events (create, update, delete) recorded as Kubernetes events |
| 4 | Verify metrics endpoint | CAPI controller exposes Prometheus metrics; scraped by cluster monitoring |
| 5 | Verify alerting | Alerts fire for CAPI controller failures or degraded Machines (`Proposed`) |

**Pass/Fail Criteria:** CAPI resources observable via CLI, events, metrics, and alerts.

---

#### TC-OBS-02: Migration documentation accuracy

| Field | Value |
|-------|-------|
| **ID** | TC-OBS-02 |
| **Title** | Verify MAPI-to-CAPI migration documentation is accurate and complete |
| **Priority** | P1 |
| **Traces to** | REQ-13 |
| **Prerequisites** | Published migration documentation; vSphere test environment |
| **Test Data** | Running MAPI cluster |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Follow migration documentation step-by-step on a test cluster | Every documented step executable as written |
| 2 | Verify prerequisites section is complete | No undocumented prerequisites encountered |
| 3 | Verify troubleshooting section | Common failure modes and resolutions documented |
| 4 | Verify rollback documentation | Rollback steps are documented and accurate |

**Pass/Fail Criteria:** Documentation is accurate, complete, and sufficient for a qualified administrator to perform migration without external help.

---

### 8.12 User Complexity

#### TC-UX-01: Install config complexity comparison

| Field | Value |
|-------|-------|
| **ID** | TC-UX-01 |
| **Title** | CAPI install config is not more complex than MAPI install config |
| **Priority** | P2 |
| **Traces to** | REQ-07 |
| **Prerequisites** | CAPI and MAPI install-config templates |
| **Test Data** | Side-by-side comparison of install configs |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Compare CAPI `install-config.yaml` fields against MAPI equivalent | No additional required fields introduced without justification |
| 2 | Verify default values cover common cases | Standard vSphere install works with minimal config |
| 3 | Review error messages for user-friendliness | Errors reference config fields and suggest fixes |

**Pass/Fail Criteria:** CAPI install config not more complex than MAPI; sensible defaults; clear error messages.

---

#### TC-UX-02: Day-2 operations UX comparison

| Field | Value |
|-------|-------|
| **ID** | TC-UX-02 |
| **Title** | Day-2 CAPI operations not more complex than MAPI equivalents |
| **Priority** | P2 |
| **Traces to** | REQ-07 |
| **Prerequisites** | Running CAPI and MAPI clusters |
| **Test Data** | Common day-2 operation playbooks |

**Steps:**

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Compare `oc` commands for scaling (CAPI vs. MAPI) | Same or simpler command interface |
| 2 | Compare `oc` commands for machine inspection | Same or equivalent output |
| 3 | Verify OCP console integration | CAPI-managed machines visible in web console with equivalent detail |

**Pass/Fail Criteria:** CAPI day-2 operations no more complex than MAPI equivalents.

---

## 9. Suspension and Resumption Criteria

### 9.1 Suspension Criteria

Testing shall be suspended if any of the following occur:

| # | Condition |
|---|-----------|
| 1 | vSphere/VCF test infrastructure becomes unavailable for > 4 hours |
| 2 | A P0 blocking defect is discovered that prevents further test execution |
| 3 | CAPI operator or CAPV controller fails to deploy on the target OCP version |
| 4 | SPLAT-2560 scope changes significantly (requires test plan revision) |
| 5 | Work on SPLAT-2560 remains on hold (current status) |

### 9.2 Resumption Criteria

Testing shall resume when:

| # | Condition |
|---|-----------|
| 1 | Infrastructure is restored and verified operational |
| 2 | P0 blocking defect is fixed and verified |
| 3 | Deployment issue is resolved; operator/controller deploy successfully |
| 4 | Updated test plan is reviewed and approved for revised scope |
| 5 | SPLAT-2560 work is reactivated and development progresses to testable state |

---

## 10. Test Deliverables

| Deliverable | Format | Owner |
|-------------|--------|-------|
| This test plan document | Markdown | N/A |
| Test case automation (Prow/CI jobs) | Go / shell scripts in openshift/release | N/A |
| MAPI baseline performance report | Markdown / JSON | N/A |
| CAPI performance report | Markdown / JSON | N/A |
| Test execution summary report | Markdown | N/A |
| Defect reports | Jira issues under SPLAT project | N/A |
| Migration documentation review findings | Markdown | N/A |

---

## 11. Test Tasks

| Task ID | Task | Dependency | Status |
|---------|------|------------|--------|
| TT-01 | Establish MAPI baseline metrics (install time, scale time, upgrade time, success rate) | vSphere test environment available | TBD |
| TT-02 | Configure vSphere test environment for CAPI testing | TT-01 | TBD |
| TT-03 | Configure VCF test environment for CAPI testing | TT-01 | TBD |
| TT-04 | Automate TC-INST-* scenarios in Prow CI | TT-02, CAPI installer builds available | TBD |
| TT-05 | Automate TC-MIG-* and TC-RB-* scenarios | TT-02, migration tooling available | TBD |
| TT-06 | Automate TC-D2-* scenarios | TT-04 | TBD |
| TT-07 | Automate TC-PERF-* scenarios | TT-01, TT-04 | TBD |
| TT-08 | Execute manual TC-UX-* and TC-OBS-02 reviews | TT-04, documentation published | TBD |
| TT-09 | Execute TC-REL-02 soak test | TT-04 | TBD |
| TT-10 | Compile test execution summary report | TT-04 through TT-09 | TBD |

---

## 12. Environment Needs

### 12.1 Infrastructure

| Component | Specification |
|-----------|---------------|
| VMware vSphere | Supported version (TBD per SPLAT-2560 scope finalization) |
| VMware Cloud Foundation | Supported version (TBD) |
| vCenter Server | Accessible from test network; service account with full VM lifecycle permissions |
| ESXi hosts | Minimum 3 hosts for HA testing; sufficient CPU/memory for 50+ node scale tests |
| Storage | vSAN or NFS datastore accessible from all hosts; support for storage policies |
| Networking | At least 2 port groups for Multi-NIC testing; DHCP and non-DHCP segments for Static IP testing |
| DNS | Wildcard DNS for cluster ingress; API and API-int DNS records |

### 12.2 Software

| Component | Version |
|-----------|---------|
| OCP | Target version TBD (version supporting CAPI dev preview) |
| CAPV | Target version TBD (per SPLAT-2560) |
| `openshift-install` | Build with CAPI vSphere support |
| `oc` CLI | Matching OCP version |
| Prow / CI-Operator | Current production version |

### 12.3 Accounts and Access

| Resource | Access Needed |
|----------|---------------|
| vCenter service account | Full VM lifecycle, resource pool, folder, storage policy, network permissions |
| vCenter restricted account | Limited permissions for TC-NEG-02 |
| OCP pull secret | Valid Red Hat pull secret |
| CI system access | Ability to create/modify Prow jobs in openshift/release |

---

## 13. Responsibilities

| Role | Responsibility |
|------|---------------|
| Test plan author | Maintain this document; update as scope evolves |
| Feature development team | Deliver testable builds; fix defects; provide migration documentation |
| QE team | Execute test cases; automate CI jobs; report defects |
| Infrastructure team | Provision and maintain vSphere/VCF test environments |
| Release team | Gate release on test results; approve waivers for non-blocking failures |

> **Note:** Specific personnel assignments are N/A at this time.

---

## 14. Staffing and Training

### 14.1 Staffing

| Role | Count | Status |
|------|-------|--------|
| QE engineers (vSphere + CAPI expertise) | TBD | N/A |
| Infrastructure engineer | TBD | N/A |
| Performance test engineer | TBD | N/A |

### 14.2 Training Needs

| Topic | Audience | Resource |
|-------|----------|----------|
| Cluster API architecture and concepts | QE team | [CAPI Quick Start](https://cluster-api.sigs.k8s.io/user/quick-start.html) |
| CAPV provider specifics | QE team | CAPV upstream documentation |
| MAPI-to-CAPI migration procedures | QE team | Internal migration documentation (per REQ-13) |
| OpenShift CAPI operator internals | QE team | [cluster-capi-operator](https://github.com/openshift/cluster-capi-operator) source and design docs |

---

## 15. Schedule

> **Note:** SPLAT-2560 work is currently on hold / dropped in priority. Schedule milestones below are placeholders pending reactivation.

| Milestone | Target Date | Status |
|-----------|-------------|--------|
| Test plan draft complete | 2026-09-02 | Draft |
| Test plan reviewed and approved | TBD | Pending |
| MAPI baseline metrics captured | TBD | Not started |
| Test environment provisioned | TBD | Not started |
| CI automation for P0 scenarios | TBD | Not started |
| P0 blocking scenario execution | TBD | Not started |
| Full test cycle (P0 + P1 + P2) | TBD | Not started |
| Test summary report published | TBD | Not started |

---

## 16. Risks and Contingencies

| Risk ID | Risk | Likelihood | Impact | Mitigation / Contingency |
|---------|------|------------|--------|--------------------------|
| RSK-01 | CAPV target version not finalized; tests may target wrong API surface | High (TBD in epic) | High | Defer CAPV-specific test automation until version locked; test against upstream main in interim |
| RSK-02 | Work on hold — feature may not reach dev preview | High (current status) | High | Maintain test plan as living document; resume when work reactivates |
| RSK-03 | vSphere/VCF test infrastructure availability | Medium | High | Establish dedicated test environment reservation; document fallback environments |
| RSK-04 | MAPI-to-CAPI migration may not support all cluster topologies | Medium | Medium | Enumerate supported topologies early; document unsupported configurations in test plan |
| RSK-05 | Performance regression in CAPI vs. MAPI | Medium | High | Establish baseline early (TT-01); continuous benchmarking in CI |
| RSK-06 | Rollback mechanism incomplete or untested | Medium | Critical | Prioritize TC-RB-01 as P0 blocking; validate before migration goes GA |
| RSK-07 | Scope of "out-of-scope" items (TBD) may expand testing needs | Medium | Medium | Review and revise test plan when TBD items are defined |
| RSK-08 | Upstream CAPI/CAPV breaking changes | Low | High | Pin to specific upstream versions; monitor upstream release notes |

---

## 17. Assumptions and Open Questions

### 17.1 Assumptions

| # | Assumption |
|---|------------|
| A-01 | CAPI-based installation will use the same `openshift-install` entry point as MAPI, with platform-type or feature-gate differentiation |
| A-02 | MAPI-to-CAPI migration will be a documented, supported procedure (not a manual hack) |
| A-03 | Migration rollback will restore the cluster to a fully functional MAPI-managed state |
| A-04 | CAPI and MAPI will not run concurrently on the same cluster in steady state (migration is a transition) |
| A-05 | Dev preview implies reduced support commitments; some P2 scenarios may be deferred |
| A-06 | Performance thresholds (110% of MAPI baseline) are proposed and subject to team agreement |

### 17.2 Open Questions

| # | Question | Impact |
|---|----------|--------|
| OQ-01 | What specific OCP versions will support CAPI on vSphere in dev preview? | Determines test matrix |
| OQ-02 | What specific vSphere and VCF versions are supported? | Determines environment provisioning |
| OQ-03 | What is the target CAPV version? | Determines API surface for test cases |
| OQ-04 | What are the explicit out-of-scope items (TBD in SPLAT-2560)? | May remove/add test scenarios |
| OQ-05 | What is the migration mechanism (operator-driven, CLI tool, manual procedure)? | Impacts TC-MIG-* test steps |
| OQ-06 | Are there cluster topology restrictions for migration (e.g., HA-only, no SNO)? | May add/remove scenarios |
| OQ-07 | What metrics define "at-least-current reliability"? | Determines TC-REL-* pass criteria |
| OQ-08 | When will work on SPLAT-2560 resume? | Determines schedule and resource planning |

---

## 18. Approvals

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Test Plan Author | N/A | N/A | N/A |
| QE Lead | N/A | N/A | N/A |
| Dev Lead | N/A | N/A | N/A |
| Product Owner | N/A | N/A | N/A |
| Program Manager | N/A | N/A | N/A |

---

## Appendix A: Scenario-to-Requirement Reverse Map

Every test scenario traces back to at least one requirement or risk:

| Scenario | Requirement(s) / Risk(s) |
|----------|--------------------------|
| TC-INST-01 | REQ-01, REQ-06 |
| TC-INST-02 | REQ-01 |
| TC-INST-03 | REQ-01, REQ-10 |
| TC-INST-04 | REQ-01, REQ-11 |
| TC-MIG-01 | REQ-02, REQ-12, REQ-13 |
| TC-MIG-02 | REQ-02, REQ-12 |
| TC-MIG-03 | REQ-02, REQ-14 |
| TC-RB-01 | REQ-15 |
| TC-RB-02 | REQ-15 |
| TC-IR-01 | REQ-01, REQ-05 |
| TC-IR-02 | REQ-08, REQ-05 |
| TC-IR-03 | REQ-02, REQ-05 |
| TC-D2-01 | REQ-08, REQ-03 |
| TC-D2-02 | REQ-08, REQ-03 |
| TC-D2-03 | REQ-08, REQ-03, REQ-04 |
| TC-D2-04 | REQ-08, REQ-03 |
| TC-PAR-01 | REQ-03 |
| TC-PAR-02 | REQ-03 |
| TC-PAR-03 | REQ-03, REQ-09 |
| TC-PERF-01 | REQ-04 |
| TC-PERF-02 | REQ-04 |
| TC-PERF-03 | REQ-04 |
| TC-REL-01 | REQ-05 |
| TC-REL-02 | REQ-05 |
| TC-NEG-01 | REQ-07 |
| TC-NEG-02 | REQ-07 |
| TC-NEG-03 | REQ-08, REQ-07 |
| TC-DEP-01 | REQ-06 |
| TC-OBS-01 | REQ-07 |
| TC-OBS-02 | REQ-13 |
| TC-UX-01 | REQ-07 |
| TC-UX-02 | REQ-07 |

---

*End of Test Plan TP-SPLAT-2560-001 v0.1*
