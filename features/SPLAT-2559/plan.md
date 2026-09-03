# Test Plan: SPLAT-2559 — Cluster API (CAPI) Support for Nutanix Installations

## 1. Test Plan Identifier

| Field                  | Value                                                                        |
| ------------------------| ------------------------------------------------------------------------------|
| **Plan ID**            | TP-SPLAT-2559-001                                                            |
| **Version**            | 1.0 (Draft)                                                                  |
| **Date Created**       | 2026-09-03                                                                   |
| **Status**             | Draft                                                                        |
| **Jira Epic**          | SPLAT-2559                                                                   |
| **Epic Title**         | [Dev Preview] Introduce Cluster API (CAPI) Support for Nutanix Installations |
| **Epic Status**        | In Progress                                                                  |
| **Epic Created**       | 2025-12-01                                                                   |
| **Epic Last Modified** | 2026-08-19                                                                   |
| **Jira Labels**        | `needs-refinement`                                                           |

### 1.1 References

| Ref ID | Source | Description |
|---|---|---|
| REF-01 | [SPLAT-2559](https://issues.redhat.com/browse/SPLAT-2559) | Parent Jira Epic |
| REF-02 | [openshift/cluster-capi-operator PR #383](https://github.com/openshift/cluster-capi-operator/pull/383) | Nutanix provider scheme registration, platform setup, e2e tests, conversion logic |
| REF-03 | [openshift/cluster-capi-operator PR #395](https://github.com/openshift/cluster-capi-operator/pull/395) | MAPI-to-CAPI conversion for Nutanix machines and MachineSets |
| REF-04 | [openshift/machine-api-operator PR #1425](https://github.com/openshift/machine-api-operator/pull/1425) | Multi-subnet e2e tests for Nutanix |
| REF-05 | [openshift/release PR #70896](https://github.com/openshift/release/pull/70896) | Optional e2e-nutanix-operator-multi-subnet CI jobs |
| REF-06 | openshift/cluster-capi-operator repository | `make unit`, `make e2e`, `make verify` CI targets |
| REF-07 | openshift/machine-api-operator repository | `make test-e2e`, `make test-e2e-tech-preview`, OTE framework |
| REF-08 | openshift/installer repository | Platform-specific unit/integration tests, CAPI build/verification |

---

## 2. Introduction

### 2.1 Purpose

This test plan defines the verification and validation strategy for SPLAT-2559: introducing Cluster API (CAPI) support for Nutanix as a platform in OpenShift Container Platform (OCP). The feature replaces the existing Machine API (MAPI) provider for Nutanix with the upstream Cluster API provider, enabling declarative machine lifecycle management, multi-installation-method support, and an upgrade path from MAPI to CAPI.

### 2.2 Scope

The test plan covers all testable requirements derived from the SPLAT-2559 epic, including:

- New cluster deployment on Nutanix via IPI, Assisted Installer, and Agent-Based Installer (ABI)
- CAPI provider integration (cluster-api-provider-nutanix v1.7.2)
- MAPI-to-CAPI migration/conversion of Machines and MachineSets
- Declarative machine create/scale/destroy lifecycle
- Networking, storage, and machine configuration parity with MAPI
- Prism Central credential management
- Performance parity or improvement relative to MAPI
- Upgrade path validation
- e2e test coverage for IPI, Assisted Installer, and ABI

### 2.3 Known Scope Gaps

The SPLAT-2559 Jira epic has the following unresolved areas that limit test-plan precision:

| Gap | Detail |
|---|---|
| **Questions to Answer** | Listed as "..." in Jira — no documented open questions are available. Test scenarios marked TBD may be affected once questions are refined. |
| **Out of Scope** | Listed as "..." in Jira — explicit exclusions are not yet defined. Section 5 lists assumed exclusions based on epic title and evidence. |
| **Epic Label** | `needs-refinement` indicates requirements are not fully baselined. |
| **No Jira Comments** | Zero comments on the epic; no additional context or decisions are documented there. |
| **Linked Sub-tasks** | Sub-task and story breakdown is not provided in this plan's input; individual acceptance criteria are unknown. |

---

## 3. Test Items

The following software items are in scope for testing:

| Item ID | Component | Repository | Version / Branch | Notes |
|---|---|---|---|---|
| TI-01 | Cluster CAPI Operator — Nutanix provider | openshift/cluster-capi-operator | PR #383 baseline | Scheme registration, platform setup, e2e framework |
| TI-02 | MAPI-to-CAPI Conversion — Nutanix | openshift/cluster-capi-operator | PR #395 baseline | Machine/MachineSet conversion logic |
| TI-03 | Machine API Operator — Multi-Subnet | openshift/machine-api-operator | PR #1425 baseline | Multi-subnet e2e, feature-gated |
| TI-04 | CI Job Definitions — Multi-Subnet | openshift/release | PR #70896 baseline | Optional presubmit jobs for release-4.21/4.22 |
| TI-05 | OpenShift Installer — Nutanix CAPI | openshift/installer | TBD | Platform-specific install config, CAPI build |
| TI-06 | Assisted Installer — Nutanix CAPI | TBD | TBD | No PR evidence supplied |
| TI-07 | Agent-Based Installer (ABI) — Nutanix CAPI | TBD | TBD | No PR evidence supplied |
| TI-08 | Cluster API Provider Nutanix (upstream) | cluster-api-provider-nutanix | v1.7.2 | Imported dependency |

---

## 4. Features to Be Tested

### 4.1 Installation Methods

| Feature ID | Feature | Installation Method | Priority |
|---|---|---|---|
| F-INST-01 | IPI cluster creation on Nutanix via CAPI | IPI | High |
| F-INST-02 | Assisted Installer cluster creation on Nutanix via CAPI | Assisted Installer | High |
| F-INST-03 | Agent-Based Installer cluster creation on Nutanix via CAPI | ABI | High |

### 4.2 CAPI Provider Integration

| Feature ID | Feature | Priority |
|---|---|---|
| F-CAPI-01 | Nutanix provider scheme registration in cluster-capi-operator | High |
| F-CAPI-02 | NutanixCluster CR creation with correct infrastructure reference | High |
| F-CAPI-03 | PrismCentral endpoint and failure domain configuration | High |
| F-CAPI-04 | Control-plane endpoint propagation | High |
| F-CAPI-05 | `managed-by` annotation on CAPI resources | Medium |
| F-CAPI-06 | CAPI Cluster readiness status | High |
| F-CAPI-07 | Provider lifecycle handling (reconciliation, health checks) | High |

### 4.3 MAPI-to-CAPI Migration

| Feature ID | Feature | Priority |
|---|---|---|
| F-MIG-01 | Machine object conversion (MAPI Machine to CAPI Machine) | High |
| F-MIG-02 | MachineSet object conversion (MAPI MachineSet to CAPI MachineSet) | High |
| F-MIG-03 | Provider config mapping — image identifiers | High |
| F-MIG-04 | Provider config mapping — subnets | High |
| F-MIG-05 | Provider config mapping — data disks | High |
| F-MIG-06 | Provider config mapping — GPUs | Medium |
| F-MIG-07 | Provider config mapping — cluster reference | High |
| F-MIG-08 | Provider config mapping — boot type | Medium |
| F-MIG-09 | Provider config mapping — project assignment | Medium |
| F-MIG-10 | Provider config mapping — categories | Medium |
| F-MIG-11 | Provider config mapping — failure domain | High |
| F-MIG-12 | Provider config mapping — user data | High |
| F-MIG-13 | Status condition conversion | High |
| F-MIG-14 | Invalid input handling (identifier, boot, GPU, disk, infrastructure) | High |

### 4.4 Machine Lifecycle

| Feature ID | Feature | Priority |
|---|---|---|
| F-LIFE-01 | Declarative machine creation | High |
| F-LIFE-02 | Machine scale-up (MachineSet replica increase) | High |
| F-LIFE-03 | Machine scale-down (MachineSet replica decrease) | High |
| F-LIFE-04 | Machine deletion / destroy | High |
| F-LIFE-05 | Machine health check and remediation | Medium |

### 4.5 Networking

| Feature ID | Feature | Priority |
|---|---|---|
| F-NET-01 | Single-subnet machine provisioning | High |
| F-NET-02 | Multi-subnet machine provisioning (feature-gated: NutanixMultiSubnets) | Medium |
| F-NET-03 | Multiple network address validation | Medium |
| F-NET-04 | Subnet mapping and unique IP assignment | Medium |
| F-NET-05 | Identifier format validation for subnets | Medium |
| F-NET-06 | Subnet limit enforcement | Medium |
| F-NET-07 | Node connectivity verification | Medium |

### 4.6 Security and Credentials

| Feature ID | Feature | Priority |
|---|---|---|
| F-SEC-01 | Prism Central credential management (Secret-based) | High |
| F-SEC-02 | Credential rotation handling | Medium |
| F-SEC-03 | TLS/certificate configuration for Prism endpoint | Medium |

### 4.7 Upgradeability

| Feature ID | Feature | Priority |
|---|---|---|
| F-UPG-01 | Upgrade from MAPI-based Nutanix cluster to CAPI-based cluster | High |
| F-UPG-02 | OCP minor version upgrade with CAPI Nutanix provider | High |
| F-UPG-03 | CAPI provider version upgrade (in-place) | Medium |

### 4.8 Observability

| Feature ID | Feature | Priority |
|---|---|---|
| F-OBS-01 | CAPI controller metrics emission | Medium |
| F-OBS-02 | Machine/MachineSet status and condition reporting | High |
| F-OBS-03 | Event generation for lifecycle operations | Medium |
| F-OBS-04 | Logging (operator and provider logs) | Medium |

### 4.9 Performance

| Feature ID | Feature | Priority |
|---|---|---|
| F-PERF-01 | Machine provisioning time parity or better vs. MAPI | Medium |
| F-PERF-02 | Scale-out time for N machines parity or better vs. MAPI | Medium |

---

## 5. Features Not to Be Tested

> **Note:** The SPLAT-2559 epic lists its "Out of Scope" as "..." — the following exclusions are inferred from the epic title and available evidence. They should be confirmed once the epic is refined.

| Exclusion | Rationale |
|---|---|
| Non-Nutanix platforms (AWS, Azure, GCP, vSphere, etc.) | Epic is Nutanix-specific |
| UPI (User-Provisioned Infrastructure) installation | Not listed in epic goals |
| MAPI-only Nutanix features with no CAPI equivalent | Out of scope for this CAPI introduction |
| Nutanix AHV hypervisor-level testing | Infrastructure provider responsibility |
| Prism Element-only deployments (without Prism Central) | CAPI provider targets Prism Central |
| Day-2 cluster operations unrelated to machine lifecycle (e.g., application workloads, OLM) | Outside machine management scope |
| Performance benchmarking beyond provisioning time comparison | Not specified in epic |
| FIPS-mode specific testing | Not mentioned in epic; if required, add as separate item |
| Disconnected / air-gapped installation | Not mentioned in epic evidence |
| Single-node OpenShift (SNO) on Nutanix | Not mentioned in epic evidence |

---

## 6. Approach

### 6.1 Test Strategy Overview

Testing follows a layered approach combining unit, integration, and end-to-end validation across the CAPI Nutanix implementation.

```
Layer 4:  Full-Stack e2e (IPI / Assisted / ABI on Nutanix hardware)
Layer 3:  Component e2e (CAPI operator + provider on live cluster)
Layer 2:  Integration (envtest-based, controller reconciliation)
Layer 1:  Unit (conversion logic, config mapping, validation)
```

### 6.2 Layer 1 — Unit Tests

| Aspect | Detail |
|---|---|
| **Scope** | Conversion functions, provider config mapping, input validation, error handling |
| **Tools** | Go `testing` package, table-driven tests |
| **Evidenced in** | PR #383 (conversion unit tests), PR #395 (valid/invalid identifier, boot, GPU, disk, infrastructure cases) |
| **CI Trigger** | `make unit` in cluster-capi-operator; `go test ./...` in installer |
| **Owner** | Feature developers |

### 6.3 Layer 2 — Integration Tests (envtest)

| Aspect | Detail |
|---|---|
| **Scope** | Controller reconciliation with mocked Nutanix API, CRD validation, webhook behavior |
| **Tools** | envtest (controller-runtime test framework), `make unit` (cluster-capi-operator bundles envtest under this target) |
| **Evidenced in** | Repository CI evidence (REF-06) |
| **CI Trigger** | `make unit` in cluster-capi-operator |
| **Owner** | Feature developers |

### 6.4 Layer 3 — Component e2e

| Aspect | Detail |
|---|---|
| **Scope** | CAPI operator with Nutanix provider on a live OCP cluster: NutanixCluster creation, infrastructure reference, readiness, managed-by annotation, control-plane endpoint, PrismCentral/failure domain config |
| **Tools** | Ginkgo/Gomega, `make e2e` (cluster-capi-operator) |
| **Evidenced in** | PR #383 (`e2e/nutanix_test.go`) |
| **CI Trigger** | `make e2e` in cluster-capi-operator |
| **Environment** | Requires live Nutanix cluster with Prism Central |
| **Owner** | Feature developers / QE |

### 6.5 Layer 4 — Full-Stack e2e

| Aspect | Detail |
|---|---|
| **Scope** | End-to-end cluster installation, machine lifecycle, multi-subnet, upgrade, migration |
| **Tools** | OTE (OpenShift Test Engine), `make test-e2e` / `make test-e2e-tech-preview` (machine-api-operator), installer e2e suites |
| **Evidenced in** | PR #1425 (multi-subnet e2e), PR #70896 (CI job definitions) |
| **CI Trigger** | Optional presubmits in openshift/release; `make test-e2e` in machine-api-operator |
| **Feature gates** | `NutanixMultiSubnets=true`, `CustomNoUpgrade` (for multi-subnet jobs) |
| **Environment** | Nutanix hardware with `nutanix-qe` CI profile |
| **Owner** | QE team |

### 6.6 Static Analysis and Verification

| Aspect | Detail |
|---|---|
| **Scope** | Code generation, formatting, linting, manifest validation |
| **Tools** | `make verify` (cluster-capi-operator) |
| **CI Trigger** | Presubmit on every PR |
| **Owner** | Feature developers |

### 6.7 Test Automation and CI

| CI Job | Repository | Releases | Trigger | Status |
|---|---|---|---|---|
| `e2e-nutanix-operator-multi-subnet` | openshift/release | 4.21, 4.22 | Optional, manual | **Evidenced** (PR #70896) |
| `make unit` presubmit | cluster-capi-operator | All | Automatic | **Evidenced** (REF-06) |
| `make e2e` presubmit | cluster-capi-operator | All | Automatic (where Nutanix env available) | **Evidenced** (REF-06) |
| `make verify` presubmit | cluster-capi-operator | All | Automatic | **Evidenced** (REF-06) |
| IPI install e2e for Nutanix CAPI | TBD | TBD | TBD | **Proposed** — no evidence of a dedicated CAPI IPI job |
| Assisted Installer e2e for Nutanix CAPI | TBD | TBD | TBD | **Proposed** — no evidence yet |
| ABI e2e for Nutanix CAPI | TBD | TBD | TBD | **Proposed** — no evidence yet |
| Upgrade e2e (MAPI-to-CAPI migration) | TBD | TBD | TBD | **Proposed** — no evidence yet |

---

## 7. Test Scenarios and Pass/Fail Criteria

### 7.1 Scenario Naming Convention

Scenario IDs follow the pattern `TS-<category>-<number>`:
- `TS-INST-*` — Installation
- `TS-CAPI-*` — CAPI provider integration
- `TS-MIG-*` — MAPI-to-CAPI migration
- `TS-LIFE-*` — Machine lifecycle
- `TS-NET-*` — Networking
- `TS-SEC-*` — Security/credentials
- `TS-UPG-*` — Upgradeability
- `TS-OBS-*` — Observability
- `TS-PERF-*` — Performance
- `TS-FAIL-*` — Failure handling

### 7.2 Scenario Evidence Labels

| Label | Meaning |
|---|---|
| **Existing** | Test code or CI job exists in referenced PRs or repositories |
| **Proposed** | Test is required by the epic but no implementation evidence is available |
| **Blocked** | Test cannot be created/executed until a dependency is resolved |
| **N/A** | Not applicable to this feature |

### 7.3 Installation Scenarios

| Scenario ID | Description | Method | Pass Criteria | Evidence |
|---|---|---|---|---|
| TS-INST-01 | Install OCP cluster on Nutanix via IPI using CAPI provider | IPI | Cluster reaches `Available` state; all nodes report `Ready`; CAPI Machine objects match expected count | **Proposed** |
| TS-INST-02 | Install OCP cluster on Nutanix via Assisted Installer using CAPI | Assisted | Cluster reaches `Available` state; assisted-service reports installation complete; CAPI resources are present | **Proposed** |
| TS-INST-03 | Install OCP cluster on Nutanix via ABI using CAPI | ABI | Cluster bootstraps successfully from agent ISO; CAPI resources reconcile | **Proposed** |
| TS-INST-04 | IPI install with custom machine configuration (disk, memory, CPU) | IPI | VMs created with specified resources; CAPI Machine spec reflects config | **Proposed** |
| TS-INST-05 | IPI install with failure domains configured | IPI | Machines distributed across specified failure domains | **Proposed** |

### 7.4 CAPI Provider Integration Scenarios

| Scenario ID | Description                                                        | Pass Criteria                                                                        | Evidence                                                       |
| -------------| --------------------------------------------------------------------| --------------------------------------------------------------------------------------| ----------------------------------------------------------------|
| TS-CAPI-01  | NutanixCluster CR is created with correct infrastructure reference | `NutanixCluster` exists; `Cluster.spec.infrastructureRef` points to it               | **Existing** — PR #383 `e2e/nutanix_test.go`                   |
| TS-CAPI-02  | `managed-by` annotation is set on CAPI resources                   | Annotation `cluster.x-k8s.io/managed-by` is present                                  | **Existing** — PR #383 `e2e/nutanix_test.go`                   |
| TS-CAPI-03  | Control-plane endpoint is correctly propagated                     | `NutanixCluster.spec.controlPlaneEndpoint` matches cluster API VIP                   | **Existing** — PR #383 `e2e/nutanix_test.go`                   |
| TS-CAPI-04  | PrismCentral configuration is applied                              | `NutanixCluster` references valid Prism Central address and port                     | **Existing** — PR #383 `e2e/nutanix_test.go`                   |
| TS-CAPI-05  | Failure domains are configured on NutanixCluster                   | `NutanixCluster.spec.failureDomains` contains expected entries                       | **Existing** — PR #383 `e2e/nutanix_test.go`                   |
| TS-CAPI-06  | NutanixCluster reaches ready state                                 | `NutanixCluster.status.ready` is `true`                                              | **Existing** — PR #383 `e2e/nutanix_test.go`                   |
| TS-CAPI-07  | CAPI Cluster references NutanixCluster endpoint                    | `Cluster.spec.controlPlaneEndpoint` is populated                                     | **Existing** — PR #383 `e2e/nutanix_test.go`                   |
| TS-CAPI-08  | Nutanix provider scheme is registered in operator                  | Operator starts without scheme registration errors; NutanixCluster GVK is recognized | **Existing** — PR #383 (scheme registration code + unit tests) |

### 7.5 MAPI-to-CAPI Migration Scenarios

| Scenario ID | Description | Pass Criteria | Evidence |
|---|---|---|---|
| TS-MIG-01 | Convert MAPI Machine to CAPI Machine — valid config | Output CAPI Machine has equivalent provider spec; no conversion errors | **Existing** — PR #395 unit tests |
| TS-MIG-02 | Convert MAPI MachineSet to CAPI MachineSet — valid config | Output CAPI MachineSet has equivalent template spec; replicas preserved | **Existing** — PR #395 unit tests |
| TS-MIG-03 | Image identifier mapping (name and UUID modes) | Correct `NutanixMachineTemplate` image reference | **Existing** — PR #395 unit tests |
| TS-MIG-04 | Subnet configuration mapping | All MAPI subnets appear in CAPI provider spec | **Existing** — PR #395 unit tests |
| TS-MIG-05 | Data disk configuration mapping | Disk size, storage container, device properties preserved | **Existing** — PR #395 unit tests |
| TS-MIG-06 | GPU passthrough configuration mapping | GPU device ID and vendor preserved | **Existing** — PR #395 unit tests |
| TS-MIG-07 | Cluster reference mapping | Prism Element cluster UUID/name carried over | **Existing** — PR #395 unit tests |
| TS-MIG-08 | Boot type mapping (Legacy, UEFI, SecureBoot) | Boot type field correctly translated | **Existing** — PR #395 unit tests |
| TS-MIG-09 | Project assignment mapping | Project reference preserved | **Existing** — PR #395 unit tests |
| TS-MIG-10 | Categories mapping | All categories carried to CAPI spec | **Existing** — PR #395 unit tests |
| TS-MIG-11 | Failure domain mapping | Failure domain name/reference preserved | **Existing** — PR #395 unit tests |
| TS-MIG-12 | User data mapping | Bootstrap data reference preserved | **Existing** — PR #395 unit tests |
| TS-MIG-13 | Status condition conversion | MAPI conditions translated to CAPI condition types | **Existing** — PR #395 unit tests |
| TS-MIG-14 | Invalid identifier input rejection | Conversion returns error for malformed identifiers | **Existing** — PR #395 unit tests |
| TS-MIG-15 | Invalid boot type input rejection | Conversion returns error for unsupported boot types | **Existing** — PR #395 unit tests |
| TS-MIG-16 | Invalid GPU config input rejection | Conversion returns error for malformed GPU specs | **Existing** — PR #395 unit tests |
| TS-MIG-17 | Invalid disk config input rejection | Conversion returns error for invalid disk specs | **Existing** — PR #395 unit tests |
| TS-MIG-18 | Invalid infrastructure reference rejection | Conversion returns error when infra ref is missing/malformed | **Existing** — PR #395 unit tests |
| TS-MIG-19 | End-to-end migration on live cluster (MAPI cluster upgraded to CAPI) | All machines continue running; workloads uninterrupted; CAPI objects replace MAPI objects | **Proposed** |

### 7.6 Machine Lifecycle Scenarios

| Scenario ID | Description | Pass Criteria | Evidence |
|---|---|---|---|
| TS-LIFE-01 | Create a new CAPI Machine on Nutanix | VM provisioned in Prism; Machine reaches `Running` phase; Node joins cluster | **Proposed** |
| TS-LIFE-02 | Scale MachineSet up by N replicas | N new Machines created; corresponding Nodes become `Ready` | **Existing** (partial) — PR #1425 covers scale-up in multi-subnet context |
| TS-LIFE-03 | Scale MachineSet down by N replicas | N Machines deleted; corresponding VMs terminated in Prism; Nodes drained and removed | **Existing** (partial) — PR #1425 covers scale-down in multi-subnet context |
| TS-LIFE-04 | Delete individual Machine | VM terminated; Node removed from cluster | **Proposed** |
| TS-LIFE-05 | MachineHealthCheck triggers remediation | Unhealthy machine replaced; new machine reaches `Running` | **Proposed** |

### 7.7 Networking Scenarios

| Scenario ID | Description | Pass Criteria | Evidence |
|---|---|---|---|
| TS-NET-01 | Single-subnet machine provisioning | Machine gets IP from specified subnet; node is reachable | **Proposed** |
| TS-NET-02 | Multi-subnet machine provisioning | Machine gets IPs from multiple subnets | **Existing** — PR #1425 `multi-subnet.go` |
| TS-NET-03 | Validate multiple network addresses on node | Node `.status.addresses` contains expected entries | **Existing** — PR #1425 |
| TS-NET-04 | Node placement respects subnet mapping | Each machine placed on correct subnet per spec | **Existing** — PR #1425 |
| TS-NET-05 | Unique IP assignment across machines | No IP collisions in provisioned machines | **Existing** — PR #1425 |
| TS-NET-06 | Subnet identifier format validation | UUID and name formats accepted; invalid formats rejected | **Existing** — PR #1425 |
| TS-NET-07 | Subnet limit enforcement | Error returned when subnet limit exceeded | **Existing** — PR #1425 |
| TS-NET-08 | Node connectivity verification post-provisioning | Nodes can reach each other and API server | **Existing** — PR #1425 |
| TS-NET-09 | Machine status and annotation correctness | Status fields and annotations reflect subnet config | **Existing** — PR #1425 |
| TS-NET-10 | Feature gate enforcement (NutanixMultiSubnets) | Multi-subnet tests skip when gate is disabled | **Existing** — PR #1425 (test skips without gate) |
| TS-NET-11 | Multi-subnet test skips without machines/IPI context | Graceful skip; no false failures | **Existing** — PR #1425 |

### 7.8 Security and Credential Scenarios

| Scenario ID | Description | Pass Criteria | Evidence |
|---|---|---|---|
| TS-SEC-01 | Prism Central credentials stored in Kubernetes Secret | Secret created in expected namespace; referenced by provider | **Proposed** |
| TS-SEC-02 | Cluster creation with valid Prism credentials | Authentication succeeds; VMs provisioned | **Proposed** |
| TS-SEC-03 | Cluster creation with invalid Prism credentials | Clear error; no partial resource creation | **Proposed** |
| TS-SEC-04 | Credential rotation without cluster disruption | Updated secret is picked up; operations continue | **Proposed** |
| TS-SEC-05 | TLS verification for Prism Central endpoint | Invalid cert rejected; valid cert accepted | **Proposed** |

### 7.9 Upgrade Scenarios

| Scenario ID | Description | Pass Criteria | Evidence |
|---|---|---|---|
| TS-UPG-01 | Upgrade MAPI-based Nutanix cluster to CAPI | All machines migrated; workloads unaffected; CAPI objects functional | **Proposed** |
| TS-UPG-02 | OCP minor version upgrade (e.g., 4.x to 4.x+1) with CAPI Nutanix | Upgrade completes; CAPI provider reconciles post-upgrade; no machine disruption | **Proposed** |
| TS-UPG-03 | CAPI provider in-place version upgrade | Provider pods restart cleanly; no machine disruption | **Proposed** |
| TS-UPG-04 | Rollback after failed upgrade attempt | Cluster returns to pre-upgrade CAPI state; machines intact | **Proposed** |

### 7.10 Failure Handling Scenarios

| Scenario ID | Description | Pass Criteria | Evidence |
|---|---|---|---|
| TS-FAIL-01 | Prism Central unreachable during machine creation | Machine stays in provisioning; retries after connectivity restored | **Proposed** |
| TS-FAIL-02 | Prism Central unreachable during scale operation | Existing machines unaffected; new machines pending | **Proposed** |
| TS-FAIL-03 | Insufficient Nutanix resources (CPU/memory/storage) | Clear error condition on Machine; no crash loop | **Proposed** |
| TS-FAIL-04 | Invalid install-config for Nutanix CAPI | Installer returns actionable error before cluster creation | **Proposed** |
| TS-FAIL-05 | CAPI operator pod crash recovery | Operator restarts; reconciliation resumes; no machine data loss | **Proposed** |
| TS-FAIL-06 | Network partition during machine provisioning | Machine reaches correct final state after partition heals | **Proposed** |

### 7.11 Observability Scenarios

| Scenario ID | Description | Pass Criteria | Evidence |
|---|---|---|---|
| TS-OBS-01 | CAPI controller emits Prometheus metrics | Metrics endpoint reachable; relevant gauges/counters present | **Proposed** |
| TS-OBS-02 | Machine status conditions are accurate | Conditions reflect actual VM state in Prism | **Proposed** |
| TS-OBS-03 | Events generated for lifecycle transitions | Events present for create, scale, delete, error conditions | **Proposed** |
| TS-OBS-04 | Operator and provider logs are structured and filterable | Logs contain structured fields; no credential leakage | **Proposed** |

### 7.12 Performance Scenarios

| Scenario ID | Description | Pass Criteria | Evidence |
|---|---|---|---|
| TS-PERF-01 | Single machine provisioning time vs. MAPI baseline | CAPI time <= MAPI time (or within documented tolerance) | **Proposed** |
| TS-PERF-02 | Batch scale-out (e.g., 10 machines) time vs. MAPI baseline | CAPI time <= MAPI time (or within documented tolerance) | **Proposed** |

---

## 8. Suspension and Resumption Criteria

### 8.1 Suspension Criteria

Testing shall be suspended if any of the following occur:

| Condition | Action |
|---|---|
| Nutanix test infrastructure unavailable (Prism Central down, hardware failure) | Suspend all Layer 3/4 tests; continue Layer 1/2 tests |
| Blocking defect in CAPI operator preventing machine creation | Suspend lifecycle and installation tests until fix merged |
| Credential/access issues preventing Nutanix API calls | Suspend all Nutanix-dependent tests |
| CI infrastructure outage | Suspend automated test runs; manual execution may continue |
| Upstream cluster-api-provider-nutanix regression | Suspend provider-dependent tests; escalate to upstream |

### 8.2 Resumption Criteria

| Condition | Resumption |
|---|---|
| Infrastructure restored and verified | Re-run failed test suites from the point of interruption |
| Blocking defect fixed and merged | Re-run full regression from the affected layer upward |
| Credentials restored | Re-run authentication-dependent tests |
| CI restored | Trigger full CI pipeline run |

---

## 9. Test Deliverables

| Deliverable | Format | Owner | Timing |
|---|---|---|---|
| This test plan document | Markdown | QE Lead | Before test execution begins |
| Unit test code (conversion, validation) | Go test files in repo | Developers | With feature PRs (existing for PRs #383, #395) |
| Component e2e test code | Go test files (`e2e/nutanix_test.go`) | Developers | With feature PRs (existing for PR #383) |
| Multi-subnet e2e test code | Go test files (`test/e2e/nutanix/multi-subnet.go`) | Developers | With feature PRs (existing for PR #1425) |
| CI job definitions | YAML in openshift/release | Developers / CI team | With feature PRs (existing for PR #70896) |
| Full-stack IPI e2e test suite | Go test files / CI job | QE | TBD — **Proposed** |
| Assisted Installer e2e test suite | Go test files / CI job | QE | TBD — **Proposed** |
| ABI e2e test suite | Go test files / CI job | QE | TBD — **Proposed** |
| Upgrade/migration e2e test suite | Go test files / CI job | QE | TBD — **Proposed** |
| Performance benchmark results | Report / dashboard | QE | TBD — **Proposed** |
| Test execution reports | JUnit XML / CI dashboards | Automated | Per CI run |
| Defect reports | Jira issues | QE | As found |
| Test summary report | Markdown / Jira comment | QE Lead | At milestone gates (TP, GA) |

---

## 10. Testing Tasks

| Task ID | Task | Depends On | Status |
|---|---|---|---|
| TT-01 | Refine SPLAT-2559 epic scope (resolve "..." in Questions and Out of Scope) | — | **Blocked** — `needs-refinement` label |
| TT-02 | Finalize test plan after scope refinement | TT-01 | **Blocked** |
| TT-03 | Review and merge PR #383 (CAPI operator Nutanix provider) | — | TBD (PR status not supplied) |
| TT-04 | Review and merge PR #395 (MAPI-to-CAPI conversion) | — | TBD (PR status not supplied) |
| TT-05 | Review and merge PR #1425 (multi-subnet e2e) | — | TBD (PR status not supplied) |
| TT-06 | Review and merge PR #70896 (CI jobs) | — | TBD (PR status not supplied) |
| TT-07 | Develop IPI e2e test suite for CAPI Nutanix | TT-03 | **Proposed** |
| TT-08 | Develop Assisted Installer e2e test suite | TT-03, TI-06 | **Proposed** |
| TT-09 | Develop ABI e2e test suite | TT-03, TI-07 | **Proposed** |
| TT-10 | Develop upgrade/migration e2e test suite | TT-03, TT-04 | **Proposed** |
| TT-11 | Provision and configure Nutanix QE lab environment | — | TBD |
| TT-12 | Configure CI jobs for IPI/Assisted/ABI e2e | TT-07, TT-08, TT-09 | **Proposed** |
| TT-13 | Execute Layer 1 (unit) test pass | TT-03, TT-04 | Ongoing with PRs |
| TT-14 | Execute Layer 2 (integration/envtest) test pass | TT-03 | Ongoing with PRs |
| TT-15 | Execute Layer 3 (component e2e) test pass | TT-03, TT-11 | TBD |
| TT-16 | Execute Layer 4 (full-stack e2e) test pass | TT-07 through TT-12, TT-11 | TBD |
| TT-17 | Performance comparison testing (CAPI vs. MAPI) | TT-16 | **Proposed** |
| TT-18 | Draft TP and GA documentation | TT-16 | **Proposed** |
| TT-19 | Final test summary report | TT-16, TT-17 | **Proposed** |

---

## 11. Environmental Needs

### 11.1 Hardware and Infrastructure

| Component | Requirement | Notes |
|---|---|---|
| Nutanix AHV cluster | Prism Central + minimum 3 AHV hosts | Required for all Layer 3/4 tests |
| Nutanix Prism Central | Version compatible with cluster-api-provider-nutanix v1.7.2 | TBD — check provider release notes |
| CPU/Memory/Storage | Sufficient for 3+ OCP clusters concurrently | TBD — sizing depends on test parallelism |
| Network | Multiple subnets for multi-subnet tests (NutanixMultiSubnets) | At least 2 subnets in the Nutanix environment |
| DNS | Resolvable API/Ingress VIPs | Standard OCP requirement |
| DHCP | Available on provisioning networks | Required for IPI |

### 11.2 Software and Tools

| Tool | Version | Purpose |
|---|---|---|
| OpenShift Container Platform | 4.21, 4.22 (per CI job evidence) | Target platform |
| cluster-api-provider-nutanix | v1.7.2 | Upstream CAPI provider |
| Go | Version per repository go.mod | Test compilation and execution |
| Ginkgo / Gomega | Per repository go.mod | e2e test framework |
| envtest (controller-runtime) | Per repository go.mod | Integration test framework |
| OTE (OpenShift Test Engine) | Latest | Full-stack e2e orchestration |
| `oc` CLI | Matching OCP version | Cluster interaction |
| CI Infrastructure | Prow / openshift/release | Automated test execution |

### 11.3 CI Profiles

| Profile | Usage | Evidence |
|---|---|---|
| `nutanix-qe` | Multi-subnet e2e jobs | **Existing** — PR #70896 |
| Nutanix IPI profile | IPI installation e2e | **TBD** |
| Nutanix Assisted profile | Assisted Installer e2e | **TBD** |
| Nutanix ABI profile | ABI e2e | **TBD** |

---

## 12. Responsibilities

| Role | Responsibility | Assignee |
|---|---|---|
| Test Plan Author | Draft and maintain this test plan | TBD |
| Epic Owner | Refine scope, answer open questions, approve test plan | TBD |
| Feature Developers | Implement unit tests, conversion tests, component e2e; fix defects | TBD |
| QE Lead | Own full-stack e2e development and execution; report results | TBD |
| QE Engineers | Write and execute test scenarios; file defect reports | TBD |
| CI/Infrastructure | Provision Nutanix lab; maintain CI job configurations | TBD |
| Documentation | TP/GA documentation for Nutanix CAPI | TBD |
| Release Manager | Gate TP/GA on test results | TBD |

---

## 13. Staffing and Training Needs

| Need | Detail | Status |
|---|---|---|
| Nutanix platform familiarity | QE staff need Nutanix AHV / Prism Central operational knowledge | TBD |
| CAPI concepts | Understanding of Cluster API resource model (Cluster, Machine, MachineSet, Infrastructure) | TBD |
| MAPI-to-CAPI migration | Knowledge of conversion semantics and migration workflow | TBD |
| envtest / Ginkgo | Go-based test framework proficiency | TBD |
| OTE framework | Full-stack e2e test authoring in OTE | TBD |
| CI pipeline management | Prow job configuration in openshift/release | TBD |

---

## 14. Schedule

> **Note:** No schedule milestones are defined in the SPLAT-2559 epic. The following is a proposed structure pending project timeline input.

| Milestone | Target Date | Dependencies | Status |
|---|---|---|---|
| Epic scope refinement complete | TBD | Resolve `needs-refinement` label, populate "Questions" and "Out of Scope" | **Blocked** |
| Test plan finalized (v2.0) | TBD | Scope refinement | **Blocked** |
| Unit/conversion tests merged | TBD | PRs #383, #395 merged | **In Progress** (PRs exist) |
| Component e2e tests passing in CI | TBD | PR #383 merged; Nutanix lab available | TBD |
| Multi-subnet e2e tests merged | TBD | PRs #1425, #70896 merged | TBD |
| Full-stack IPI e2e developed and passing | TBD | IPI CAPI support complete; lab available | **Proposed** |
| Assisted Installer e2e developed | TBD | Assisted Installer CAPI support complete | **Proposed** |
| ABI e2e developed | TBD | ABI CAPI support complete | **Proposed** |
| Upgrade/migration e2e passing | TBD | Migration path implemented and stable | **Proposed** |
| Tech Preview (TP) gate | TBD | Core e2e passing; TP docs ready | TBD |
| General Availability (GA) gate | TBD | All e2e suites passing; GA docs ready; performance validated | TBD |

---

## 15. Risks and Contingencies

| Risk ID | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R-01 | Nutanix hardware unavailability for e2e testing | Medium | High | Maintain backup lab capacity; prioritize unit/envtest coverage for logic validation |
| R-02 | Epic scope remains unrefined (`needs-refinement`) | High | Medium | Escalate to epic owner; proceed with test plan based on available evidence; iterate after refinement |
| R-03 | Assisted Installer / ABI support delayed | Medium | High | Decouple test plan sections; execute IPI tests independently; add Assisted/ABI tests when ready |
| R-04 | Upstream cluster-api-provider-nutanix regression or incompatibility | Low | High | Pin to tested version (v1.7.2); monitor upstream releases; maintain compatibility test |
| R-05 | MAPI-to-CAPI migration path not fully implemented | Medium | High | Prioritize unit test coverage for conversion; develop migration e2e incrementally |
| R-06 | CI job flakiness on Nutanix infrastructure | Medium | Medium | Implement retry logic in CI; investigate and fix flaky tests promptly |
| R-07 | Performance regression vs. MAPI | Low | Medium | Establish MAPI baseline metrics early; monitor CAPI metrics throughout development |
| R-08 | Credential/security testing gaps due to TBD scope | Medium | Medium | Document assumptions; add security scenarios as scope clarifies |
| R-09 | Feature gate interactions (NutanixMultiSubnets, CustomNoUpgrade) | Low | Medium | Test with and without feature gates; document gate dependencies |
| R-10 | Insufficient multi-subnet test environment (need 2+ subnets) | Low | Medium | Ensure lab provisioning includes multi-subnet configuration from the start |
| R-11 | Google Doc containing additional context unreadable | N/A | Low | Plan is based solely on Jira and PR evidence; revisit if doc becomes available |

---

## 16. Approvals

| Role | Name | Date | Signature |
|---|---|---|---|
| Epic Owner | TBD | TBD | |
| QE Lead | TBD | TBD | |
| Engineering Manager | TBD | TBD | |
| Release Manager | TBD | TBD | |

---

## 17. Traceability Matrix

This matrix maps SPLAT-2559 requirements (derived from the Jira epic goals) to test scenarios and their evidence status.

| Req ID | Requirement (from SPLAT-2559) | Test Scenarios | Evidence Status |
|---|---|---|---|
| REQ-01 | OCP Nutanix cluster deployment via IPI | TS-INST-01, TS-INST-04, TS-INST-05 | **Proposed** — no IPI CAPI e2e evidence |
| REQ-02 | OCP Nutanix cluster deployment via Assisted Installer | TS-INST-02 | **Proposed** — no Assisted CAPI evidence |
| REQ-03 | OCP Nutanix cluster deployment via ABI | TS-INST-03 | **Proposed** — no ABI CAPI evidence |
| REQ-04 | Replace MAPI with CAPI | TS-CAPI-01 through TS-CAPI-08, TS-MIG-01 through TS-MIG-19 | **Partially Existing** — PR #383 (provider integration e2e + unit), PR #395 (conversion unit tests); live migration e2e proposed |
| REQ-05 | Declarative create/scale/destroy lifecycle | TS-LIFE-01 through TS-LIFE-05 | **Partially Existing** — PR #1425 (scale-up/down in multi-subnet); general lifecycle proposed |
| REQ-06 | Parity for networking/storage/machine configuration | TS-NET-01 through TS-NET-11, TS-MIG-03 through TS-MIG-12 | **Partially Existing** — PR #1425 (multi-subnet e2e), PR #395 (config mapping unit tests); single-subnet and storage e2e proposed |
| REQ-07 | Provider lifecycle handling | TS-CAPI-06, TS-CAPI-07, TS-FAIL-05, TS-FAIL-06 | **Partially Existing** — PR #383 (readiness check); failure handling proposed |
| REQ-08 | e2e coverage for IPI/Assisted/ABI | TS-INST-01 through TS-INST-05 | **Proposed** — full-stack e2e not yet evidenced |
| REQ-09 | TP and GA documentation | TT-18 (test task) | **Proposed** — documentation deliverable |
| REQ-10 | Secure Prism credential management | TS-SEC-01 through TS-SEC-05 | **Proposed** — no security-specific test evidence |
| REQ-11 | Performance parity or better vs. MAPI | TS-PERF-01, TS-PERF-02 | **Proposed** — no performance test evidence |
| REQ-12 | Validated upgrade path | TS-UPG-01 through TS-UPG-04 | **Proposed** — no upgrade e2e evidence |

### 17.1 Evidence Coverage Summary

| Category | Total Scenarios | Existing | Proposed | Blocked | N/A |
|---|---|---|---|---|---|
| Installation (TS-INST) | 5 | 0 | 5 | 0 | 0 |
| CAPI Integration (TS-CAPI) | 8 | 8 | 0 | 0 | 0 |
| Migration (TS-MIG) | 19 | 18 | 1 | 0 | 0 |
| Lifecycle (TS-LIFE) | 5 | 2 (partial) | 3 | 0 | 0 |
| Networking (TS-NET) | 11 | 10 | 1 | 0 | 0 |
| Security (TS-SEC) | 5 | 0 | 5 | 0 | 0 |
| Upgrade (TS-UPG) | 4 | 0 | 4 | 0 | 0 |
| Failure Handling (TS-FAIL) | 6 | 0 | 6 | 0 | 0 |
| Observability (TS-OBS) | 4 | 0 | 4 | 0 | 0 |
| Performance (TS-PERF) | 2 | 0 | 2 | 0 | 0 |
| **Total** | **69** | **38** | **31** | **0** | **0** |

> **Key Insight:** Existing test evidence is concentrated in CAPI provider integration (e2e), MAPI-to-CAPI conversion (unit tests), and multi-subnet networking (e2e). Installation methods, security, upgrade, failure handling, observability, and performance have no evidenced test implementations and require new test development.

---

*End of Test Plan TP-SPLAT-2559-001*
