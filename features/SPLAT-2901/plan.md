# Test Plan: OpenShift Cluster Migration on vSphere Phase 2

## IEEE 829-2008 Level Test Plan

| Field | Value |
|---|---|
| **Identifier** | SPLAT-2901-LTP-001 |
| **Version** | 0.1-DRAFT |
| **Status** | Draft -- pending spike completion and design decisions |
| **Date** | 2026-09-03 |
| **Author** | Engineering (auto-generated from source analysis) |
| **Epic** | [SPLAT-2901](https://redhat.atlassian.net/browse/SPLAT-2901) |
| **Repository** | [openshift/vcf-migration-operator](https://github.com/openshift/vcf-migration-operator) (main branch) |

---

## 1. Introduction

### 1.1 Purpose

This document defines the Level Test Plan for SPLAT-2901, "OpenShift Cluster Migration on vSphere Phase 2." It covers the test strategy, conditions, and cases required to validate UPI cluster migration and cross-vCenter vMotion capabilities in the `vcf-migration-operator`.

### 1.2 Epic Status Caveat

**SPLAT-2901 is currently a placeholder epic.** Its description contains only the OCP/Telco Definition of Done template with unfilled sections (Epic Goal, Scenarios, Acceptance Criteria are all stub markers). The epic carries the `needs-refinement` label. It has **no defined acceptance criteria of its own** beyond the template boilerplate.

All scope in this plan is derived from the four verified child issues:

| Child Issue | Type | Status | Summary |
|---|---|---|---|
| [SPLAT-2703](https://redhat.atlassian.net/browse/SPLAT-2703) | Story | Backlog | Control plane migration for UPI clusters |
| [SPLAT-2704](https://redhat.atlassian.net/browse/SPLAT-2704) | Story | Backlog | Worker migration for UPI clusters |
| [SPLAT-2708](https://redhat.atlassian.net/browse/SPLAT-2708) | Spike | In Progress | Investigate how to solve UPI migration |
| [SPLAT-2801](https://redhat.atlassian.net/browse/SPLAT-2801) | Spike | In Progress | Investigate cross-vCenter vMotion for UPI VCF migration operator |

SPLAT-2900 is unrelated to this epic and is explicitly excluded from scope.

### 1.3 Scope Boundary: IPI Baseline vs. Phase 2 UPI/vMotion

The current `vcf-migration-operator` implementation is **IPI-only**. All workload migration logic assumes the existence of MachineSets and a ControlPlaneMachineSet (CPMS). UPI clusters lack these Machine API objects and cannot be migrated by the current operator.

This plan explicitly separates:

1. **Baseline IPI coverage** -- existing implementation and tests, verified in the current source.
2. **Proposed UPI coverage** (SPLAT-2703, SPLAT-2704) -- not yet implemented; test cases are provisional and blocked on design decisions from SPLAT-2708.
3. **Proposed cross-vCenter vMotion coverage** (SPLAT-2801) -- not yet implemented; test cases are provisional and blocked on spike completion.

---

## 2. References

### 2.1 Jira Issues

- [SPLAT-2901 (Epic)](https://redhat.atlassian.net/browse/SPLAT-2901) -- Placeholder epic
- [SPLAT-2703](https://redhat.atlassian.net/browse/SPLAT-2703) -- UPI control plane migration
- [SPLAT-2704](https://redhat.atlassian.net/browse/SPLAT-2704) -- UPI worker migration
- [SPLAT-2708](https://redhat.atlassian.net/browse/SPLAT-2708) -- Investigate UPI migration approach
- [SPLAT-2801](https://redhat.atlassian.net/browse/SPLAT-2801) -- Investigate cross-vCenter vMotion
- [SPLAT-2654](https://redhat.atlassian.net/browse/SPLAT-2654) -- IPI control plane migration (Closed, cloned by SPLAT-2703)
- [SPLAT-2655](https://redhat.atlassian.net/browse/SPLAT-2655) -- IPI worker migration (Closed, cloned by SPLAT-2704)
- [OCPSTRAT-3598](https://redhat.atlassian.net/browse/OCPSTRAT-3598) -- Parent strategy item

### 2.2 Source Repository

- Repository: <https://github.com/openshift/vcf-migration-operator>
- Branch: `main`
- Key source paths verified:
  - `api/v1alpha1/vmwarecloudfoundationmigration_types.go` -- CRD types (MigrationState, conditions, spec/status)
  - `internal/controller/vmwarecloudfoundationmigration_controller.go` -- Reconciler, condition handlers
  - `internal/controller/preflight.go` -- Preflight checks (feature gate, CSI, PVs, privileges, vSphere connectivity)
  - `internal/controller/helpers.go` -- Helper functions
  - `internal/openshift/machines.go` -- MachineSet/CPMS management
  - `internal/openshift/infrastructure.go` -- Infrastructure CR management
  - `internal/openshift/secrets.go` -- Secret management
  - `internal/openshift/configmaps.go` -- ConfigMap management
  - `internal/openshift/pods.go` -- Pod management (restart vSphere pods)
  - `internal/openshift/operators.go` -- ClusterOperator health checks
  - `internal/openshift/machineconfigpools.go` -- MachineConfigPool convergence
  - `internal/openshift/version.go` -- Version/feature gate checks
  - `internal/vsphere/session.go` -- vSphere session management
  - `internal/vsphere/folder.go` -- VM folder creation
  - `internal/vsphere/tags.go` -- Tag/category operations
  - `internal/metadata/metadata.go` -- Migration metadata generation
  - `config/samples/migration_v1alpha1_vmwarecloudfoundationmigration.yaml` -- Sample CR

### 2.3 Existing Test Files Verified

| Test File | Type | Framework |
|---|---|---|
| `internal/controller/suite_test.go` | Integration (envtest) | Ginkgo/Gomega |
| `internal/controller/vmwarecloudfoundationmigration_controller_test.go` | Integration + Unit | Ginkgo + testing.T |
| `internal/controller/preflight_test.go` | Unit + Integration (govmomi simulator) | testing.T |
| `internal/controller/destination_initialized_test.go` | Integration (govmomi simulator) | testing.T |
| `internal/controller/workload_migration_rollout_test.go` | Unit (fake clients) | testing.T |
| `internal/controller/ready_test.go` | Unit (fake clients) | testing.T |
| `internal/openshift/machines_test.go` | Unit | testing.T |
| `internal/openshift/infrastructure_test.go` | Unit | testing.T |
| `internal/openshift/secrets_test.go` | Unit | testing.T |
| `internal/openshift/configmaps_test.go` | Unit | testing.T |
| `internal/openshift/pods_test.go` | Unit | testing.T |
| `internal/openshift/operators_test.go` | Unit | testing.T |
| `internal/openshift/machineconfigpools_test.go` | Unit | testing.T |
| `internal/openshift/version_test.go` | Unit | testing.T |
| `internal/vsphere/session_test.go` | Unit | testing.T |
| `internal/vsphere/folder_test.go` | Unit (govmomi simulator) | testing.T |
| `internal/vsphere/tags_test.go` | Unit (govmomi simulator) | testing.T |
| `internal/metadata/metadata_test.go` | Unit | testing.T |
| `test/e2e/e2e_test.go` | E2E (Kind cluster) | Ginkgo/Gomega |

### 2.4 External References

- [OKD vSphere UPI Installation Requirements](https://docs.okd.io/4.18/installing/installing_vsphere/upi/upi-vsphere-installation-reqs.html) (referenced by SPLAT-2708)
- [Broadcom KB 433258 -- Cross-vCenter migration CNS support](https://knowledge.broadcom.com/external/article/433258/) (referenced by SPLAT-2801)
- [govmomi issue #1975](https://github.com/vmware/govmomi/issues/1975) -- govc CLI cross-vCenter limitation (referenced by SPLAT-2801)
- [govmomi issue #1600](https://github.com/vmware/govmomi/issues/1600) -- cross-vCenter support in govmomi library (referenced by SPLAT-2801)

---

## 3. Test Items

### 3.1 Current Implementation (IPI Baseline)

The following components are implemented and testable today:

| Item | Source Path | Description |
|---|---|---|
| VmwareCloudFoundationMigration CRD | `api/v1alpha1/vmwarecloudfoundationmigration_types.go` | Singleton CR with Pending/Running/Paused states |
| Reconciler condition pipeline | `internal/controller/vmwarecloudfoundationmigration_controller.go` | Ordered: InfrastructurePrepared, DestinationInitialized, MultiSiteConfigured, WorkloadMigrated, SourceCleaned, Ready |
| Preflight checks | `internal/controller/preflight.go` | Feature gate, operator health, CSI PV blocker, CSI driver state, MHC/autoscaler detection, vSphere connectivity and privilege validation |
| Destination initialization | `internal/controller/vmwarecloudfoundationmigration_controller.go` (ensureDestinationInitialized) | VM folder creation, region/zone tag creation and attachment, cluster ownership tag |
| Multi-site configuration | `internal/controller/vmwarecloudfoundationmigration_controller.go` (ensureMultiSiteConfigured) | Add target vCenter to secrets/Infrastructure/cloud-provider-config, restart pods, wait for readiness |
| Workload migration (IPI) | `internal/controller/vmwarecloudfoundationmigration_controller.go` (ensureWorkloadMigrated) | Create target MachineSets from source, wait for worker readiness, update CPMS failure domains, wait for control plane rollout, scale old MachineSets to 0, delete source MachineSets |
| Source cleanup | `internal/controller/vmwarecloudfoundationmigration_controller.go` (ensureSourceCleaned) | Remove source vCenter from Infrastructure/config/secrets, restart pods, generate metadata |
| Ready gate | `internal/controller/vmwarecloudfoundationmigration_controller.go` (ensureReady) | Sustained stability (6 consecutive stable observations at 30s intervals), operator health, MachineConfigPool convergence, Infrastructure vCenter verification |

### 3.2 Proposed Implementation (Phase 2 -- Not Yet Implemented)

| Item | Jira | Description | Status |
|---|---|---|---|
| UPI control plane shape discovery | SPLAT-2703 | Define authoritative source for UPI control plane VM shape | Blocked on SPLAT-2708 |
| UPI control plane target synthesis | SPLAT-2703 | Synthesize target template/providerSpec for UPI control plane | Blocked on SPLAT-2708 |
| UPI worker shape discovery | SPLAT-2704 | Discover or accept user-supplied worker VM shape | Blocked on SPLAT-2708 |
| UPI worker target synthesis | SPLAT-2704 | Synthesize destination inputs for UPI workers | Blocked on SPLAT-2708 |
| UPI static IP / guest customization handling | SPLAT-2703, SPLAT-2704 | Explicitly handle or declare unsupported | Blocked on SPLAT-2708 |
| UPI external IPAM handling | SPLAT-2703, SPLAT-2704 | Explicitly handle or declare unsupported | Blocked on SPLAT-2708 |
| cross-vCenter vMotion (govmomi Relocate) | SPLAT-2801 | VirtualMachine.Relocate with ServiceLocator | Spike in progress |
| Cold-move fallback | SPLAT-2801 | Powered-off Relocate for CPU vendor mismatch | Spike in progress |
| vMotion concurrency control | SPLAT-2801 | Safe parallelism limits for Relocate operations | Spike in progress |
| Cluster config repointing | SPLAT-2801 | Update vsphere-creds, cloud-provider-config, Infrastructure CR after vMotion | Spike in progress |

---

## 4. Features to Be Tested

### 4.1 IPI Baseline (Existing -- Verification and Regression)

1. Singleton resource enforcement (only `name: cluster` is reconciled)
2. State-gated reconciliation (only `Running` state triggers work)
3. Ordered condition pipeline progression
4. Preflight validation (all blockers)
5. Destination initialization (folders, tags, idempotency, concurrency safety)
6. Multi-site configuration (secrets, Infrastructure, ConfigMap, pod restart, pod readiness wait)
7. Worker MachineSet creation from source MachineSets
8. Worker and node readiness gating
9. CPMS failure domain update and control plane rollout
10. Source MachineSet scale-down and deletion
11. Source cleanup (Infrastructure, ConfigMap, secrets, metadata generation)
12. Ready gate with sustained stability counter
13. Status merge safety (concurrent writer, stale generation, stale failure protection)

### 4.2 UPI Migration (Proposed -- SPLAT-2703, SPLAT-2704)

1. UPI cluster detection (absence of MachineSets/CPMS)
2. UPI control plane shape discovery or user input
3. UPI worker shape discovery or user input
4. Static IP and guest customization handling/declaration
5. External IPAM handling/declaration
6. UPI-specific prerequisites documentation validation
7. Drain/disruption/recovery behavior for UPI workers
8. Non-reliance on IPI-only CPMS workflow

### 4.3 Cross-vCenter vMotion (Proposed -- SPLAT-2801)

1. govmomi VirtualMachine.Relocate with ServiceLocator
2. Licensing tier detection/validation
3. Cold-move fallback (powered-off Relocate)
4. providerID and Instance UUID preservation
5. Cluster configuration repointing after move
6. vMotion concurrency limits
7. CCM node-deletion behavior during VM disappearance

---

## 5. Features Not to Be Tested

| Feature | Reason |
|---|---|
| vSphere CSI/CNS persistent volume handling after cross-vCenter vMotion | Explicitly out of scope per SPLAT-2801 description. CSI PV breakage is a known boundary/risk called out below. |
| SPLAT-2900 | Unrelated to this epic; explicitly excluded. |
| OLM bundle installation/upgrade | Covered by separate OLM CI; not a feature test item. |
| vSphere simulator fidelity against real vCenter | Simulator tests validate operator logic, not vCenter API fidelity. |
| Velero-based PV migration | Mentioned in SPLAT-2801 investigation questions but not in scope for operator testing. |
| EVC (Enhanced vMotion Compatibility) mode interaction | Not tested in the spike; documented as untested in SPLAT-2801 findings. |

---

## 6. Test Approach and Strategy

### 6.1 Test Levels

| Level | Tool/Framework | Scope | Run Command |
|---|---|---|---|
| **Unit** | `testing.T`, table-driven | Individual functions, helpers, pure logic | `go test ./internal/... -run TestXxx -v` |
| **Integration (envtest)** | Ginkgo/Gomega + controller-runtime envtest | Controller reconciliation with real API server, fake vSphere | `make test` |
| **Integration (govmomi simulator)** | `testing.T` + govmomi `simulator.VPX()` | vSphere operations against in-memory vCenter simulator | `make test` |
| **E2E smoke** | Ginkgo/Gomega + Kind cluster | Manager deployment, pod readiness, metrics endpoint | `make test-e2e` |
| **E2E functional** | TBD (real vSphere, multi-vCenter lab) | Full migration workflow on IPI/UPI clusters | TBD (blocked on lab availability) |
| **Manual/Exploratory** | Human-driven on lab environment | Cross-vCenter vMotion, cold move, UPI edge cases | TBD |

### 6.2 Strategy by Feature Area

**IPI Baseline**: Existing automated tests provide strong coverage. New test cases supplement gaps found during this analysis (see Section 10).

**UPI Migration**: Cannot be automated until design decisions from SPLAT-2708 are finalized. Test cases in this plan are provisional. Once the approach is chosen (VM re-creation vs. vMotion vs. hybrid), test automation will follow the same patterns as IPI: unit tests with fake clients, integration tests with govmomi simulator, and E2E tests against a real vSphere UPI cluster.

**Cross-vCenter vMotion**: Requires a real multi-vCenter lab for E2E validation. govmomi simulator does not support cross-vCenter Relocate. Unit tests can cover concurrency limiting logic and configuration repointing. Integration/E2E tests are blocked on lab setup and spike completion.

### 6.3 Test Data and Fixtures

- govmomi `simulator.VPX()` model for vSphere integration tests
- Kubernetes fake clients (`fakekube`, `fakemachineclient`, `configfake`, `dynamicfake`) for unit tests
- envtest with CRD from `config/crd/bases` for controller integration tests
- Kind cluster for E2E smoke tests
- Real multi-vCenter lab (TBD) for functional E2E

---

## 7. Item Pass/Fail Criteria

### 7.1 Unit and Integration Tests

- All tests pass (`make test` exit code 0).
- No test regressions from baseline.
- Code coverage does not decrease for touched packages.

### 7.2 E2E Smoke

- Manager pod reaches Running state within 2 minutes.
- Metrics endpoint responds with HTTP 200 and includes `controller_runtime_reconcile_total`.

### 7.3 E2E Functional (IPI Baseline)

- Migration CR transitions through all conditions in order: InfrastructurePrepared -> DestinationInitialized -> MultiSiteConfigured -> WorkloadMigrated -> SourceCleaned -> Ready.
- All target workers reach Ready state with corresponding Ready nodes.
- Control plane rollout completes (3/3 updated, 3/3 ready).
- Source MachineSets are deleted.
- Infrastructure CR contains only target vCenters.
- All ClusterOperators are Available=True, Progressing=False, Degraded=False.
- CompletionTime is set on the migration CR.

### 7.4 UPI Migration (Provisional)

- UPI cluster (no MachineSets/CPMS) is detected and the appropriate migration path is selected.
- Control plane nodes are migrated without assuming CPMS workflow.
- Worker nodes are migrated without assuming MachineSet workflow.
- Static IP / guest customization / external IPAM constraints are enforced or documented.
- Cluster returns to healthy state post-migration.

### 7.5 Cross-vCenter vMotion (Provisional)

- VMs are relocated to target vCenter with providerID and UUID preserved.
- Cold-move fallback works when CPU vendor mismatch prevents live vMotion.
- Cluster configuration is correctly repointed after move.
- Concurrency limits are respected.

---

## 8. Suspension and Resumption Criteria

### 8.1 Suspension Criteria

- **Spike blockers**: Testing for SPLAT-2703/2704 features is suspended until SPLAT-2708 delivers design decisions.
- **vMotion testing**: Suspended until SPLAT-2801 spike resolves licensing and cold-move questions.
- **Lab unavailability**: E2E functional testing is suspended if multi-vCenter lab is unavailable.
- **Build failures**: All testing suspended if `make build` or `make test` fails on the baseline.

### 8.2 Resumption Criteria

- SPLAT-2708 spike delivers a decision document specifying the UPI migration approach.
- SPLAT-2801 spike delivers a decision on govmomi Relocate viability and licensing.
- Multi-vCenter lab is provisioned and accessible.
- Baseline build and tests pass.

---

## 9. Deliverables

| Deliverable | Status |
|---|---|
| This test plan (SPLAT-2901-LTP-001) | Draft |
| Automated unit/integration test cases for IPI baseline | Existing (see Section 2.3) |
| Automated test cases for UPI migration | TBD -- blocked on SPLAT-2708 |
| Automated test cases for cross-vCenter vMotion unit logic | TBD -- blocked on SPLAT-2801 |
| E2E functional test suite for IPI migration | TBD -- requires lab |
| E2E functional test suite for UPI migration | TBD -- blocked on implementation |
| E2E functional test suite for vMotion migration | TBD -- blocked on implementation and lab |
| Test results report | TBD |
| Polarion test plan (per Done Checklist) | TBD |

---

## 10. Detailed Test Conditions and Cases

### 10.1 Legend

| Column | Meaning |
|---|---|
| **ID** | Unique test case identifier |
| **Traceability** | Jira issue(s) this case validates |
| **Coverage** | `Existing` = automated and present in repo; `Proposed-Auto` = should be automated; `Proposed-Manual` = requires manual/exploratory; `Blocked` = cannot proceed until a dependency is resolved |
| **Gate** | `Acceptance` = must pass for feature acceptance; `Regression` = must pass to prevent regression; `Informational` = provides insight but is not a hard gate |

### 10.2 IPI Baseline: Reconciler Core (Existing Coverage)

| ID | Condition | Expected Result | Traceability | Coverage | Gate |
|---|---|---|---|---|---|
| TC-IPI-001 | Reconcile a CR with `name: cluster` in Pending state | Reconciler returns without error; no conditions set; no StartTime | SPLAT-2654 | Existing (`vmwarecloudfoundationmigration_controller_test.go`: "should successfully reconcile the resource") | Regression |
| TC-IPI-002 | Reconcile a CR with a non-singleton name (e.g., `not-cluster`) in Running state | Accepted=False with reason UnsupportedName; Warning event emitted; no workflow conditions set; no StartTime | SPLAT-2654 | Existing (`vmwarecloudfoundationmigration_controller_test.go`: "should ignore the resource, mark it as not accepted, and record a warning event") | Regression |
| TC-IPI-003 | Two concurrent status writers commit different conditions | Both conditions survive in final status (merge, not overwrite) | SPLAT-2654 | Existing (`vmwarecloudfoundationmigration_controller_test.go`: "does not discard a condition committed by another writer") | Regression |
| TC-IPI-004 | Stale-generation reconcile attempts to overwrite current-generation condition | Current-generation condition preserved; stale update rejected | SPLAT-2654 | Existing (`vmwarecloudfoundationmigration_controller_test.go`: "does not let a stale-generation reconcile overwrite a current-generation condition") | Regression |
| TC-IPI-005 | Stale failure attempts to overwrite concurrent success on same condition | Committed success (True) preserved; stale failure rejected | SPLAT-2654 | Existing (`vmwarecloudfoundationmigration_controller_test.go`: "does not let a stale failure overwrite a concurrent success") | Regression |
| TC-IPI-006 | sanitizeRFC1123 with various inputs (valid, underscore, uppercase, spaces, empty, all-invalid) | Correct RFC 1123 label output for each case | SPLAT-2654 | Existing (`vmwarecloudfoundationmigration_controller_test.go`: TestSanitizeRFC1123) | Regression |
| TC-IPI-007 | workerMachineSetName with various infraID/fdName combinations | Correct `{infraID}-worker-{sanitized-fdName}` output | SPLAT-2654 | Existing (`vmwarecloudfoundationmigration_controller_test.go`: TestWorkerMachineSetName) | Regression |

### 10.3 IPI Baseline: Preflight Checks (Existing Coverage)

| ID | Condition | Expected Result | Traceability | Coverage | Gate |
|---|---|---|---|---|---|
| TC-PRE-001 | Happy path: feature gate enabled, no blockers, valid vSphere connectivity | Message contains "Preflight validation passed" | SPLAT-2654 | Existing (`preflight_test.go`: "happy path passes when gate enabled") | Regression |
| TC-PRE-002 | Feature gate VSphereMultiVCenterDay2 disabled | Error containing "feature gate VSphereMultiVCenterDay2 is not enabled" | SPLAT-2654 | Existing (`preflight_test.go`: "blocks when gate disabled") | Regression |
| TC-PRE-003 | vSphere CSI PV exists in cluster | Error containing PV name; short-circuits before target validation | SPLAT-2654 | Existing (`preflight_test.go`: "csi pv blocker short circuits before target validation") | Regression |
| TC-PRE-004 | ClusterCSIDriver management state is not Removed | Error containing "ClusterCSIDriver"; short-circuits before target validation | SPLAT-2654 | Existing (`preflight_test.go`: "csi managed blocker short circuits") | Regression |
| TC-PRE-005 | MachineHealthCheck resources present (excluding termination handler) | Error listing interfering MHC resources | SPLAT-2654 | Existing (`preflight_test.go`: "interfering resource blocker short circuits") | Regression |
| TC-PRE-006 | Cluster upgrade in progress (Progressing=True) | Error "cluster upgrade is in progress" | SPLAT-2654 | Existing (`preflight_test.go`: "cluster upgrade in progress blocks migration") | Regression |
| TC-PRE-007 | Degraded ClusterOperator | Error "cluster operators are not healthy" | SPLAT-2654 | Existing (`preflight_test.go`: "degraded cluster operator blocks migration") | Regression |
| TC-PRE-008 | Missing target folder in failure domain topology | Error containing `target folder "/missing-folder"` | SPLAT-2654 | Existing (`preflight_test.go`: "missing target folder fails during target validation") | Regression |
| TC-PRE-009 | Storage/cluster managementState is Managed but CSI is Removed | Warning in message; not a blocker | SPLAT-2654 | Existing (`preflight_test.go`: "storage managed warns but does not block") | Regression |
| TC-PRE-010 | No vSphere CSI PVs (non-CSI PVs present) | Passes without error | SPLAT-2654 | Existing (`preflight_test.go`: TestCheckNoVSphereCSIPersistentVolumes "passes without vsphere csi pvs") | Regression |
| TC-PRE-011 | machine-api-termination-handler MHC present (no user MHCs) | Passes; termination handler is allow-listed | SPLAT-2654 | Existing (`preflight_test.go`: "skips platform-default machine-api-termination-handler") | Regression |
| TC-PRE-012 | Duplicate failure domain names in spec | Error "duplicate failure domain names are not allowed" | SPLAT-2654 | Existing (`preflight_test.go`: TestValidateUniqueFailureDomainNames) | Regression |
| TC-PRE-013 | ClusterAutoscaler resource present | Error listing ClusterAutoscaler as interfering | SPLAT-2654 | Existing (`preflight_test.go`: "fails when cluster autoscaler exists") | Regression |
| TC-PRE-014 | vSphere privilege validation (missing privilege, denied privilege, empty requested) | Correctly identifies missing/denied privileges | SPLAT-2654 | Existing (`preflight_test.go`: TestMissingPrivileges) | Regression |
| TC-PRE-015 | Target vCenter configuration detection (all combos: missing servers, missing FDs, fully present) | Correct boolean for hasTargetVCenterConfiguration | SPLAT-2654 | Existing (`preflight_test.go`: TestHasTargetVCenterConfiguration) | Regression |

### 10.4 IPI Baseline: Destination Initialization (Existing Coverage)

| ID | Condition | Expected Result | Traceability | Coverage | Gate |
|---|---|---|---|---|---|
| TC-DEST-001 | First reconcile: folder, ownership tag, and failure domain tags needed | Folder created, ownership tag attached, region/zone tags created and attached; DestinationInitialized=True | SPLAT-2654 | Existing (`destination_initialized_test.go`: "creates folder, ownership tag, and failure domain tags on first reconcile") | Regression |
| TC-DEST-002 | Ownership tag already attached from prior reconcile (with narrow associable types) | Skips re-validation; DestinationInitialized=True | SPLAT-2654 | Existing (`destination_initialized_test.go`: "skips re-validating the ownership category when the folder already has the tag attached") | Regression |
| TC-DEST-003 | Repeat reconcile after full initialization | Succeeds without error; skips already-configured target | SPLAT-2654 | Existing (`destination_initialized_test.go`: "a repeat reconcile after full initialization still succeeds") | Regression |
| TC-DEST-004 | Two concurrent reconciles from cold state | Both succeed; folder has ownership tag at the end | SPLAT-2654 | Existing (`destination_initialized_test.go`: "two concurrent reconciles from a cold state both succeed") | Regression |

### 10.5 IPI Baseline: Workload Migration Rollout (Existing Coverage)

| ID | Condition | Expected Result | Traceability | Coverage | Gate |
|---|---|---|---|---|---|
| TC-WKL-001 | CPMS generation not yet observed | Requeue 15s; message "Waiting for control plane rollout to start (CPMS generation 2/1 observed)" | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: "requeues while CPMS generation is not observed") | Regression |
| TC-WKL-002 | Control plane rolling out (1/3 updated, 1/3 ready) | Requeue 30s; message "Control plane rolling out (1/3 updated, 1/3 ready)"; event emitted | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: "reports progress while control plane is rolling out") | Regression |
| TC-WKL-003 | Control plane rollout complete, source MachineSets with replicas > 0 | Source MachineSets scaled to 0; requeue 30s | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: "scales source machinesets down to zero and requeues") | Regression |
| TC-WKL-004 | All source machines/nodes deleted, source MachineSets at 0 replicas | Source MachineSets deleted; WorkloadMigrated=True | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: "completes workload migration and deletes zero-replica source machinesets") | Regression |
| TC-WKL-005 | Old worker deletion stalled (machine with DrainError) | Requeue 30s; condition message includes machine name, error reason, and node count | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: "reports old worker deletion stall detail") | Regression |
| TC-WKL-006 | Target workers ready, CPMS already targets target FDs | Routes to rollout path; requeue 15s for rollout start | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: TestEnsureWorkloadMigratedRolloutGate "routes to rollout path") | Regression |
| TC-WKL-007 | CPMS not updated, target workers not ready | Stays in worker phase; requeue 30s | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: "stays in worker phase when CPMS not updated") | Regression |
| TC-WKL-008 | Target workers partially ready (1/2 machines, 1/2 nodes) | Reports readiness counts; requeue 30s | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: "reports worker readiness counts while target workers are pending") | Regression |
| TC-WKL-009 | Target MachineSets missing; source MachineSets exist | Creates target MachineSets; requeue 30s | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: "stays in worker phase when target machinesets missing") | Regression |
| TC-WKL-010 | Stall event debouncing: same machines -> no re-emit; different machines -> new event; many machines -> bounded note | Event emitted once per distinct machine set; note capped at 1024 bytes with omission summary | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: TestOldWorkersStallEventDebounce) | Regression |
| TC-WKL-011 | Rollout logs machine-level detail without sensitive data | Log contains machine names, phases, error reasons; does not contain node names or error messages | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: TestRolloutLogsMachineLevelDetail) | Regression |
| TC-WKL-012 | Condition message bounding for large stall detail | Message truncated to 32768 chars with omitted machine count | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: TestBoundConditionMessage) | Regression |
| TC-WKL-013 | Stall detail sorting (machinesets and machines sorted alphabetically) | Detail and key sorted by MachineSet name, then by machine name within | SPLAT-2654 | Existing (`workload_migration_rollout_test.go`: TestOldWorkerStallDetailSorting) | Regression |

### 10.6 IPI Baseline: Ready Gate (Existing Coverage)

| ID | Condition | Expected Result | Traceability | Coverage | Gate |
|---|---|---|---|---|---|
| TC-RDY-001 | Operator progressing (etcd Progressing=True) | Requeue 30s; message contains "progressing=etcd" | SPLAT-2654 | Existing (`ready_test.go`: TestEnsureReadyRequeuesWhenOperatorProgressing) | Regression |
| TC-RDY-002 | All stable: require sustained stability before Ready=True | N-1 observations requeue with "Waiting for sustained cluster stability (N/6)"; Nth observation sets Ready=True and CompletionTime | SPLAT-2654 | Existing (`ready_test.go`: TestEnsureReadyRequiresSustainedStabilityBeforeCompletion) | Regression |
| TC-RDY-003 | MachineConfigPool not converged (master pool Updated=False) | Requeue 30s; message contains "pools-not-updated=master" | SPLAT-2654 | Existing (`ready_test.go`: TestEnsureReadyBlocksWhenPoolNotConverged) | Regression |
| TC-RDY-004 | Unstable observation resets stability counter | Counter resets to 0; full window must re-accumulate | SPLAT-2654 | Existing (`ready_test.go`: TestEnsureReadyResetsStabilityCounterOnUnstableObservation) | Regression |
| TC-RDY-005 | Long gap between ensureReady checks resets counter | Counter resets to 1 on first observation after gap > 90s | SPLAT-2654 | Existing (`ready_test.go`: TestEnsureReadyResetsCounterAfterLongEnsureReadyGap) | Regression |
| TC-RDY-006 | Rapid stable observations (event-driven) are not all counted | Counter stays at 1 after N rapid observations; only observations >= 30s apart count | SPLAT-2654 | Existing (`ready_test.go`: TestEnsureReadyDoesNotCountRapidStableObservations) | Regression |
| TC-RDY-007 | Check errors (operator, pool, infrastructure) reset stability counter | Counter resets to 0 on any check error; next stable observation starts at 1/N | SPLAT-2654 | Existing (`ready_test.go`: TestEnsureReadyResetsStabilityCounterOnCheckErrors) | Regression |

### 10.7 IPI Baseline: E2E Smoke (Existing Coverage)

| ID | Condition | Expected Result | Traceability | Coverage | Gate |
|---|---|---|---|---|---|
| TC-E2E-001 | Deploy manager to Kind cluster | Controller-manager pod reaches Running state | SPLAT-2654 | Existing (`test/e2e/e2e_test.go`: "should run successfully") | Regression |
| TC-E2E-002 | Metrics endpoint accessible | HTTP 200; response contains `controller_runtime_reconcile_total` | SPLAT-2654 | Existing (`test/e2e/e2e_test.go`: "should ensure the metrics endpoint is serving metrics") | Regression |

### 10.8 Proposed IPI Gaps (No Existing Coverage)

| ID | Condition | Expected Result | Traceability | Coverage | Gate |
|---|---|---|---|---|---|
| TC-IPI-GAP-001 | Paused state during active migration | Reconciler returns without further progress; conditions unchanged | SPLAT-2654 | Proposed-Auto | Regression |
| TC-IPI-GAP-002 | Transition from Running to Paused and back to Running | Migration resumes from last incomplete condition | SPLAT-2654 | Proposed-Auto | Regression |
| TC-IPI-GAP-003 | Non-target vCenter still present in Infrastructure during Ready check | Ready=False; message contains "non-target vCenter ... still present" | SPLAT-2654 | Proposed-Auto | Regression |
| TC-IPI-GAP-004 | Multiple failure domains across different target vCenters | Folders and tags created on each target vCenter; worker MachineSets distributed across FDs | SPLAT-2654 | Proposed-Auto | Acceptance |
| TC-IPI-GAP-005 | ensureMultiSiteConfigured: config already applied, waiting for pod readiness | Only runs pod readiness check, does not re-apply config | SPLAT-2654 | Proposed-Auto | Regression |

### 10.9 UPI Control Plane Migration (SPLAT-2703) -- Proposed

> **Note**: All cases below are provisional. They are blocked on SPLAT-2708 delivering a design decision on how UPI migration is implemented (VM re-creation, vMotion, or hybrid).

| ID | Condition | Expected Result | Traceability | Coverage | Gate |
|---|---|---|---|---|---|
| TC-UPI-CP-001 | UPI cluster detected (no CPMS, no MachineSets) | Reconciler selects UPI-specific migration path (not IPI CPMS workflow) | SPLAT-2703, SPLAT-2708 | Blocked | Acceptance |
| TC-UPI-CP-002 | UPI control plane shape source of truth is determined (live VM inspection, existing Machine objects, or user input) | Correct providerSpec/template synthesized for target vCenter | SPLAT-2703 | Blocked | Acceptance |
| TC-UPI-CP-003 | UPI control plane with static IPs | Migration either handles static IP assignment or returns a clear unsupported error | SPLAT-2703 | Blocked | Acceptance |
| TC-UPI-CP-004 | UPI control plane with guest customization | Migration either handles guest customization or returns a clear unsupported error | SPLAT-2703 | Blocked | Acceptance |
| TC-UPI-CP-005 | UPI control plane with external IPAM | Migration either handles external IPAM or returns a clear unsupported error | SPLAT-2703 | Blocked | Acceptance |
| TC-UPI-CP-006 | UPI control plane migration prerequisites not met | Clear error message listing unmet prerequisites | SPLAT-2703 | Blocked | Acceptance |
| TC-UPI-CP-007 | UPI control plane migration recovery after partial failure | Migration can resume from the last successful node; cluster state is consistent | SPLAT-2703 | Blocked | Acceptance |
| TC-UPI-CP-008 | Verify UPI control plane tests do not import or rely on CPMS-specific code paths from SPLAT-2654 | Code review / test assertion that CPMS objects are not created or updated | SPLAT-2703 | Blocked | Acceptance |

### 10.10 UPI Worker Migration (SPLAT-2704) -- Proposed

> **Note**: All cases below are provisional. Blocked on SPLAT-2708.

| ID | Condition | Expected Result | Traceability | Coverage | Gate |
|---|---|---|---|---|---|
| TC-UPI-WK-001 | UPI cluster detected (no MachineSets) | Reconciler selects UPI worker migration path (not IPI MachineSet workflow) | SPLAT-2704, SPLAT-2708 | Blocked | Acceptance |
| TC-UPI-WK-002 | Worker shape discovery for UPI cluster (e.g., from live VMs or user config) | Correct replacement worker inputs synthesized | SPLAT-2704 | Blocked | Acceptance |
| TC-UPI-WK-003 | UPI worker with static IP assignment | Migration either handles static IP or returns clear unsupported error | SPLAT-2704 | Blocked | Acceptance |
| TC-UPI-WK-004 | UPI worker with guest networking configuration | Migration either handles guest networking or returns clear unsupported error | SPLAT-2704 | Blocked | Acceptance |
| TC-UPI-WK-005 | UPI worker with external IPAM | Migration either handles external IPAM or returns clear unsupported error | SPLAT-2704 | Blocked | Acceptance |
| TC-UPI-WK-006 | Workload drain behavior for UPI worker migration | Drain initiated before worker removal; PDB violations documented; disruption budget respected or force-delete documented | SPLAT-2704 | Blocked | Acceptance |
| TC-UPI-WK-007 | Recovery after UPI worker migration partial failure | Remaining workers are not orphaned; migration can resume | SPLAT-2704 | Blocked | Acceptance |
| TC-UPI-WK-008 | Verify UPI worker tests do not import or rely on MachineSet-specific code paths from SPLAT-2655 | Code review / test assertion that MachineSets are not created | SPLAT-2704 | Blocked | Acceptance |

### 10.11 UPI Migration Investigation (SPLAT-2708) -- Proposed

| ID | Condition | Expected Result | Traceability | Coverage | Gate |
|---|---|---|---|---|---|
| TC-INV-001 | Spike delivers a design document specifying the UPI migration approach | Document exists; approach is one of: VM re-creation, vMotion, hybrid, or another | SPLAT-2708 | Proposed-Manual | Acceptance |
| TC-INV-002 | Spike references installer UPI/vSphere material | Design document cites installer repo `upi/vsphere` and OKD UPI installation docs | SPLAT-2708 | Proposed-Manual | Informational |
| TC-INV-003 | Spike results feed into SPLAT-2703, SPLAT-2704, and SPLAT-2702 | Child stories are updated with implementation direction | SPLAT-2708 | Proposed-Manual | Acceptance |

### 10.12 Cross-vCenter vMotion Investigation (SPLAT-2801) -- Proposed

> **Note**: vSphere CSI/CNS persistent volume breakage is explicitly out of scope for SPLAT-2801 but is called out as a boundary/risk in Section 12.

| ID | Condition | Expected Result | Traceability | Coverage | Gate |
|---|---|---|---|---|---|
| TC-VMO-001 | govmomi VirtualMachine.Relocate with ServiceLocator between two vCenters (same CPU vendor, vSphere 8 to 8) | VM relocates successfully; Instance UUID preserved; providerID (BIOS UUID) preserved; IP/MAC preserved | SPLAT-2801 | Proposed-Manual (lab required) | Acceptance |
| TC-VMO-002 | Hot vMotion between same-version vCenters (vSphere 9 to 9) | VM relocates successfully while powered on; cluster remains functional | SPLAT-2801 | Proposed-Manual (lab required) | Acceptance |
| TC-VMO-003 | Cross-vendor CPU mismatch (AMD to Intel, vSphere 8 to 9) blocks live vMotion | Live vMotion fails; operator falls back to cold-move (drain, power off, relocate, power on) | SPLAT-2801 | Proposed-Manual (lab required) | Acceptance |
| TC-VMO-004 | Cold-move sequence: drain -> power off -> relocate -> power on | VM relocates while powered off; UUID/providerID preserved; node re-registers after power on | SPLAT-2801 | Proposed-Manual (lab required) | Acceptance |
| TC-VMO-005 | providerID goes null after cold migrate; verify recovery | CCM re-populates providerID after node boot and vCenter registration; or operator handles re-population | SPLAT-2801 | Proposed-Manual (lab required) | Acceptance |
| TC-VMO-006 | CCM deletes Node object when VM disappears from source vCenter | Node object deleted during vMotion; node re-registers on target with same name/IP (cosmetic disruption) | SPLAT-2801 | Proposed-Manual (lab required) | Informational |
| TC-VMO-007 | Cluster config left pointing at source vCenter after move | providerIDs empty, CSI TopologyRetrievalFailed, operators degraded; verifies that config must be updated before or during move | SPLAT-2801 | Proposed-Manual (lab required) | Informational |
| TC-VMO-008 | Cluster config repointing (vsphere-creds, cloud-provider-config, Infrastructure CR) before/during vMotion | After repointing, cluster stabilizes; providerIDs populated; CSI functional (for non-CSI-PV clusters) | SPLAT-2801 | Proposed-Manual (lab required) | Acceptance |
| TC-VMO-009 | vMotion concurrency: parallel worker relocation within vSphere host limits | Relocations respect per-host concurrent vMotion cap (default 2-8); no relocations fail due to resource saturation | SPLAT-2801 | Proposed-Manual (lab required) | Acceptance |
| TC-VMO-010 | PDB violations during drain in cold-move path | Force-delete stuck pods documented; static-pod guards (etcd-guard, kube-apiserver-guard) handled | SPLAT-2801 | Proposed-Manual (lab required) | Acceptance |
| TC-VMO-011 | vMotion with large disk (1TB+ storage per worker) | Sequential relocation completes; timing documented (SPLAT-2801 notes ~210 min sequential for 1TB on vSphere 9 to 9 hot move) | SPLAT-2801 | Proposed-Manual (lab required) | Informational |
| TC-VMO-012 | Licensing validation: Enterprise Plus / VCF required for live vMotion | Operator detects license tier or documentation specifies prerequisites | SPLAT-2801 | Blocked (licensing investigation pending) | Acceptance |
| TC-VMO-013 | Cold-move path: vSphere 9 to 8 with 1TB storage | Cold relocate succeeds; performance characteristics documented | SPLAT-2801 | Proposed-Manual (lab required) | Informational |

### 10.13 Boundary/Risk Test Cases

| ID | Condition | Expected Result | Traceability | Coverage | Gate |
|---|---|---|---|---|---|
| TC-BND-001 | vSphere CSI PVs present when cross-vCenter vMotion is attempted | Preflight blocks migration (existing behavior); CSI/CNS breakage risk is documented but NOT validated by this test plan (explicitly out of scope per SPLAT-2801) | SPLAT-2801 | Existing (preflight blocks CSI PVs) | Acceptance |
| TC-BND-002 | Migration attempted on non-vSphere platform | Preflight fails (Infrastructure platform type check) | SPLAT-2654 | Proposed-Auto | Regression |
| TC-BND-003 | Empty failureDomains in spec | Error "spec.failureDomains must not be empty" | SPLAT-2654 | Existing (preflight_test.go covers this path indirectly) | Regression |
| TC-BND-004 | failureDomain.topology.template missing | Error "spec.failureDomains[N].topology.template is required" | SPLAT-2654 | Proposed-Auto | Regression |

---

## 11. Environmental Needs

### 11.1 Unit and Integration Tests

| Component | Requirement |
|---|---|
| Go | Version per `go.mod` (currently Go 1.24+) |
| envtest binaries | Kubernetes 1.35.0 (per Makefile `ENVTEST_K8S_VERSION`) |
| golangci-lint | v2.1.0 (per Makefile `GOLANGCI_LINT_VERSION`) |
| govmomi simulator | Bundled via vendor (govmomi v0.52.0+) |
| Network | None (all tests use in-process simulators or fake clients) |

### 11.2 E2E Smoke Tests

| Component | Requirement |
|---|---|
| Kind | Pre-installed; cluster name `vcf-migration-operator-test-e2e` |
| Container runtime | Podman or Docker (per Makefile `CONTAINER_TOOL`) |
| Container image | Built via `make docker-build` |

### 11.3 E2E Functional Tests (TBD)

| Component | Requirement |
|---|---|
| vSphere lab | Two vCenters (source and target) with vSphere 8.x or 9.x |
| vSphere licensing | Enterprise Plus or VCF for live vMotion; Standard for cold-move fallback testing |
| OpenShift cluster (IPI) | Deployed on source vCenter with Machine API objects |
| OpenShift cluster (UPI) | Deployed on source vCenter without Machine API objects (TBD -- blocked on SPLAT-2708) |
| Network | vMotion network between source and target ESXi hosts |
| Storage | Shared or independent datastores; no vSphere CSI PVs for vMotion tests |

---

## 12. Risks and Contingencies

| Risk | Impact | Likelihood | Mitigation |
|---|---|---|---|
| SPLAT-2708 spike does not deliver a clear UPI approach | All UPI test cases (TC-UPI-*) remain blocked; Phase 2 scope unvalidated | Medium | Escalate if spike exceeds time-box; partial delivery (e.g., "re-creation only, vMotion deferred") unblocks subset of tests |
| SPLAT-2801 licensing investigation inconclusive | Cannot determine if live vMotion is universally available to target customers | Medium | Default to cold-move fallback path; document licensing prerequisite |
| vSphere CSI/CNS breakage after cross-vCenter vMotion | Any cluster with vSphere CSI PVs will have broken storage after vMotion; current preflight blocks CSI PVs but this does not address clusters that add CSI PVs after migration | High (if not blocked) | Preflight blocks migration with CSI PVs; post-migration CSI PV creation is out of scope but should be documented as a known limitation |
| CCM deletes Node objects during VM disappearance | Cosmetic disruption during vMotion; pods may be rescheduled | Medium | Documented in SPLAT-2801 findings; node re-registers with same name/IP; operator should handle or document this transient state |
| providerID goes null after cold migrate | CCM cannot find VM on target vCenter until config is repointed | High (cold-move path) | Repoint cluster config before or during cold move; documented in SPLAT-2801 spike findings |
| govmomi simulator does not support cross-vCenter Relocate | Cannot automate vMotion-specific tests at integration level | High | Rely on E2E functional tests against real vSphere; unit-test concurrency/config logic only |
| Multi-vCenter lab unavailability | E2E functional tests for both IPI and UPI migration cannot run | Medium | Prioritize lab provisioning; use CI-provided vSphere resources if available |
| PDB violations during drain block cold-move | Worker pods stuck behind PDB min-available; drain hangs | Medium | Document force-delete procedure; operator may implement configurable drain timeout |

---

## 13. Responsibilities, Staffing, and Training

| Role | Responsibility | Status |
|---|---|---|
| Dev team (SPLAT) | Implement UPI migration and vMotion features; write unit/integration tests | Active |
| QE team | Write E2E functional tests; execute manual/exploratory tests; maintain Polarion test plan | TBD |
| Spike investigators | Complete SPLAT-2708 and SPLAT-2801; deliver design documents | In Progress |
| Lab administrators | Provision multi-vCenter lab for E2E functional testing | TBD |
| Release enablement | Provide documentation for UPI migration prerequisites and limitations | TBD |

Training needs: N/A for current scope. vSphere-specific knowledge (govmomi, vMotion, CNS) is available within the team per SPLAT-2801 spike findings.

---

## 14. Schedule

| Milestone | Target Date | Dependency |
|---|---|---|
| SPLAT-2708 spike completion | TBD | In Progress |
| SPLAT-2801 spike completion | TBD | In Progress |
| UPI migration design document | TBD | SPLAT-2708 |
| UPI test case finalization | TBD | UPI design document |
| UPI implementation and unit tests | TBD | UPI design document |
| vMotion implementation and unit tests | TBD | SPLAT-2801 |
| E2E functional lab provisioning | TBD | Lab request |
| IPI E2E functional tests | TBD | Lab availability |
| UPI E2E functional tests | TBD | UPI implementation + lab |
| vMotion E2E functional tests | TBD | vMotion implementation + lab |
| Polarion test plan | TBD | All test cases finalized |
| Test execution report | TBD | All tests executed |

> **Note**: Dates are TBD and dependency-based. No dates are invented. The schedule will be refined when spike results are available.

---

## 15. Approvals

| Role | Name | Date | Signature |
|---|---|---|---|
| Engineering Lead | TBD | | |
| QE Lead | TBD | | |
| Product Owner | TBD | | |

---

## Appendix A: Current Test Coverage Summary

### A.1 Test File to Feature Mapping

| Feature Area | Test File(s) | Test Count (approx) | Coverage Assessment |
|---|---|---|---|
| Reconciler core (singleton, states, status merge) | `vmwarecloudfoundationmigration_controller_test.go` | 9 (Ginkgo Its + testing.T funcs) | Strong -- covers singleton enforcement, state gating, concurrent writer safety |
| Preflight checks | `preflight_test.go` | 10+ table-driven subtests across 5 top-level tests | Strong -- covers feature gate, CSI PV, CSI driver, MHC/autoscaler, operator health, privilege validation, connectivity, storage management |
| Destination initialization | `destination_initialized_test.go` | 4 subtests (2 top-level tests) | Good -- covers happy path, idempotency, concurrency; uses govmomi simulator |
| Workload migration rollout | `workload_migration_rollout_test.go` | 15+ subtests across 5 top-level tests | Strong -- covers CPMS rollout, scale-down, stall detection, debouncing, message bounding, logging, sorting |
| Ready gate | `ready_test.go` | 7 top-level tests | Strong -- covers stability counter, pool convergence, operator health, counter reset, rapid observation filtering, error recovery |
| OpenShift helpers | `machines_test.go`, `infrastructure_test.go`, `secrets_test.go`, `configmaps_test.go`, `pods_test.go`, `operators_test.go`, `machineconfigpools_test.go`, `version_test.go` | Multiple per file | Good -- unit tests for all manager classes |
| vSphere operations | `session_test.go`, `folder_test.go`, `tags_test.go` | Multiple per file | Good -- uses govmomi simulator |
| Metadata generation | `metadata_test.go` | Multiple | Good |
| E2E smoke | `test/e2e/e2e_test.go` | 2 | Basic -- verifies deployment and metrics only |

### A.2 Notable Coverage Gaps in Current Tests

1. **No Paused state test**: The Paused -> Running -> Paused transition is not tested.
2. **No multi-failure-domain worker distribution test**: Creating MachineSets across multiple failure domains with correct replica distribution is tested only at the helper level.
3. **No ensureMultiSiteConfigured integration test**: Multi-site configuration is covered indirectly through the reconciler flow but has no dedicated test with govmomi simulator.
4. **No ensureSourceCleaned integration test**: Source cleanup is covered indirectly but has no dedicated test.
5. **No E2E functional test**: The E2E suite only tests manager deployment, not actual migration flow.

### A.3 UPI and vMotion Coverage: None

There is **zero existing test coverage** for UPI migration or cross-vCenter vMotion. The current codebase contains no:

- UPI detection logic
- VirtualMachine.Relocate calls
- ServiceLocator construction
- Cold-move (power off / relocate / power on) sequences
- vMotion concurrency limiting
- providerID preservation verification

All test cases in Sections 10.9--10.12 are entirely new work.

---

## Appendix B: Condition Pipeline Reference

The reconciler processes conditions in this fixed order (from `conditionOrder` in `vmwarecloudfoundationmigration_controller.go`):

```
1. InfrastructurePrepared  -- preflight checks, migration path selection
2. DestinationInitialized  -- target vCenter folders, tags
3. MultiSiteConfigured     -- cluster recognizes both vCenters
4. WorkloadMigrated        -- compute running on target (MachineSets, CPMS)
5. SourceCleaned           -- old vCenter fully detached
6. Ready                   -- aggregate: all operators healthy, pools converged
```

The `Accepted` condition is set independently for non-singleton resources.

### Status Conditions by State

| Spec State | Reconciler Behavior |
|---|---|
| `Pending` | Returns immediately; no conditions processed |
| `Running` | Processes conditions in order; first non-True condition is handled |
| `Paused` | Returns immediately; no conditions processed |

---

## Appendix C: API Reference

### Spec Fields

| Field | Type | Description |
|---|---|---|
| `state` | `MigrationState` (Pending/Running/Paused) | Controls workflow; default Pending |
| `targetVCenterCredentialsSecret` | `SecretReference` | Secret with `{server}.username` and `{server}.password` keys |
| `failureDomains` | `[]VSpherePlatformFailureDomainSpec` | Target failure domains (min 1); each requires Name, Region, Zone, Server, Topology |

### Status Fields

| Field | Type | Description |
|---|---|---|
| `conditions` | `[]metav1.Condition` | Map by type; see Appendix B for types |
| `startTime` | `*metav1.Time` | Set on first Running reconcile |
| `completionTime` | `*metav1.Time` | Set when Ready=True |

### Condition Reasons

| Reason | Meaning |
|---|---|
| `Progressing` | Work in progress |
| `Completed` | Condition satisfied |
| `Failed` | Condition failed (error in message) |
| `Paused` | Migration paused by user |
| `Pending` | Migration not started |
| `UnsupportedName` | CR name is not the singleton name |
