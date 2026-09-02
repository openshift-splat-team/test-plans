# Test Plan: SPLAT-2925 — Support EC2 Dedicated Hosts on AWS

**Plan Status:** Draft
**Feature Epic:** [SPLAT-2925](https://issues.redhat.com/browse/SPLAT-2925) — [GA] Support EC2 Dedicated Hosts on AWS
**Parent Feature:** [OCPSTRAT-3236](https://issues.redhat.com/browse/OCPSTRAT-3236)
**Document Version:** 1.0
**Date:** 2026-09-02

---

## Overview

- [1. Test Plan Identifier](#1-test-plan-identifier)
- [2. References](#2-references)
- [3. Introduction](#3-introduction)
- [4. Test Items](#4-test-items)
- [5. Features to Be Tested](#5-features-to-be-tested)
- [6. Features Not to Be Tested](#6-features-not-to-be-tested)
- [7. Approach](#7-approach)
- [8. Item Pass/Fail Criteria](#8-item-passfail-criteria)
- [9. Suspension and Resumption Criteria](#9-suspension-and-resumption-criteria)
- [10. Test Deliverables](#10-test-deliverables)
- [11. Testing Tasks](#11-testing-tasks)
- [12. Environmental Needs](#12-environmental-needs)
- [13. Responsibilities](#13-responsibilities)
- [14. Staffing and Training Needs](#14-staffing-and-training-needs)
- [15. Schedule](#15-schedule)
- [16. Risks and Contingencies](#16-risks-and-contingencies)
- [17. Approvals](#17-approvals)
- [18. Test Scenarios](#18-test-scenarios)
- [19. Traceability Matrix](#19-traceability-matrix)

---

## 1. Test Plan Identifier

| Field | Value |
|-------|-------|
| Identifier | SPLAT-2925-TP-001 |
| Feature | [GA] Support EC2 Dedicated Hosts on AWS |
| Epic | SPLAT-2925 |
| Parent | OCPSTRAT-3236 |
| Components | Cluster Infrastructure, Hive, Install |
| Priority | Critical |

---

## 2. References

| Type | Link / Description |
|------|--------------------|
| Feature Epic | [SPLAT-2925](https://issues.redhat.com/browse/SPLAT-2925) |
| Strategy Feature | [OCPSTRAT-3236](https://issues.redhat.com/browse/OCPSTRAT-3236) |
| Enhancement PR | [openshift/enhancements#1781](https://github.com/openshift/enhancements/pull/1781) |
| Feature Gate Definition | `openshift/api` — `features/features.go`: `FeatureGateAWSDedicatedHosts` |
| Installer Types | `openshift/installer` — `pkg/types/aws/machinepool.go` |
| Installer Validation | `openshift/installer` — `pkg/types/aws/validation/machinepool.go`, `pkg/asset/installconfig/aws/validation.go` |
| Feature Gate Gating | `openshift/installer` — `pkg/types/aws/validation/featuregates.go` |
| Machine Generation (MAPI) | `openshift/installer` — `pkg/asset/machines/aws/machines.go`, `machinesets.go` |
| Machine Generation (CAPI) | `openshift/installer` — `pkg/asset/machines/aws/awsmachines.go`, `clusterapi_machinesets.go` |
| Dedicated Host Retrieval | `openshift/installer` — `pkg/asset/installconfig/aws/dedicatedhosts.go` |
| IAM Permissions | `openshift/installer` — `pkg/asset/installconfig/aws/permissions.go` |
| BYO-Host Tagging | `openshift/installer` — `pkg/asset/cluster/aws/aws.go` |
| Destroy Logic | `openshift/installer` — `pkg/destroy/aws/ec2helpers.go` |
| Install-Config CRD | `openshift/installer` — `data/data/install.openshift.io_installconfigs.yaml` |
| CAPA Types (vendored) | `sigs.k8s.io/cluster-api-provider-aws/v2/api/v1beta2/awsmachine_types.go` |
| HyperShift API | `openshift/hypershift` — `api/hypershift/v1beta1/aws.go` (`PlacementOptions.Tenancy`) |
| Existing Unit Tests | `pkg/types/aws/validation/machinepool_test.go`, `featuregates_test.go`, `pkg/asset/installconfig/aws/validation_test.go` |
| AWS Documentation | [EC2 Dedicated Hosts](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-hosts-overview.html) |

---

## 3. Introduction

### 3.1 Purpose

This test plan covers the GA-readiness testing of SPLAT-2925: enabling OpenShift deployments on AWS using EC2 Dedicated Hosts for compute (worker) nodes. EC2 Dedicated Hosts are physical servers fully dedicated to a customer, allowing them to use existing per-socket or per-core software licenses (BYOL), meet compliance requirements, and gain visibility and control over instance placement.

### 3.2 Background

The feature was requested by a ROSA customer to support third-party software licensing. The implementation adds a `hostPlacement` field to the installer's `install-config.yaml` compute pool definition, gated behind the `AWSDedicatedHosts` feature gate. The feature currently exists at TechPreview maturity and must be promoted to GA for SPLAT-2925.

### 3.3 Scope Summary

**Confirmed scope:**
- OpenShift deployed on AWS using EC2 Dedicated Hosts for **compute (worker) nodes**
- Self-managed and managed deployment considerations
- Multi-node topology confirmed
- Install-time documentation required
- Potential interoperability work from OCPCLOUD and Hive

**Open questions (sourced from Jira):**
- Control-plane / compact topology support — currently **blocked by validation** (see Section 6)
- Managed deployment integration via Hive — **no AWS Dedicated Host fields found in Hive API** (see Section 6)
- HyperShift host-ID pinning — **partial support only** (see Section 6)

---

## 4. Test Items

The following software items are under test, identified by repository and path from source inspection:

### 4.1 openshift/installer

| Item | Path | Description |
|------|------|-------------|
| `HostPlacement` type | `pkg/types/aws/machinepool.go` | Struct with `Affinity` (enum: `DedicatedHost`, `AnyAvailable`) and `DedicatedHost[]` (each with validated `ID`) |
| Feature gate gating | `pkg/types/aws/validation/featuregates.go` | Requires `AWSDedicatedHosts` gate when `hostPlacement` is set on any compute pool |
| MachinePool validation | `pkg/types/aws/validation/machinepool.go` | Validates affinity, blocks control-plane, requires host IDs |
| Platform validation | `pkg/asset/installconfig/aws/validation.go` | Validates host existence, region, zone, uniqueness, ownership, blocks `defaultMachinePlatform` |
| Dedicated host retrieval | `pkg/asset/installconfig/aws/dedicatedhosts.go` | Uses `ec2:DescribeHosts` to retrieve and verify host metadata |
| IAM permissions | `pkg/asset/installconfig/aws/permissions.go` | `PermissionDedicatedHosts` (`ec2:DescribeHosts`), `PermissionDynamicHostAllocation` (`ec2:ReleaseHosts`) |
| Machine generation (MAPI) | `pkg/asset/machines/aws/machines.go` | Sets `Placement.Tenancy=host`, `Host.Affinity=DedicatedHost`, `AllocationStrategy=UserProvided` |
| Machine generation (CAPI) | `pkg/asset/machines/aws/awsmachines.go` | Sets `Tenancy="host"`, `HostAffinity="host"`, `HostID=<id>` |
| MachineSet generation | `pkg/asset/machines/aws/machinesets.go`, `clusterapi_machinesets.go` | Per-AZ host matching via `DedicatedHost()` helper |
| Worker asset | `pkg/asset/machines/worker.go` | Retrieves dedicated hosts when affinity is `DedicatedHost` |
| BYO-host tagging | `pkg/asset/cluster/aws/aws.go` | Tags user-provided hosts with `kubernetes.io/cluster/<id>: shared` |
| Destroy logic | `pkg/destroy/aws/ec2helpers.go` | `deleteEC2DedicatedHost()` via `ec2:ReleaseHosts` with error handling |
| Install-config CRD | `data/data/install.openshift.io_installconfigs.yaml` | OpenAPI schema with CEL validation rules |

### 4.2 openshift/api

| Item | Path | Description |
|------|------|-------------|
| Feature gate | `features/features.go` | `FeatureGateAWSDedicatedHosts` — currently TechPreview/DevPreview; must be promoted to GA |

### 4.3 CAPA (vendored dependency)

| Item | Path | Description |
|------|------|-------------|
| AWSMachineSpec | `api/v1beta2/awsmachine_types.go` | `Tenancy`, `HostID`, `HostAffinity`, `DynamicHostAllocation` fields |

### 4.4 openshift/hypershift

| Item | Path | Description |
|------|------|-------------|
| PlacementOptions | `api/hypershift/v1beta1/aws.go` | `Tenancy` field with enum `default\|dedicated\|host`; no host-ID pinning |

### 4.5 openshift/hive

| Item | Status | Description |
|------|--------|-------------|
| MachinePool API | **No AWS Dedicated Host fields found** | IBM Cloud dedicated host support exists; AWS support is absent |

---

## 5. Features to Be Tested

### 5.1 Configuration and Validation

- `install-config.yaml` `compute[].platform.aws.hostPlacement` field parsing and schema validation
- `affinity` enum: `DedicatedHost` and `AnyAvailable`
- `dedicatedHost[].id` format validation (regex `^h-[0-9a-f]{17}$`, exactly 19 characters)
- CEL validation rule: `dedicatedHost` required when affinity is `DedicatedHost`, forbidden otherwise
- Host existence verification via `ec2:DescribeHosts`
- Region and zone validation for each dedicated host
- One-host-per-zone constraint
- Duplicate host ID detection
- Host ownership check (no `kubernetes.io/cluster/<other>: owned` tags)
- Rejection of `hostPlacement` in `defaultMachinePlatform`
- Rejection of `hostPlacement` on control-plane pools

### 5.2 Feature Gate

- `AWSDedicatedHosts` feature gate activation and gating behavior
- Feature gate promotion from TechPreview to GA (Default profile)
- Feature gate field-path reporting for `compute[N].platform.aws.hostPlacement`

### 5.3 Machine Generation

- MAPI path: `Placement.Tenancy = host`, `Host.Affinity = DedicatedHost`, `AllocationStrategy = UserProvided`, `DedicatedHost.ID` set
- CAPI path: `Tenancy = "host"`, `HostAffinity = "host"`, `HostID` set
- Per-AZ host-to-zone matching in MachineSet generation
- AWSMachineTemplate generation with correct host fields

### 5.4 IAM Permissions

- `ec2:DescribeHosts` required for user-provided dedicated hosts
- Permission simulation during credential validation

### 5.5 Installation and Lifecycle

- End-to-end installation with pre-allocated dedicated hosts on compute pool
- Worker nodes launched on correct dedicated hosts
- Multi-AZ deployment with one host per zone
- Instance restart affinity behavior (instance returns to same host after stop/start)
- `AnyAvailable` affinity: instances placed on hosts with auto-placement enabled

### 5.6 Tagging and Destroy

- BYO-host tagging with `kubernetes.io/cluster/<clusterID>: shared`
- Destroy flow: dynamically allocated hosts released via `ec2:ReleaseHosts`
- Destroy flow: user-provided (shared-tagged) hosts not released
- Error handling: `InvalidHostID.NotFound` (no-op), `UnauthorizedOperation`/`AccessDenied` (warn and skip)

### 5.7 Documentation

- Install-time documentation for configuring dedicated hosts

---

## 6. Features Not to Be Tested

| Feature | Reason |
|---------|--------|
| **Control-plane dedicated hosts** | Validation in `machinepool.go` explicitly blocks `hostPlacement` on control-plane pools. Code comment: "dedicated hosts are only supported on worker pools." This is a confirmed design restriction, not a bug. Whether this should change for GA is an **open question in the Jira**. |
| **Compact / SNO topologies** | Compact topologies use control-plane nodes as workers. Since dedicated hosts are blocked on control-plane pools, compact/SNO is not supported. |
| **Dynamic host allocation via installer** | The CAPA `DynamicHostAllocation` field (which calls `ec2:AllocateHosts`) exists in the vendored CAPA types but is **not exposed in the installer's `install-config.yaml` schema**. The installer only supports user-provided (pre-allocated) hosts. The `PermissionDynamicHostAllocation` (`ec2:ReleaseHosts`) permission group exists solely for destroy-time cleanup of hosts that CAPA may have dynamically allocated. **Dynamic allocation is conditional/deferred and is not a GA acceptance criterion unless explicitly scoped in.** |
| **Hive / managed deployment integration** | Source inspection of `openshift/hive` found **no AWS-specific dedicated host fields** in the MachinePool API (`apis/hive/v1/`) or controller (`pkg/controller/machinepool/`). IBM Cloud dedicated host support exists, but AWS support is absent. This is an **open integration gap** for ROSA/managed deployments. Hive integration is listed as potential work from OCPCLOUD in the Jira but cannot be tested until implementation exists. |
| **HyperShift host-ID pinning** | HyperShift's `PlacementOptions.Tenancy` supports `host` (via CAPA), but there is **no installer-level `hostPlacement` struct, `hostID` field, or dedicated-host validation** on the HyperShift NodePool API. HyperShift relies on CAPA's underlying `HostID`/`DynamicHostAllocation` fields, which are not surfaced through the NodePool CRD. This is **partial support only** — an open question for GA scope. |
| **Edge compute pools** | Edge pools (`MachinePoolEdgeRoleName`) with dedicated hosts have no specific test coverage or known requirement. N/A unless scoped in. |
| **FIPS mode** | No dedicated-host-specific FIPS behavior identified. Standard FIPS testing applies at the platform level. TBD whether dedicated-host-specific FIPS validation is needed. |
| **Disconnected / proxy installs** | No dedicated-host-specific disconnected/proxy behavior identified. The `ec2:DescribeHosts` API call requires internet/VPC endpoint access, but this is standard AWS API connectivity. TBD. |

---

## 7. Approach

### 7.1 Test Strategy

Testing is organized in three tiers:

1. **Unit tests** (existing + additions): Validate configuration parsing, validation rules, feature gate gating, machine generation logic, and permission assembly. These run without AWS credentials in the installer's CI.

2. **Integration / e2e tests**: End-to-end installation on AWS with pre-allocated dedicated hosts, verifying instances land on the correct hosts. These require AWS credentials and pre-provisioned dedicated hosts.

3. **Manual / exploratory tests**: Lifecycle scenarios (stop/start affinity, destroy behavior, upgrade/rollback), negative paths, and documentation review.

### 7.2 Existing Test Coverage

Source inspection confirmed the following unit tests already exist:

| File | Coverage |
|------|----------|
| `pkg/types/aws/validation/machinepool_test.go` | `hostPlacement` on compute (valid/invalid), control-plane blocked, affinity variants, host ID requirements |
| `pkg/types/aws/validation/featuregates_test.go` | Feature gate detection for dedicated hosts across single and multiple compute pools, field path validation |
| `pkg/asset/installconfig/aws/validation_test.go` | Host not found, wrong region, wrong zone, duplicates, same-zone conflict, `defaultMachinePlatform` forbidden, host owned by another cluster |

### 7.3 Test Gap Analysis

| Gap | Priority |
|-----|----------|
| No e2e test for end-to-end install with dedicated hosts at GA quality | High |
| No test for `AnyAvailable` affinity behavior | Medium |
| No test for destroy flow with dedicated-host cleanup | Medium |
| No test for feature gate promotion (TechPreview → GA) | High |
| No test for MAPI vs CAPI machine generation parity | Medium |
| No upgrade/rollback test | Medium |
| No multi-AZ e2e test | Medium |

---

## 8. Item Pass/Fail Criteria

### 8.1 Pass Criteria

- All unit tests in `pkg/types/aws/validation/` and `pkg/asset/installconfig/aws/` pass
- Feature gate `AWSDedicatedHosts` is promoted to the Default (GA) profile in `openshift/api`
- End-to-end installation with pre-allocated dedicated host completes successfully
- Worker nodes are confirmed running on the specified dedicated host(s) via `ec2:DescribeInstances`
- `openshift-install destroy cluster` completes without error, shared-tagged hosts are not released
- All negative-path validation tests reject invalid configurations with correct error messages
- Install-time documentation is published and accurate

### 8.2 Fail Criteria

- Any unit test failure in dedicated-host-related test files
- Feature gate not promoted to GA profile
- Worker nodes not placed on specified dedicated hosts
- Destroy flow releases user-provided (shared-tagged) hosts
- Validation accepts invalid configurations (wrong region, duplicate hosts, control-plane placement)
- Missing or inaccurate install-time documentation

---

## 9. Suspension and Resumption Criteria

### 9.1 Suspension Criteria

- Feature gate `AWSDedicatedHosts` not promoted to GA in `openshift/api` — blocks all GA-level testing
- Pre-allocated dedicated hosts unavailable in CI AWS account — blocks e2e tests
- Critical bug in `ec2:DescribeHosts` integration — blocks validation tests
- CAPA vendored dependency incompatible with host placement fields — blocks machine generation

### 9.2 Resumption Criteria

- Blocker resolved and verified in a development build
- Dedicated hosts provisioned in CI account
- Updated CAPA dependency vendored into installer

---

## 10. Test Deliverables

| Deliverable | Description |
|-------------|-------------|
| This test plan | SPLAT-2925-TP-001, attached to SPLAT-2925 |
| Unit test additions | PRs to `openshift/installer` for any gaps identified in Section 7.3 |
| E2e test PR | PR to `openshift/installer` or `openshift/release` for dedicated-host e2e scenario |
| Test execution report | Summary of test results with pass/fail counts |
| Bug reports | Jira tickets for any defects found |

---

## 11. Testing Tasks

| ID | Task | Dependency | Status |
|----|------|------------|--------|
| T1 | Verify feature gate promotion PR in `openshift/api` | None | TBD |
| T2 | Review and augment existing unit tests for completeness | T1 | TBD |
| T3 | Provision dedicated hosts in CI AWS account | None | TBD |
| T4 | Develop e2e test scenario for single-AZ install | T1, T3 | TBD |
| T5 | Develop e2e test scenario for multi-AZ install | T4 | TBD |
| T6 | Test destroy flow with shared-tagged hosts | T4 | TBD |
| T7 | Test `AnyAvailable` affinity path | T3 | TBD |
| T8 | Test upgrade from TechPreview to GA-enabled cluster | T1 | TBD |
| T9 | Review and validate install-time documentation | T1 | TBD |
| T10 | Execute negative-path validation tests | T2 | TBD |
| T11 | Verify MAPI and CAPI machine generation parity | T4 | TBD |

---

## 12. Environmental Needs

### 12.1 Target Environments

| Dimension | Confirmed Value | Notes |
|-----------|----------------|-------|
| Cloud Provider | AWS | Required |
| OCP Version | TBD | GA target version to be confirmed |
| Topology | Multi-node | Confirmed in Jira scope |
| Topology | Compact / SNO | **Not supported** — control-plane pool blocked |
| Deployment Model | Self-managed (IPI) | Confirmed |
| Deployment Model | Managed (ROSA) | Intended but **Hive integration absent** — TBD |
| Deployment Model | HyperShift | **Partial** — tenancy only, no host-ID pinning — TBD |
| AWS Region | TBD | Must match dedicated host availability |
| Instance Family | TBD | Must be supported by dedicated host type |
| Architecture | amd64 | Primary; arm64 TBD pending dedicated host type availability |
| FIPS | TBD | No dedicated-host-specific FIPS behavior identified |
| Disconnected | TBD | Requires VPC endpoint for `ec2:DescribeHosts` API |
| Proxy | TBD | No dedicated-host-specific proxy behavior identified |
| Machine Management | MAPI (default) | Confirmed path in source |
| Machine Management | CAPI (feature-gated) | Confirmed path in source; requires `ClusterAPIMachineManagementAWS` gate |

### 12.2 Infrastructure Requirements

| Requirement | Description |
|-------------|-------------|
| Dedicated Hosts | At least 2 pre-allocated EC2 Dedicated Hosts in different AZs within a single region |
| Instance Type | Must match the dedicated host's supported instance family (e.g., `m5.xlarge` on `m5` host) |
| IAM Permissions | `ec2:DescribeHosts` for validation; `ec2:ReleaseHosts` for destroy cleanup; standard installer permissions |
| VPC / Subnets | Standard IPI networking; dedicated hosts must be in AZs matching the cluster's subnets |

---

## 13. Responsibilities

| Role | Responsibility |
|------|----------------|
| SPLAT team | Feature implementation, unit tests, feature gate promotion |
| QE / Test Author | Test plan, e2e test development, test execution |
| OCPCLOUD team | Hive integration (if scoped for GA) |
| HyperShift team | Host-ID pinning support (if scoped for GA) |
| Documentation team | Install-time documentation |
| CI/Infrastructure | Dedicated host provisioning in CI account |

---

## 14. Staffing and Training Needs

| Need | Description |
|------|-------------|
| AWS Dedicated Host expertise | Testers must understand EC2 Dedicated Host concepts: allocation, auto-placement, host affinity, instance families, licensing implications |
| Installer codebase familiarity | Understanding of the asset DAG, feature gate system, MAPI vs CAPI paths |
| CAPA familiarity | Understanding of CAPA `AWSMachineSpec` fields for dedicated host placement |

---

## 15. Schedule

| Milestone | Target Date | Notes |
|-----------|-------------|-------|
| Feature gate promotion PR | TBD | Prerequisite for all GA testing |
| Test plan approval | TBD | This document |
| Unit test gap closure | TBD | After feature gate promotion |
| E2e test development | TBD | After CI dedicated hosts provisioned |
| Test execution complete | TBD | Before GA release cut |

---

## 16. Risks and Contingencies

| ID | Risk | Impact | Likelihood | Mitigation |
|----|------|--------|------------|------------|
| R1 | Feature gate `AWSDedicatedHosts` not promoted to GA in time | Blocks all GA testing | Medium | Track promotion PR; escalate early |
| R2 | No dedicated hosts available in CI AWS account | Blocks e2e tests | Medium | Pre-provision hosts; budget for dedicated host costs |
| R3 | Hive integration not implemented for GA | Managed (ROSA) deployment path untestable | High | Clearly mark as out-of-scope / post-GA if Hive work is deferred |
| R4 | HyperShift host-ID pinning not implemented | HyperShift dedicated host testing limited to `tenancy: host` | High | Test tenancy-only path; document gap |
| R5 | Dynamic host allocation scope ambiguity | Test plan may under- or over-cover | Medium | Confirm with feature owner that dynamic allocation is deferred |
| R6 | Control-plane / compact support decision delayed | May require test plan revision | Low | Current plan assumes compute-only; revise if scope changes |
| R7 | Instance family mismatch between host type and instance type | E2e test failures | Low | Verify host type supports chosen instance type before test |
| R8 | CAPA vendored dependency update changes host fields | Machine generation tests may break | Low | Pin CAPA version; revalidate after bump |

---

## 17. Approvals

| Role | Name | Date | Status |
|------|------|------|--------|
| Feature Owner | TBD | | Pending |
| QE Lead | TBD | | Pending |
| Engineering Manager | TBD | | Pending |

---

## 18. Test Scenarios

### 18.1 Configuration and Validation

#### CV-01: Valid Dedicated Host Placement on Compute Pool

| Field | Value |
|-------|-------|
| **Objective** | Verify that a valid `hostPlacement` configuration with affinity `DedicatedHost` and a correctly formatted host ID is accepted |
| **Preconditions** | Pre-allocated dedicated host `h-1234567890abcdef0` in `us-east-1a`; feature gate `AWSDedicatedHosts` enabled |
| **Steps** | 1. Create `install-config.yaml` with `compute[0].platform.aws.hostPlacement.affinity: DedicatedHost` and `dedicatedHost[0].id: h-1234567890abcdef0` with zone `us-east-1a` in pool zones. 2. Run `openshift-install create manifests`. 3. Verify no validation errors. |
| **Expected Result** | Configuration accepted; manifests generated successfully |
| **Traceability** | SPLAT-2925 |

#### CV-02: Valid AnyAvailable Affinity

| Field | Value |
|-------|-------|
| **Objective** | Verify that `affinity: AnyAvailable` is accepted without `dedicatedHost` entries |
| **Preconditions** | Feature gate `AWSDedicatedHosts` enabled |
| **Steps** | 1. Create `install-config.yaml` with `compute[0].platform.aws.hostPlacement.affinity: AnyAvailable`. 2. Verify no `dedicatedHost` field is present. 3. Run `openshift-install create manifests`. |
| **Expected Result** | Configuration accepted; no dedicated host fields set on generated machines |
| **Traceability** | SPLAT-2925 |

#### CV-03: Invalid Host ID Format

| Field | Value |
|-------|-------|
| **Objective** | Verify that a host ID not matching `^h-[0-9a-f]{17}$` is rejected |
| **Preconditions** | Feature gate enabled |
| **Steps** | 1. Set `dedicatedHost[0].id: invalid-host-id`. 2. Run installer validation. |
| **Expected Result** | CEL validation error: "hostID must start with 'h-' followed by 17 lowercase hexadecimal characters" |
| **Traceability** | SPLAT-2925 |

#### CV-04: Host ID Too Short / Too Long

| Field | Value |
|-------|-------|
| **Objective** | Verify that host IDs not exactly 19 characters are rejected |
| **Preconditions** | Feature gate enabled |
| **Steps** | 1. Set `dedicatedHost[0].id: h-12345` (too short). 2. Run installer validation. 3. Set `dedicatedHost[0].id: h-1234567890abcdef01` (too long). 4. Run installer validation. |
| **Expected Result** | Schema validation errors for `minLength: 19` and `maxLength: 19` |
| **Traceability** | SPLAT-2925 |

#### CV-05: DedicatedHost Required with Affinity DedicatedHost

| Field | Value |
|-------|-------|
| **Objective** | Verify that `dedicatedHost` is required when affinity is `DedicatedHost` |
| **Preconditions** | Feature gate enabled |
| **Steps** | 1. Set `hostPlacement.affinity: DedicatedHost` without `dedicatedHost` field. 2. Run installer validation. |
| **Expected Result** | CEL/validation error: "dedicatedHost is required when affinity is DedicatedHost, and forbidden otherwise" |
| **Traceability** | SPLAT-2925 |

#### CV-06: DedicatedHost Forbidden with Affinity AnyAvailable

| Field | Value |
|-------|-------|
| **Objective** | Verify that `dedicatedHost` is forbidden when affinity is `AnyAvailable` |
| **Preconditions** | Feature gate enabled |
| **Steps** | 1. Set `hostPlacement.affinity: AnyAvailable` with `dedicatedHost[0].id: h-1234567890abcdef0`. 2. Run installer validation. |
| **Expected Result** | CEL/validation error: "dedicatedHost is required when affinity is DedicatedHost, and forbidden otherwise" |
| **Traceability** | SPLAT-2925 |

#### CV-07: Host Not Found in AWS

| Field | Value |
|-------|-------|
| **Objective** | Verify that a host ID not found via `ec2:DescribeHosts` is rejected |
| **Preconditions** | Feature gate enabled; host `h-aaaaaaaaaaaaaaaaa` does not exist in AWS account |
| **Steps** | 1. Set `dedicatedHost[0].id: h-aaaaaaaaaaaaaaaaa`. 2. Run installer validation with AWS credentials. |
| **Expected Result** | Validation error: "dedicated host h-aaaaaaaaaaaaaaaaa not found" |
| **Traceability** | SPLAT-2925 |

#### CV-08: Host in Wrong Region

| Field | Value |
|-------|-------|
| **Objective** | Verify that a host in a different region than the cluster is rejected |
| **Preconditions** | Host `h-1234567890abcdef0` in `us-west-2a`; cluster configured for `us-east-1` |
| **Steps** | 1. Configure cluster in `us-east-1`. 2. Set host ID pointing to a `us-west-2` host. 3. Run installer validation. |
| **Expected Result** | Validation error: "dedicated host ... is in zone us-west-2a which is not in the cluster's region us-east-1" |
| **Traceability** | SPLAT-2925 |

#### CV-09: Host Zone Not in Pool Zones

| Field | Value |
|-------|-------|
| **Objective** | Verify that a host whose zone is not in the pool's configured zones is rejected |
| **Preconditions** | Pool zones: `[us-east-1a]`; host in `us-east-1b` |
| **Steps** | 1. Configure pool with zone `us-east-1a`. 2. Set host in zone `us-east-1b`. 3. Run validation. |
| **Expected Result** | Validation error: "machine pool specifies zones ... but dedicated host ... is in zone us-east-1b" |
| **Traceability** | SPLAT-2925 |

#### CV-10: Duplicate Host IDs

| Field | Value |
|-------|-------|
| **Objective** | Verify that duplicate host IDs in the same configuration are rejected |
| **Preconditions** | Feature gate enabled |
| **Steps** | 1. Set `dedicatedHost[0].id` and `dedicatedHost[1].id` to the same value. 2. Run validation. |
| **Expected Result** | Validation error: "duplicate dedicated host ... (first seen at index 0)" |
| **Traceability** | SPLAT-2925 |

#### CV-11: Multiple Hosts in Same Zone

| Field | Value |
|-------|-------|
| **Objective** | Verify that multiple hosts in the same AZ are rejected |
| **Preconditions** | Two different hosts both in `us-east-1a` |
| **Steps** | 1. Set two different host IDs that resolve to the same zone. 2. Run validation. |
| **Expected Result** | Validation error: "multiple dedicated hosts configured for zone us-east-1a" |
| **Traceability** | SPLAT-2925 |

#### CV-12: Host Owned by Another Cluster

| Field | Value |
|-------|-------|
| **Objective** | Verify that a host tagged as owned by another cluster is rejected |
| **Preconditions** | Host tagged with `kubernetes.io/cluster/<other-cluster>: owned` |
| **Steps** | 1. Set host ID to a host owned by another cluster. 2. Run validation. |
| **Expected Result** | Validation error: "Dedicated host ... is owned by other cluster ... and cannot be used for new installations" |
| **Traceability** | SPLAT-2925 |

#### CV-13: HostPlacement in defaultMachinePlatform Rejected

| Field | Value |
|-------|-------|
| **Objective** | Verify that `hostPlacement` in `platform.aws.defaultMachinePlatform` is rejected |
| **Preconditions** | Feature gate enabled |
| **Steps** | 1. Set `platform.aws.defaultMachinePlatform.hostPlacement` with a valid configuration. 2. Run validation. |
| **Expected Result** | Validation error: "dedicated hosts cannot be configured in defaultMachinePlatform, they must be specified per machine pool" |
| **Traceability** | SPLAT-2925 |

#### CV-14: HostPlacement on Control-Plane Pool Rejected

| Field | Value |
|-------|-------|
| **Objective** | Verify that `hostPlacement` on a control-plane pool is rejected |
| **Preconditions** | Feature gate enabled |
| **Steps** | 1. Set `controlPlane.platform.aws.hostPlacement` with a valid configuration. 2. Run validation. |
| **Expected Result** | Validation error: "dedicated hosts are not supported on control plane pools" |
| **Traceability** | SPLAT-2925 |

### 18.2 Feature Gate

#### FG-01: Feature Gate Required When HostPlacement Set

| Field | Value |
|-------|-------|
| **Objective** | Verify that the `AWSDedicatedHosts` feature gate is required when `hostPlacement` is configured |
| **Preconditions** | Feature gate **not** enabled (Default profile without `AWSDedicatedHosts`) |
| **Steps** | 1. Configure `hostPlacement` on a compute pool. 2. Run installer. |
| **Expected Result** | Installer rejects configuration because the `AWSDedicatedHosts` feature gate is not enabled |
| **Traceability** | SPLAT-2925 |

#### FG-02: Feature Gate Not Required When HostPlacement Absent

| Field | Value |
|-------|-------|
| **Objective** | Verify that the `AWSDedicatedHosts` feature gate is not triggered when no `hostPlacement` is configured |
| **Preconditions** | Standard install-config without `hostPlacement` |
| **Steps** | 1. Run `GatedFeatures()` on the install config. 2. Check that `AWSDedicatedHosts` is not in the returned list. |
| **Expected Result** | No `AWSDedicatedHosts` feature gate requirement |
| **Traceability** | SPLAT-2925 |

#### FG-03: Feature Gate Promotion to GA

| Field | Value |
|-------|-------|
| **Objective** | Verify that the `AWSDedicatedHosts` feature gate is enabled in the Default (GA) profile after promotion |
| **Preconditions** | Promotion PR merged in `openshift/api` |
| **Steps** | 1. Check feature gate definition in `features/features.go`. 2. Verify it includes `.enable(inDefault())` or equivalent GA enablement. 3. Confirm installer vendoring picks up the change. |
| **Expected Result** | Feature gate enabled by default; `hostPlacement` accepted without explicit TechPreview feature set |
| **Traceability** | SPLAT-2925 |

### 18.3 Machine Generation

#### MG-01: MAPI Machine Spec with Dedicated Host

| Field | Value |
|-------|-------|
| **Objective** | Verify that MAPI machine generation produces correct placement fields for dedicated hosts |
| **Preconditions** | Valid `hostPlacement` with `DedicatedHost` affinity and host ID |
| **Steps** | 1. Generate machines via MAPI path. 2. Inspect `AWSMachineProviderConfig`. |
| **Expected Result** | `Placement.Tenancy = host`, `Placement.Host.Affinity = DedicatedHost`, `Placement.Host.DedicatedHost.AllocationStrategy = UserProvided`, `DedicatedHost.ID = <hostID>` |
| **Traceability** | SPLAT-2925 |

#### MG-02: CAPI Machine Spec with Dedicated Host

| Field | Value |
|-------|-------|
| **Objective** | Verify that CAPI machine generation produces correct placement fields for dedicated hosts |
| **Preconditions** | Valid `hostPlacement` with `DedicatedHost` affinity and host ID |
| **Steps** | 1. Generate AWSMachine/AWSMachineTemplate via CAPI path. 2. Inspect spec. |
| **Expected Result** | `spec.Tenancy = "host"`, `spec.HostAffinity = "host"`, `spec.HostID = <hostID>` |
| **Traceability** | SPLAT-2925 |

#### MG-03: MAPI/CAPI Parity

| Field | Value |
|-------|-------|
| **Objective** | Verify that MAPI and CAPI paths produce functionally equivalent dedicated host placement |
| **Preconditions** | Same `hostPlacement` configuration |
| **Steps** | 1. Generate machines via both paths. 2. Compare placement semantics. |
| **Expected Result** | Both paths target the same dedicated host with equivalent tenancy and affinity settings |
| **Traceability** | SPLAT-2925 |

#### MG-04: Per-AZ Host Matching in MachineSet

| Field | Value |
|-------|-------|
| **Objective** | Verify that each MachineSet targets the correct dedicated host for its AZ |
| **Preconditions** | Two hosts in different AZs (e.g., `us-east-1a` and `us-east-1b`) |
| **Steps** | 1. Generate MachineSets for a multi-AZ pool. 2. Verify each MachineSet's host ID matches the host in its AZ. |
| **Expected Result** | MachineSet for `us-east-1a` references host in `us-east-1a`; MachineSet for `us-east-1b` references host in `us-east-1b` |
| **Traceability** | SPLAT-2925 |

#### MG-05: No Host Fields When AnyAvailable

| Field | Value |
|-------|-------|
| **Objective** | Verify that `AnyAvailable` affinity does not set host-specific fields on machines |
| **Preconditions** | `hostPlacement.affinity: AnyAvailable` configured |
| **Steps** | 1. Generate machines. 2. Inspect placement fields. |
| **Expected Result** | No `HostID`, `Tenancy`, or `HostAffinity` fields set on machine spec (instances rely on AWS auto-placement) |
| **Traceability** | SPLAT-2925 |

### 18.4 Installation and Lifecycle

#### IL-01: End-to-End Install with Pre-Allocated Host (Single AZ)

| Field | Value |
|-------|-------|
| **Objective** | Verify that a full IPI installation places worker nodes on the specified dedicated host |
| **Preconditions** | One pre-allocated dedicated host in a single AZ; feature gate enabled |
| **Steps** | 1. Create `install-config.yaml` with `hostPlacement` targeting the host. 2. Run `openshift-install create cluster`. 3. After cluster is ready, verify worker node instances via `aws ec2 describe-instances --filters "Name=host-id,Values=<hostID>"`. |
| **Expected Result** | Worker instances are running on the specified dedicated host; cluster is healthy |
| **Traceability** | SPLAT-2925 |

#### IL-02: End-to-End Install with Pre-Allocated Hosts (Multi-AZ)

| Field | Value |
|-------|-------|
| **Objective** | Verify multi-AZ installation with one dedicated host per AZ |
| **Preconditions** | Two or more pre-allocated dedicated hosts in different AZs |
| **Steps** | 1. Configure `hostPlacement` with hosts in each AZ. 2. Install cluster. 3. Verify each worker is on the correct host for its AZ. |
| **Expected Result** | Workers distributed across AZs, each on the correct dedicated host |
| **Traceability** | SPLAT-2925 |

#### IL-03: Instance Restart Affinity

| Field | Value |
|-------|-------|
| **Objective** | Verify that a stopped and restarted instance returns to the same dedicated host |
| **Preconditions** | Cluster installed with `DedicatedHost` affinity on a pre-allocated host |
| **Steps** | 1. Identify a worker instance on the dedicated host. 2. Stop the instance via AWS API. 3. Start the instance. 4. Verify the instance is on the same host. |
| **Expected Result** | Instance returns to the same dedicated host after stop/start |
| **Traceability** | SPLAT-2925 |

#### IL-04: AnyAvailable Placement Behavior

| Field | Value |
|-------|-------|
| **Objective** | Verify that `AnyAvailable` affinity places instances on hosts with auto-placement enabled |
| **Preconditions** | Dedicated host with auto-placement enabled in the cluster's AZ; feature gate enabled |
| **Steps** | 1. Configure `hostPlacement.affinity: AnyAvailable`. 2. Install cluster. 3. Verify worker placement. |
| **Expected Result** | Workers placed on available dedicated hosts (AWS auto-placement behavior) |
| **Traceability** | SPLAT-2925 |

### 18.5 IAM Permissions

#### IAM-01: ec2:DescribeHosts Permission Required

| Field | Value |
|-------|-------|
| **Objective** | Verify that `ec2:DescribeHosts` is included in required permissions when dedicated hosts are configured |
| **Preconditions** | `hostPlacement` configured on a compute pool |
| **Steps** | 1. Call `RequiredPermissionGroups()`. 2. Verify `PermissionDedicatedHosts` is in the returned list. 3. Verify `ec2:DescribeHosts` is in the expanded permissions. |
| **Expected Result** | `ec2:DescribeHosts` included in required permissions |
| **Traceability** | SPLAT-2925 |

#### IAM-02: Permission Simulation Failure

| Field | Value |
|-------|-------|
| **Objective** | Verify that credential validation fails when `ec2:DescribeHosts` is not granted |
| **Preconditions** | IAM credentials without `ec2:DescribeHosts`; `hostPlacement` configured |
| **Steps** | 1. Run installer with restricted credentials. |
| **Expected Result** | Credential validation fails with insufficient permissions error |
| **Traceability** | SPLAT-2925 |

### 18.6 Tagging and Destroy

#### TD-01: BYO-Host Tagging

| Field | Value |
|-------|-------|
| **Objective** | Verify that user-provided dedicated hosts are tagged with `kubernetes.io/cluster/<clusterID>: shared` |
| **Preconditions** | Cluster installed with pre-allocated dedicated hosts |
| **Steps** | 1. After installation, describe the dedicated host via AWS API. 2. Check tags. |
| **Expected Result** | Host has tag `kubernetes.io/cluster/<clusterID>: shared` |
| **Traceability** | SPLAT-2925 |

#### TD-02: Shared-Tagged Host Not Released on Destroy

| Field | Value |
|-------|-------|
| **Objective** | Verify that `openshift-install destroy cluster` does not release user-provided (shared-tagged) hosts |
| **Preconditions** | Cluster installed with pre-allocated dedicated hosts, hosts tagged as `shared` |
| **Steps** | 1. Run `openshift-install destroy cluster`. 2. Verify the dedicated host still exists and is available. |
| **Expected Result** | Dedicated host is not released; host remains available in AWS account |
| **Traceability** | SPLAT-2925 |

#### TD-03: Dynamically Allocated Host Released on Destroy

| Field | Value |
|-------|-------|
| **Objective** | Verify that dynamically allocated hosts (by CAPA) are released during destroy |
| **Preconditions** | Cluster with a CAPA-allocated dedicated host (tagged as `owned`) |
| **Steps** | 1. Run `openshift-install destroy cluster`. 2. Verify destroy attempts `ec2:ReleaseHosts`. |
| **Expected Result** | Dynamically allocated host is released; `InvalidHostID.NotFound` treated as success |
| **Traceability** | SPLAT-2925 — **Note: dynamic allocation is conditional/deferred; this scenario applies only if CAPA dynamically allocates hosts during cluster lifecycle** |

#### TD-04: Destroy Without ReleaseHosts Permission

| Field | Value |
|-------|-------|
| **Objective** | Verify that destroy handles `UnauthorizedOperation`/`AccessDenied` gracefully when `ec2:ReleaseHosts` is not granted |
| **Preconditions** | IAM credentials without `ec2:ReleaseHosts` |
| **Steps** | 1. Run `openshift-install destroy cluster`. |
| **Expected Result** | Warning logged: "User does not have permission to release dedicated hosts, skipping"; destroy continues |
| **Traceability** | SPLAT-2925 |

### 18.7 Upgrade and Rollback

#### UR-01: Upgrade from TechPreview to GA-Enabled Cluster

| Field | Value |
|-------|-------|
| **Objective** | Verify that a cluster installed with the TechPreview `AWSDedicatedHosts` gate can upgrade to a GA version where the gate is in Default |
| **Preconditions** | Cluster installed with TechPreview feature set and dedicated hosts |
| **Steps** | 1. Install cluster with `featureSet: TechPreviewNoUpgrade` and dedicated host placement. 2. Upgrade to GA OCP version. 3. Verify dedicated host placement is preserved. |
| **Expected Result** | Upgrade succeeds; worker nodes remain on dedicated hosts; MachineSet/AWSMachineTemplate specs unchanged |
| **Traceability** | SPLAT-2925 |

#### UR-02: Rollback Behavior

| Field | Value |
|-------|-------|
| **Objective** | Verify that rollback from a GA version to a version without `AWSDedicatedHosts` does not disrupt running workloads |
| **Preconditions** | Cluster running on dedicated hosts at GA version |
| **Steps** | 1. Attempt downgrade (if supported). 2. Verify worker nodes remain on dedicated hosts. |
| **Expected Result** | Running workloads are not disrupted; MachineSets retain dedicated host configuration |
| **Traceability** | SPLAT-2925 |

### 18.8 Managed / Hive Integration (Open Question)

#### HV-01: Hive MachinePool with Dedicated Host

| Field | Value |
|-------|-------|
| **Objective** | Verify that Hive's MachinePool API supports dedicated host configuration for AWS |
| **Preconditions** | Hive API with AWS dedicated host fields (currently **absent** — see Section 6) |
| **Steps** | 1. Create a Hive MachinePool with dedicated host placement. 2. Verify the generated MachineSet targets the dedicated host. |
| **Expected Result** | MachineSet with correct dedicated host placement |
| **Traceability** | SPLAT-2925 — **BLOCKED: Hive API has no AWS dedicated host fields. This test is deferred until Hive integration is implemented.** |

### 18.9 HyperShift Integration (Open Question)

#### HS-01: HyperShift NodePool with Tenancy Host

| Field | Value |
|-------|-------|
| **Objective** | Verify that HyperShift NodePool with `tenancy: host` places instances on dedicated hosts |
| **Preconditions** | Dedicated hosts with auto-placement enabled in the NodePool's AZ |
| **Steps** | 1. Create a HyperShift NodePool with `placementOptions.tenancy: host`. 2. Verify instances are placed on dedicated hosts. |
| **Expected Result** | Instances use dedicated host tenancy |
| **Traceability** | SPLAT-2925 — **Note: HyperShift supports `tenancy: host` but does NOT support host-ID pinning. Instances are auto-placed on available hosts.** |

#### HS-02: HyperShift NodePool Host-ID Pinning

| Field | Value |
|-------|-------|
| **Objective** | Verify that HyperShift NodePool can target a specific dedicated host by ID |
| **Preconditions** | HyperShift API with host-ID field (currently **absent** — see Section 6) |
| **Steps** | 1. Create NodePool with specific host ID. 2. Verify instances land on the specified host. |
| **Expected Result** | Instances on specified host |
| **Traceability** | SPLAT-2925 — **BLOCKED: HyperShift NodePool API has no host-ID field. This test is deferred until HyperShift host-ID pinning is implemented.** |

### 18.10 Negative and Error Paths

#### NE-01: Install-Config with Invalid Affinity Value

| Field | Value |
|-------|-------|
| **Objective** | Verify that an unsupported `affinity` value is rejected |
| **Preconditions** | Feature gate enabled |
| **Steps** | 1. Set `hostPlacement.affinity: InvalidValue`. 2. Run validation. |
| **Expected Result** | Schema validation error: affinity must be one of `DedicatedHost` or `AnyAvailable` |
| **Traceability** | SPLAT-2925 |

#### NE-02: Missing Affinity Field

| Field | Value |
|-------|-------|
| **Objective** | Verify that `hostPlacement` without `affinity` is rejected |
| **Preconditions** | Feature gate enabled |
| **Steps** | 1. Set `hostPlacement` with `dedicatedHost` but no `affinity`. 2. Run validation. |
| **Expected Result** | Validation error: "affinity is required when hostPlacement is configured" |
| **Traceability** | SPLAT-2925 |

#### NE-03: Empty Host ID

| Field | Value |
|-------|-------|
| **Objective** | Verify that an empty host ID is rejected |
| **Preconditions** | Feature gate enabled |
| **Steps** | 1. Set `dedicatedHost[0].id: ""`. 2. Run validation. |
| **Expected Result** | Validation error: "a hostID must be specified when configuring 'dedicatedHost'" |
| **Traceability** | SPLAT-2925 |

#### NE-04: HostPlacement on Non-Worker Pool

| Field | Value |
|-------|-------|
| **Objective** | Verify that `hostPlacement` on a named pool that is not "worker" is rejected |
| **Preconditions** | Feature gate enabled |
| **Steps** | 1. Set `hostPlacement` on a pool named "infra" or any non-"worker" name. 2. Run validation. |
| **Expected Result** | Validation error: "dedicated hosts are only supported on worker pools, not on <poolName> pools" |
| **Traceability** | SPLAT-2925 |

### 18.11 Documentation

#### DOC-01: Install-Time Documentation

| Field | Value |
|-------|-------|
| **Objective** | Verify that install-time documentation for dedicated hosts is published and accurate |
| **Preconditions** | GA documentation published |
| **Steps** | 1. Review documentation for accuracy against implemented configuration fields. 2. Verify example `install-config.yaml` snippets. 3. Verify IAM permission requirements are documented. 4. Verify supported topologies (compute-only, multi-AZ) are documented. 5. Verify limitations (no control-plane, no compact/SNO) are documented. |
| **Expected Result** | Documentation is complete, accurate, and consistent with implementation |
| **Traceability** | SPLAT-2925 |

### 18.12 Regression and CI

#### RC-01: Existing Unit Tests Pass

| Field | Value |
|-------|-------|
| **Objective** | Verify all existing dedicated-host unit tests continue to pass |
| **Preconditions** | Code at GA-candidate commit |
| **Steps** | 1. Run `go test ./pkg/types/aws/validation/...`. 2. Run `go test ./pkg/asset/installconfig/aws/...`. |
| **Expected Result** | All tests pass |
| **Traceability** | SPLAT-2925 |

#### RC-02: Standard IPI Install Without Dedicated Hosts

| Field | Value |
|-------|-------|
| **Objective** | Verify that a standard AWS IPI install without `hostPlacement` is unaffected by the feature |
| **Preconditions** | GA build with `AWSDedicatedHosts` feature gate in Default profile |
| **Steps** | 1. Install a standard AWS IPI cluster without `hostPlacement`. 2. Verify no dedicated host fields in machine specs. |
| **Expected Result** | Standard installation behavior unchanged; no regressions |
| **Traceability** | SPLAT-2925 |

#### RC-03: CI Integration

| Field | Value |
|-------|-------|
| **Objective** | Verify that a dedicated-host e2e job is integrated into CI |
| **Preconditions** | CI job configured in `openshift/release`; dedicated hosts provisioned in CI account |
| **Steps** | 1. Trigger CI job. 2. Verify end-to-end test passes. |
| **Expected Result** | CI job passes consistently |
| **Traceability** | SPLAT-2925 |

---

## 19. Traceability Matrix

| Scenario ID | Feature Area | SPLAT-2925 Requirement | Blocked / Deferred? |
|-------------|--------------|------------------------|---------------------|
| CV-01 | Config/Validation | Compute pool dedicated host placement | No |
| CV-02 | Config/Validation | AnyAvailable affinity | No |
| CV-03 | Config/Validation | Host ID format validation | No |
| CV-04 | Config/Validation | Host ID length validation | No |
| CV-05 | Config/Validation | DedicatedHost requires host IDs | No |
| CV-06 | Config/Validation | AnyAvailable forbids host IDs | No |
| CV-07 | Config/Validation | Host existence in AWS | No |
| CV-08 | Config/Validation | Host region validation | No |
| CV-09 | Config/Validation | Host zone-pool alignment | No |
| CV-10 | Config/Validation | Duplicate host ID detection | No |
| CV-11 | Config/Validation | One host per zone constraint | No |
| CV-12 | Config/Validation | Host ownership check | No |
| CV-13 | Config/Validation | defaultMachinePlatform rejection | No |
| CV-14 | Config/Validation | Control-plane pool rejection | No |
| FG-01 | Feature Gate | Gate required when hostPlacement set | No |
| FG-02 | Feature Gate | Gate not required without hostPlacement | No |
| FG-03 | Feature Gate | GA promotion | No |
| MG-01 | Machine Generation | MAPI dedicated host spec | No |
| MG-02 | Machine Generation | CAPI dedicated host spec | No |
| MG-03 | Machine Generation | MAPI/CAPI parity | No |
| MG-04 | Machine Generation | Per-AZ host matching | No |
| MG-05 | Machine Generation | AnyAvailable no host fields | No |
| IL-01 | Installation | Single-AZ e2e install | No |
| IL-02 | Installation | Multi-AZ e2e install | No |
| IL-03 | Installation | Restart affinity | No |
| IL-04 | Installation | AnyAvailable placement | No |
| IAM-01 | IAM | DescribeHosts permission | No |
| IAM-02 | IAM | Permission simulation failure | No |
| TD-01 | Tags/Destroy | BYO-host shared tagging | No |
| TD-02 | Tags/Destroy | Shared host not released | No |
| TD-03 | Tags/Destroy | Dynamic host released | Conditional/Deferred — dynamic allocation not exposed by installer |
| TD-04 | Tags/Destroy | Destroy without ReleaseHosts | No |
| UR-01 | Upgrade | TechPreview to GA upgrade | No |
| UR-02 | Upgrade | Rollback behavior | No |
| HV-01 | Hive/Managed | Hive MachinePool integration | **BLOCKED** — Hive API has no AWS dedicated host fields |
| HS-01 | HyperShift | Tenancy host support | No (partial — tenancy only) |
| HS-02 | HyperShift | Host-ID pinning | **BLOCKED** — HyperShift API has no host-ID field |
| NE-01 | Negative | Invalid affinity value | No |
| NE-02 | Negative | Missing affinity | No |
| NE-03 | Negative | Empty host ID | No |
| NE-04 | Negative | Non-worker pool rejection | No |
| DOC-01 | Documentation | Install-time documentation | No |
| RC-01 | Regression | Existing unit tests | No |
| RC-02 | Regression | Standard install unaffected | No |
| RC-03 | Regression | CI integration | No |

### Dynamic Host Allocation Note

Dynamic host allocation via `ec2:AllocateHosts` is **not exposed by the installer's `install-config.yaml` schema**. The installer only supports user-provided (pre-allocated) hosts. While the vendored CAPA dependency (`sigs.k8s.io/cluster-api-provider-aws/v2`) supports `DynamicHostAllocation` at the `AWSMachineSpec` level, this capability is not surfaced through the installer configuration. The `PermissionDynamicHostAllocation` permission group (`ec2:ReleaseHosts`) exists solely for destroy-time cleanup. Dynamic allocation is **not a GA acceptance criterion** and test scenario TD-03 is marked as conditional/deferred accordingly.

---

*End of Test Plan SPLAT-2925-TP-001*
