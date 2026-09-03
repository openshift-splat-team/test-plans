# Test Plan: Support for vCenter In-Place Upgrade on OpenShift vSphere Clusters

## 1. Test Plan Identifier

**ID:** SPLAT-2926-TP-001  
**Version:** 0.1 (Draft — needs-refinement)  
**Date:** 2026-09-03  
**Status:** New / Needs Refinement  

## 2. References

| Ref | Link / ID | Description |
|-----|-----------|-------------|
| R1 | [SPLAT-2926](https://redhat.atlassian.net/browse/SPLAT-2926) | Epic: Support for vCenter in-place upgrade on OpenShift vSphere clusters |
| R2 | [OCPSTRAT-3479](https://redhat.atlassian.net/browse/OCPSTRAT-3479) | Parent feature: vSphere cluster survives vCenter upgrade/replacement |
| R3 | [OCPSTRAT-3695](https://redhat.atlassian.net/browse/OCPSTRAT-3695) | Refinement: in-place upgrade preserving network identity, MAC addresses, VM UUIDs |
| R4 | [SPLAT-2499](https://redhat.atlassian.net/browse/SPLAT-2499) | Investigation: OCP on nested vSphere — install, PV, machineset, VCSA upgrade |
| R5 | [OSDOCS-20751](https://redhat.atlassian.net/browse/OSDOCS-20751) | Documentation tracking for supported procedure |
| R6 | [openshift/cloud-provider-vsphere](https://github.com/openshift/cloud-provider-vsphere) | CPI e2e tests (test/e2e/) |
| R7 | [openshift/vmware-vsphere-csi-driver](https://github.com/openshift/vmware-vsphere-csi-driver) | CSI e2e tests (tests/e2e/) |
| R8 | [openshift/machine-config-operator](https://github.com/openshift/machine-config-operator) | MCO e2e tests (test/e2e-*/) |

## 3. Introduction

This test plan covers verification that an OpenShift cluster running on VMware vSphere continues to operate correctly after an **in-place vCenter Server Appliance (VCSA) upgrade**. The upgrade preserves the vCenter network identity (hostname/IP), MAC addresses, and VM UUIDs; ESXi hosts, VMs, nodes, disks, and datastores remain unchanged.

Because SPLAT-2926 is status **New** with label **needs-refinement**, this plan distinguishes confirmed scope from items marked **TBD**. Sections marked TBD are refinement blockers.

## 4. Test Items

| Item | Component | Source |
|------|-----------|--------|
| TI-1 | vSphere Cloud Provider / cloud-controller-manager | openshift/cloud-provider-vsphere |
| TI-2 | vSphere CSI Storage Driver | openshift/vmware-vsphere-csi-driver |
| TI-3 | Machine Config Operator (MCO) | openshift/machine-config-operator |
| TI-4 | Machine API (MAPI / CAPI on vSphere) | Cluster Infrastructure |
| TI-5 | Documented upgrade procedure | OSDOCS-20751 (TBD — not yet complete) |

**OCP version(s):** TBD  
**vCenter source → target version(s):** TBD  
**ESXi version(s):** TBD  

## 5. Features to Be Tested

| ID | Feature | Acceptance Criterion |
|----|---------|----------------------|
| F-1 | Cluster health baseline before vCenter upgrade | Cluster passes baseline health checks |
| F-2 | In-place VCSA upgrade completes without cluster redeployment | Upgrade finishes; cluster is not reinstalled |
| F-3 | Machine/MachineSet health and providerID mapping | All Machines report Running; providerIDs resolve correctly |
| F-4 | cloud-controller-manager reconnection | CCM reconnects; nodes show correct addresses and topology |
| F-5 | MCO reconciliation after vCenter identity change | MCO reports Available; MachineConfigPools finish reconciliation |
| F-6 | PV/PVC accessibility and data continuity | Existing volumes remain accessible; reads/writes succeed |
| F-7 | Workload continuity during maintenance window | Pods remain running or recover within tolerance (TBD) |
| F-8 | Delayed vCenter availability / retry | Components recover after transient vCenter outage during upgrade |
| F-9 | Documentation / procedure validation | Published procedure can be followed end-to-end |

## 6. Features Not to Be Tested

| ID | Exclusion | Rationale |
|----|-----------|-----------|
| NF-1 | vCenter migration to different hostname/IP/network identity | Out of scope per OCPSTRAT-3695; deferred |
| NF-2 | Cross-vCenter workload migration | Out of scope per OCPSTRAT-3695; deferred |
| NF-3 | ESXi host, VM, disk, or datastore changes during upgrade | Assumed unchanged per OCPSTRAT-3479 |
| NF-4 | Post-upgrade rollback/recovery procedure | TBD — only testable if product design defines a rollback path |
| NF-5 | Non-vSphere platform behavior | Not applicable |

## 7. Test Approach

### 7.1 Strategy

1. **Pre-upgrade baseline** — run cluster health, storage, and machine-state checks.
2. **Execute vCenter in-place upgrade** — perform VCSA upgrade preserving identity.
3. **Post-upgrade verification** — re-run baseline checks; confirm reconciliation.
4. **Negative / boundary validation** — confirm out-of-scope scenarios are rejected or documented as unsupported.

### 7.2 Existing Test Assets

- **CPI e2e** (R6): `test/e2e/cpi_vm_test.go` validates node IP, providerID, VM restart, and readiness via govmomi.
- **CSI e2e** (R7): `tests/e2e/` validates PVC provisioning, volume attachment/mount, CNS state, StatefulSet volumes, and disruptive stop/restart of vCenter services.
- **MCO e2e** (R8): `test/e2e-2of2/mco_test.go`, `mcn_test.go` inspect ClusterOperator machine-config, MachineConfigPools, node state, and metrics. `pkg/controller/template/test_data/controller_config_vsphere.yaml` provides vSphere-specific template data.

### 7.3 Automation Status

Automation status is **TBD**. Existing e2e suites listed above can be reused as post-upgrade validation but do not currently orchestrate a vCenter upgrade step. CI job names and pipeline integration are TBD refinement items.

## 8. Test Cases

### TC-01: Baseline Cluster Health

| Field | Detail |
|-------|--------|
| **ID** | TC-01 |
| **Objective** | Confirm cluster is healthy before vCenter upgrade |
| **Preconditions** | OCP cluster installed on vSphere; all ClusterOperators Available; all nodes Ready |
| **Steps** | 1. Verify all ClusterOperators report Available=True, Degraded=False. 2. Verify all nodes report Ready. 3. Verify all MachineConfigPools report Updated=True, Degraded=False. 4. Verify existing PVCs are Bound and pods using them are Running. |
| **Expected Result** | All checks pass; baseline artifact captured |
| **Evidence** | `oc get co`, `oc get nodes`, `oc get mcp`, `oc get pvc -A` output snapshots |

### TC-02: In-Place vCenter Upgrade Execution

| Field | Detail |
|-------|--------|
| **ID** | TC-02 |
| **Objective** | Execute VCSA in-place upgrade with preserved network identity |
| **Preconditions** | TC-01 passed; vCenter source version documented; upgrade ISO/media ready |
| **Steps** | 1. Initiate VCSA in-place upgrade per VMware procedure. 2. Confirm hostname/IP, MAC addresses, and VM UUIDs are unchanged post-upgrade. 3. Confirm vCenter services report healthy. |
| **Expected Result** | VCSA upgrade completes; vCenter identity preserved; vCenter API reachable |
| **Evidence** | vCenter build number before/after; govc or DCLI identity verification |

### TC-03: Machine and MachineSet Health Post-Upgrade

| Field | Detail |
|-------|--------|
| **ID** | TC-03 |
| **Objective** | All Machines report Running; providerIDs map correctly |
| **Preconditions** | TC-02 completed |
| **Steps** | 1. List Machines (`oc get machines -n openshift-machine-api`). 2. Verify phase=Running for all. 3. Verify providerID on each node matches the VM UUID in vCenter. 4. Scale a MachineSet +1/−1 and confirm lifecycle works. |
| **Expected Result** | Machines healthy; providerID correct; scale operations succeed |
| **Evidence** | Machine list, providerID comparison, MachineSet events |

### TC-04: Cloud Controller Manager Reconnection

| Field | Detail |
|-------|--------|
| **ID** | TC-04 |
| **Objective** | CCM reconnects to upgraded vCenter; node addresses and topology are correct |
| **Preconditions** | TC-02 completed |
| **Steps** | 1. Check CCM pod logs for successful vCenter connection. 2. Verify node `.status.addresses` include correct InternalIP and ExternalIP. 3. Verify node topology labels (region/zone) match expected values. 4. Reference CPI e2e assertions from `cpi_vm_test.go` for comparison. |
| **Expected Result** | CCM connected; node addresses and topology labels correct |
| **Evidence** | CCM pod logs; `oc get nodes -o yaml` extracts |

### TC-05: MCO Reconciliation

| Field | Detail |
|-------|--------|
| **ID** | TC-05 |
| **Objective** | MCO reconciles after vCenter upgrade; CO and MCP healthy |
| **Preconditions** | TC-02 completed |
| **Steps** | 1. Verify ClusterOperator `machine-config` is Available=True, Degraded=False. 2. Verify all MachineConfigPools show Updated=True, Degraded=False. 3. Confirm no unexpected node reboots or config drift. |
| **Expected Result** | MCO fully reconciled; no degradation |
| **Evidence** | `oc get co machine-config -o yaml`, `oc get mcp`, MCO pod logs |

### TC-06: PV/PVC Accessibility and Data Continuity

| Field | Detail |
|-------|--------|
| **ID** | TC-06 |
| **Objective** | Existing persistent volumes remain accessible; data integrity preserved |
| **Preconditions** | TC-01 created test PVCs with written data; TC-02 completed |
| **Steps** | 1. Verify pre-existing PVCs remain Bound. 2. Read data from mounted volumes; compare checksums to baseline. 3. Write new data and confirm success. 4. Restart a StatefulSet pod; confirm volume reattaches and data persists. 5. Verify CSI node and controller pods are healthy. 6. Verify volume attachment and CNS metadata via CSI driver checks (reference `tests/e2e/` patterns). |
| **Expected Result** | All PVCs Bound; data reads match baseline; new writes succeed; StatefulSet volumes reattach |
| **Evidence** | PVC status, checksum comparison, CSI pod logs, `oc describe volumeattachment` |

### TC-07: Workload Continuity During Maintenance Window

| Field | Detail |
|-------|--------|
| **ID** | TC-07 |
| **Objective** | Application pods remain running or recover within tolerance during vCenter upgrade |
| **Preconditions** | TC-01 deployed a long-running test workload; TC-02 in progress |
| **Steps** | 1. Deploy a canary Deployment and a monitoring probe before upgrade. 2. During vCenter upgrade, monitor pod status and API availability. 3. After upgrade, confirm pods are Running and probe gaps are within tolerance. |
| **Expected Result** | API remains reachable; pod disruption within tolerance (TBD — tolerance values not yet defined) |
| **Evidence** | Probe logs with timestamps; pod event timeline |

### TC-08: Delayed vCenter Availability / Retry

| Field | Detail |
|-------|--------|
| **ID** | TC-08 |
| **Objective** | Components recover after extended vCenter unavailability during upgrade window |
| **Preconditions** | TC-02 scenario but vCenter services are slow to return |
| **Steps** | 1. Simulate or observe prolonged vCenter API downtime (upgrade window). 2. Monitor CCM, CSI, and Machine API controller retry/backoff logs. 3. After vCenter returns, confirm all controllers reconnect without manual intervention. |
| **Expected Result** | All controllers recover automatically; no manual restarts required |
| **Evidence** | Controller pod logs showing reconnection; CO status timeline |

### TC-09: Negative — Changed Identity Rejection

| Field | Detail |
|-------|--------|
| **ID** | TC-09 |
| **Objective** | Confirm that changed vCenter hostname/IP is out of scope and documented as unsupported |
| **Preconditions** | Documentation procedure (OSDOCS-20751) available |
| **Steps** | 1. Review published procedure for explicit statement that changed identity is unsupported. 2. (Optional) If feasible in test env, change vCenter hostname and confirm cluster does NOT recover automatically. |
| **Expected Result** | Procedure documents the limitation; cluster does not auto-recover with changed identity |
| **Evidence** | Documentation excerpt; cluster error state (if tested) |

### TC-10: Cross-vCenter Migration Rejection

| Field | Detail |
|-------|--------|
| **ID** | TC-10 |
| **Objective** | Confirm cross-vCenter migration is out of scope and documented as unsupported |
| **Preconditions** | Documentation procedure available |
| **Steps** | 1. Review documentation for explicit exclusion of cross-vCenter migration. |
| **Expected Result** | Documented as unsupported; no acceptance test coverage expected |
| **Evidence** | Documentation excerpt |

### TC-11: Documentation Procedure Validation

| Field | Detail |
|-------|--------|
| **ID** | TC-11 |
| **Objective** | Published upgrade procedure is complete and can be followed end-to-end |
| **Preconditions** | OSDOCS-20751 procedure published (TBD) |
| **Steps** | 1. Follow the published procedure step-by-step on a test cluster. 2. Note any missing, ambiguous, or incorrect steps. 3. Confirm all prerequisite and post-upgrade verification steps are included. |
| **Expected Result** | Procedure is complete and accurate; no undocumented manual steps |
| **Evidence** | Annotated walkthrough log; filed doc bugs if any |

## 9. Acceptance Criteria Traceability Matrix

| Acceptance Criterion (from OCPSTRAT-3479) | Test ID(s) | Feature ID | Status |
|--------------------------------------------|------------|------------|--------|
| Cluster survives vCenter upgrade without redeployment | TC-02, TC-03 | F-2, F-3 | Planned |
| Machines recover healthy | TC-03 | F-3 | Planned |
| MCO reconciles after vCenter identity change | TC-05 | F-5 | Planned |
| Persistent Volumes remain accessible | TC-06 | F-6 | Planned |
| Supported procedure documented | TC-11 | F-9 | Planned (doc TBD) |
| Cloud controller reconnects and nodes healthy | TC-04 | F-4 | Planned |
| Workloads remain available during window | TC-07 | F-7 | Planned |
| Component retry/recovery on delayed availability | TC-08 | F-8 | Planned |
| Changed identity is out-of-scope / unsupported | TC-09, TC-10 | NF-1, NF-2 | Planned |

## 10. Item Pass/Fail Criteria

- **Pass:** All test cases TC-01 through TC-11 pass with expected results met. No ClusterOperator is Degraded. No data loss on persistent volumes. Documentation procedure is followable.
- **Fail:** Any ClusterOperator remains Degraded after reconciliation timeout (TBD). Any PV data loss. Any Machine stuck in non-Running phase. Documentation has blocking gaps.

## 11. Suspension and Resumption Criteria

- **Suspend:** vCenter upgrade fails or leaves vCenter in an unrecoverable state; cluster is destroyed or unreachable; blocking infrastructure issue.
- **Resume:** Environment restored to a known-good state matching TC-01 preconditions.

## 12. Test Deliverables

| Deliverable | Format | Owner |
|-------------|--------|-------|
| This test plan | Markdown | TBD |
| Test case execution log | Spreadsheet or CI artifacts | TBD |
| Baseline/post-upgrade cluster snapshots | `oc adm inspect` bundles | TBD |
| Defect reports | Jira issues linked to SPLAT-2926 | TBD |
| Automation scripts (if developed) | Shell / Go in openshift repos | TBD |

## 13. Testing Tasks

| Task | Dependency | Owner | Status |
|------|-----------|-------|--------|
| Finalize OCP and vCenter version matrix | Needs refinement of SPLAT-2926 | TBD | TBD |
| Provision vSphere test environment with upgradeable VCSA | Version matrix | TBD | TBD |
| Develop or adapt automation for vCenter upgrade step | Environment ready | TBD | TBD |
| Execute TC-01 through TC-08 | Automation or manual procedure | TBD | TBD |
| Execute TC-09, TC-10 (negative/boundary) | Documentation available | TBD | TBD |
| Execute TC-11 (documentation walkthrough) | OSDOCS-20751 published | TBD | TBD |
| Collect and archive evidence artifacts | Test execution | TBD | TBD |

## 14. Environmental Needs

| Need | Detail |
|------|--------|
| vSphere infrastructure | ESXi hosts, shared datastore, networking — versions TBD |
| VCSA (source version) | TBD — must be upgradeable to target |
| VCSA (target version) | TBD |
| OCP cluster | Installed on vSphere with MAPI machines, PVCs, test workloads |
| Test tooling | `oc`, `govc` / govmomi, `openshift-tests` (subset TBD) |
| Network access | Cluster API, vCenter API, ESXi management |

## 15. Responsibilities

| Role | Responsibility |
|------|---------------|
| SPLAT team | Test plan authorship, execution, defect triage |
| Storage team | CSI/PV test cases (TC-06) and CSI e2e guidance |
| MCO team | MCO reconciliation test cases (TC-05) |
| Cluster Infrastructure team | Machine API / CCM test cases (TC-03, TC-04) |
| Documentation team | Procedure authorship (OSDOCS-20751) |

## 16. Staffing and Training Needs

- Familiarity with VCSA upgrade procedures.
- Access to vSphere environment with admin credentials.
- Knowledge of OCP vSphere-specific operators (CCM, CSI, MCO, Machine API).
- Ability to run or adapt existing e2e test suites from R6, R7, R8.

## 17. Schedule

| Milestone | Target | Notes |
|-----------|--------|-------|
| Test plan approval | TBD | Blocked on SPLAT-2926 refinement |
| Environment provisioned | TBD | Blocked on version matrix |
| Test execution — manual | TBD | |
| Test execution — automated (if applicable) | TBD | Depends on CI integration |
| Results report | TBD | |

## 18. Risks and Contingencies

| Risk | Impact | Mitigation |
|------|--------|------------|
| Version matrix not finalized | Cannot provision environment | Escalate refinement of SPLAT-2926 |
| vCenter upgrade fails in test env | Blocks all post-upgrade tests | Maintain environment snapshots; document restore procedure |
| CSI or CCM changes land during test window | Test results may not reflect GA code | Pin component versions; retest after code freeze |
| Documentation (OSDOCS-20751) not ready | TC-09, TC-10, TC-11 cannot execute | Test remaining cases; defer doc validation |
| Workload tolerance thresholds undefined | TC-07 pass/fail criteria unclear | Agree on SLOs during refinement |
| No rollback design exists | TC for rollback cannot be written | Mark NF-4 as TBD until design clarifies |

## 19. Approvals

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Test Lead | TBD | | |
| Engineering Lead | TBD | | |
| Product Owner | TBD | | |

## 20. Open Questions / Refinement Needed

1. **OCP version scope** — Which OCP versions are in scope? (Blocks environment provisioning and version matrix.)
2. **vCenter version matrix** — Source and target VCSA versions for test coverage. (Blocks TC-02 and environment setup.)
3. **Workload disruption tolerance** — What is the acceptable API/pod disruption window during vCenter upgrade? (Blocks TC-07 pass/fail criteria.)
4. **Rollback/recovery design** — Is a rollback procedure defined? If so, a test case is needed. (Currently NF-4 / TBD.)
5. **CI automation plan** — Will a new CI job orchestrate the vCenter upgrade, or is this manual-only initially? (Blocks automation task.)
6. **Exact e2e test subset** — Which existing e2e tests from CPI, CSI, and MCO suites should be run post-upgrade? (Blocks automation scope.)
7. **ESXi version constraints** — Are there minimum ESXi versions required for the upgrade path?
8. **Documentation completeness** — When will OSDOCS-20751 be ready for validation? (Blocks TC-09, TC-10, TC-11.)
9. **Reconciliation timeout SLOs** — How long should we wait for MCO/CCM/CSI to reconcile before declaring failure? (Blocks pass/fail criteria in section 10.)
10. **MachineSet scale-out during upgrade** — Should we test creating new Machines while vCenter is mid-upgrade, or only after? (Scope question for TC-03.)
