# Test Plan: vSphere Multi-Account Credential Management

**Document ID:** TP-SPLAT-2724-001
**Version:** 0.1 DRAFT
**Date:** 2026-09-02
**Status:** Draft -- Pending Review

---

## Table of Contents

1. [Test Plan Identification](#1-test-plan-identification)
2. [References](#2-references)
3. [Objectives](#3-objectives)
4. [Scope](#4-scope)
5. [Test Items](#5-test-items)
6. [Features to Be Tested](#6-features-to-be-tested)
7. [Features Not to Be Tested](#7-features-not-to-be-tested)
8. [Approach](#8-approach)
9. [Pass/Fail Criteria](#9-passfail-criteria)
10. [Suspension and Resumption Criteria](#10-suspension-and-resumption-criteria)
11. [Test Deliverables](#11-test-deliverables)
12. [Environmental Needs](#12-environmental-needs)
13. [Staffing and Responsibilities](#13-staffing-and-responsibilities)
14. [Schedule and Milestones](#14-schedule-and-milestones)
15. [Risks and Contingencies](#15-risks-and-contingencies)
16. [Approvals](#16-approvals)
17. [Traceability Matrix](#17-traceability-matrix)
18. [Test Scenarios](#18-test-scenarios)
19. [Detailed Test Cases](#19-detailed-test-cases)
20. [Test Data](#20-test-data)
21. [Tools](#21-tools)
22. [N/A and TBD Registry](#22-na-and-tbd-registry)

---

## 1. Test Plan Identification

| Field | Value |
|-------|-------|
| Document ID | TP-SPLAT-2724-001 |
| Version | 0.1 DRAFT |
| Created | 2026-09-02 |
| Author | TBD |
| Jira Epic | SPLAT-2724 |
| Parent Strategy | OCPSTRAT-2933 |

This document defines the test plan for the vSphere multi-account credential management feature described in Jira epic SPLAT-2724. It follows the IEEE 829 Standard for Software Test Documentation.

---

## 2. References

| ID | Reference | URL |
|----|-----------|-----|
| REF-01 | Jira Epic SPLAT-2724 | https://redhat.atlassian.net/browse/SPLAT-2724 |
| REF-02 | Enhancement Proposal | https://github.com/openshift/enhancements/pull/2081 |
| REF-03 | vSphere Install Docs (OCP 4.21) | https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html-single/installing_on_vmware_vsphere/index |
| REF-04 | Parent Strategy OCPSTRAT-2933 | https://redhat.atlassian.net/browse/OCPSTRAT-2933 |
| REF-05 | IEEE 829-2008 Standard for Software and System Test Documentation | N/A (industry standard) |

---

## 3. Objectives

The objectives of this test plan are to:

1. **Validate credential separation** -- Verify that the OpenShift installer/vSphere integration supports distinct vCenter credentials for initial provisioning (high privilege) versus subsequent day-2 operations (restricted privilege).
2. **Validate blast-radius reduction** -- Confirm that high-privilege provisioning credentials are not persisted in cluster configuration unless explicitly requested.
3. **Validate component credential isolation** -- Confirm that each component (machine-api, storage, diagnostics, cloud controller) can receive its own distinct credentials.
4. **Validate backward compatibility** -- Confirm that when component-specific credentials are not provided, provisioning credentials are stored and used as they are today.
5. **Validate atomicity** -- Confirm that the installation transition from provisioning to operational credentials is atomic.
6. **Validate compliance posture** -- Confirm that the feature improves auditability and meets the acceptance criteria defined in SPLAT-2724.

---

## 4. Scope

### 4.1 In Scope

Based on the epic's stated scope and acceptance criteria:

- Component-specific credential support in `install-config.yaml` (field names/schema: **TBD**)
- Behavior of the `vsphere-cloud-credentials` secret in `kube-system`:
  - When component-specific credentials **are** provided: provisioning credentials are not stored; each component's credentials are provided as separate files
  - When component-specific credentials **are not** provided: provisioning credentials are stored as they are now
- Validation logic for provisioning permissions versus minimal operational permissions (exact permission matrix: **TBD**)
- Non-functional requirement: high-privilege credentials must not persist in cluster config unless explicitly requested
- Non-functional requirement: provisioning-to-operational credential transition must be atomic
- Acceptance: install a new vSphere IPI cluster with separate provisioning and operational credentials
- Acceptance: documentation/UX for both credential inputs and lifecycle behavior

### 4.2 Out of Scope

- Non-vSphere platform credential management
- Credential rotation automation after initial install (rotation behavior: **TBD** per gap registry)
- Upgrade and reinstall behavior (not defined in the epic; see **TBD** registry)
- Performance benchmarks beyond the atomicity requirement (thresholds: **TBD**)

---

## 5. Test Items

The following items are subject to testing. Exact implementation artifacts (file names, secret keys, encoding) are **TBD** and will be filled in as the implementation is finalized.

| Item ID | Test Item | Description |
|---------|-----------|-------------|
| TI-01 | `install-config.yaml` schema | Component-specific credential fields (exact field names: **TBD**) |
| TI-02 | `vsphere-cloud-credentials` secret (`kube-system`) | Secret that holds operational credentials; behavior differs based on whether component-specific credentials are provided |
| TI-03 | Component credential files | Separate credential files created per component when component-specific credentials are supplied (exact file names/keys: **TBD**) |
| TI-04 | Provisioning credential lifecycle | Storage, non-storage, and removal of high-privilege credentials |
| TI-05 | Permission validation logic | Validation of provisioning permissions vs. minimal operational permissions (exact privileges: **TBD**) |
| TI-06 | Credential transition mechanism | Atomic transition from provisioning to operational credentials during installation |
| TI-07 | machine-api component credentials | Credential isolation for machine-api |
| TI-08 | storage component credentials | Credential isolation for storage (CSI) |
| TI-09 | diagnostics component credentials | Credential isolation for diagnostics |
| TI-10 | cloud controller component credentials | Credential isolation for cloud controller |
| TI-11 | Documentation and UX | User-facing documentation for credential inputs and lifecycle |

---

## 6. Features to Be Tested

| Feature ID | Feature | Source (Epic Scope) |
|------------|---------|---------------------|
| FT-01 | Component-specific credential input via `install-config.yaml` | Scope: "component-specific credential support in install-config.yaml" |
| FT-02 | Provisioning credential non-persistence | Non-functional: "do not persist high-privilege credentials in cluster config unless explicitly requested" |
| FT-03 | Per-component credential file creation in `vsphere-cloud-credentials` | Scope: "each component's credentials are provided as separate files" |
| FT-04 | Fallback to provisioning credentials when component credentials are absent | Scope: "when component-specific credentials are not provided, provisioning credentials are stored as they are now" |
| FT-05 | Provisioning-to-operational transition atomicity | Non-functional: "installation transition from provisioning to operational credentials must be atomic" |
| FT-06 | Permission validation (provisioning vs. operational) | Scope: "validation of provisioning permissions vs minimal operational permissions" |
| FT-07 | End-to-end IPI install with credential separation | Acceptance: "install a new vSphere IPI cluster with separate provisioning and operational credentials" |
| FT-08 | Documentation and UX for credential inputs and lifecycle | Acceptance: "documentation/UX captured for both credential inputs and lifecycle behavior" |

---

## 7. Features Not to Be Tested

| Feature | Rationale |
|---------|-----------|
| Non-vSphere platform credential management | Epic is vSphere-specific |
| Automatic credential rotation after install | Not mentioned in epic scope; rotation behavior is **TBD** |
| Upgrade from previous OCP version to version with this feature | Not defined in epic (see TBD registry) |
| Reinstall / re-provisioning scenarios | Not defined in epic (see TBD registry) |
| Performance under high-concurrency credential operations | No performance thresholds defined in epic (see TBD registry) |

---

## 8. Approach

### 8.1 Test Strategy

Testing proceeds in three tiers:

1. **Unit / component tests** -- Validate individual behaviors (credential file creation, schema validation, permission checks) in isolation. Framework and hooks: **TBD**.
2. **Integration tests** -- Validate interactions between the installer, credential storage, and consuming components (machine-api, storage, diagnostics, cloud controller) on a real or simulated vSphere environment.
3. **End-to-end (E2E) tests** -- Full vSphere IPI cluster installation with credential separation, verifying acceptance criteria from the epic.

### 8.2 Test Types

| Type | Coverage Target |
|------|-----------------|
| Positive / happy path | Credential separation works as designed |
| Negative / error handling | Invalid credentials, missing fields, partial configuration |
| Security / secret hygiene | High-privilege credentials not leaked, secret contents verified |
| Atomicity / failure injection | Transition interrupted mid-install; verify atomic rollback or completion |
| Backward compatibility | No component-specific credentials provided; existing behavior preserved |
| Documentation / UX | Docs accuracy, error messages, user guidance |

### 8.3 Parameterization

Many implementation details are **TBD** (see Section 22). Test cases use symbolic parameters enclosed in `{BRACES}` that must be bound to actual values once the implementation is finalized:

- `{INSTALL_CONFIG_FIELD}` -- the install-config.yaml field(s) for component credentials
- `{SECRET_KEY_FORMAT}` -- the key format within the `vsphere-cloud-credentials` secret
- `{COMPONENT_CRED_FILE}` -- per-component credential file name
- `{PROV_PRIVS}` -- set of provisioning-level vCenter privileges
- `{OP_PRIVS}` -- set of minimal operational vCenter privileges
- `{OCP_VERSION}` -- target OCP version/build
- `{VCENTER_VERSION}` -- minimum supported vCenter version

---

## 9. Pass/Fail Criteria

### 9.1 Individual Test Case Criteria

Each test case specifies its own expected result. A test case **passes** if the observed result matches the expected result. A test case **fails** if:

- The observed result contradicts the expected result, OR
- An unexpected error, crash, or credential exposure occurs during execution

### 9.2 Overall Test Plan Criteria

| Criterion | Threshold |
|-----------|-----------|
| All P1 (critical) test cases pass | 100% |
| All P2 (major) test cases pass | >= 95% (remaining P2 failures triaged and tracked) |
| P3 (minor / exploratory) test cases | Informational; failures logged but do not block |
| Secret-leak findings (any priority) | 0 tolerated |
| Atomicity-violation findings | 0 tolerated |

### 9.3 Acceptance Criteria Mapping

The plan **passes overall** only when both epic acceptance criteria are demonstrated:

1. A new vSphere IPI cluster is installed with separate provisioning and operational credentials (covered by TC-SPLAT2724-07-001).
2. Documentation/UX for credential inputs and lifecycle behavior is reviewed and verified (covered by TC-SPLAT2724-08-001 through 08-003).

---

## 10. Suspension and Resumption Criteria

### 10.1 Suspension Criteria

Testing shall be suspended if any of the following occur:

1. **Infrastructure unavailable** -- vSphere/vCenter test environment is inaccessible for more than 4 hours.
2. **Credential leak detected** -- Any test reveals that high-privilege credentials are persisted or exposed contrary to the non-functional requirement. Testing stops until the leak is root-caused and fixed.
3. **Atomicity violation** -- The provisioning-to-operational transition is observed to be non-atomic (partial state persisted). Testing stops until the defect is resolved.
4. **Blocking defect in installer** -- The installer cannot complete a basic vSphere IPI install (with or without credential separation), indicating a foundational defect outside the scope of this feature.
5. **Test environment compromise** -- Any indication that test credentials have been exposed outside the test environment.

### 10.2 Resumption Criteria

Testing resumes when:

- The triggering condition is resolved and verified by the test lead.
- A new build or patch is available (if a code defect caused suspension).
- Test environment is restored and confirmed operational.

---

## 11. Test Deliverables

| Deliverable | Description | Status |
|-------------|-------------|--------|
| Test Plan (this document) | IEEE 829 test plan | Draft |
| Test Cases | Detailed cases in Section 19 | Included (parameterized) |
| Test Execution Log | Record of each test run with pass/fail, date, tester, build | TBD |
| Defect Log | Bugs found during execution, linked to Jira | TBD |
| Test Summary Report | Final summary with pass rates, open defects, recommendation | TBD |

---

## 12. Environmental Needs

### 12.1 Infrastructure

| Resource | Requirement | Notes |
|----------|-------------|-------|
| vCenter Server | >= 1 instance | Version: **TBD** |
| ESXi hosts | Sufficient for IPI cluster deployment | Count: **TBD** |
| vSphere service accounts | >= 5 (1 provisioning + 4 component) | Provisioning process: **TBD** |
| Network | Routable between installer host and vCenter | Topology: **TBD** |
| DNS / DHCP | As required by OCP vSphere IPI | Standard IPI requirements |
| OCP build | Build containing the SPLAT-2724 implementation | Version: **TBD** |

### 12.2 Credential Requirements

All test credentials must be:

- Created specifically for testing (no production credentials)
- Scoped to the test vCenter environment only
- Rotated or destroyed after test completion
- Never committed to source control or included in test artifacts

| Credential Set | Purpose | Privilege Level |
|----------------|---------|-----------------|
| `PROV_CRED` | Provisioning credential (high privilege) | Full provisioning privileges (**TBD**: exact privilege list) |
| `MAPI_CRED` | machine-api operational credential | Minimal operational privileges (**TBD**) |
| `STORAGE_CRED` | storage operational credential | Minimal operational privileges (**TBD**) |
| `DIAG_CRED` | diagnostics operational credential | Minimal operational privileges (**TBD**) |
| `CCM_CRED` | cloud controller operational credential | Minimal operational privileges (**TBD**) |
| `INVALID_CRED` | Deliberately invalid credential for negative tests | No privileges |

### 12.3 Software

| Software | Version | Notes |
|----------|---------|-------|
| OpenShift Container Platform | **TBD** | Must contain SPLAT-2724 implementation |
| `oc` CLI | Matching OCP version | For cluster interaction |
| vCenter Server | **TBD** | Minimum supported version |

---

## 13. Staffing and Responsibilities

| Role | Responsibility | Assigned To |
|------|---------------|-------------|
| Test Lead | Plan maintenance, execution oversight, go/no-go recommendation | **TBD** |
| QE Engineer(s) | Test case execution, defect filing, evidence collection | **TBD** |
| Dev Engineer(s) | Defect triage, fix delivery, implementation detail clarification | **TBD** |
| Infrastructure Admin | vSphere environment provisioning, credential creation | **TBD** |
| Documentation Owner | Doc review, UX validation | **TBD** |

---

## 14. Schedule and Milestones

| Milestone | Target Date | Exit Criteria |
|-----------|-------------|---------------|
| Test plan review & approval | **TBD** | All approvers sign off (Section 16) |
| TBD parameters resolved | **TBD** | All `{PARAM}` placeholders bound to actual values |
| Test environment ready | **TBD** | Infrastructure and credentials provisioned |
| Unit / component test execution | **TBD** | All P1 unit cases pass |
| Integration test execution | **TBD** | All P1 integration cases pass |
| E2E test execution | **TBD** | Both acceptance criteria demonstrated |
| Test summary report delivered | **TBD** | Report submitted with go/no-go recommendation |

---

## 15. Risks and Contingencies

| Risk ID | Risk | Impact | Likelihood | Mitigation |
|---------|------|--------|------------|------------|
| R-01 | vSphere test environment unavailable or shared contention | Test execution delayed | Medium | Reserve dedicated environment; define scheduling windows |
| R-02 | TBD parameters not resolved before test execution | Test cases cannot be executed as written | High | Track TBD resolution as a prerequisite milestone; escalate blockers early |
| R-03 | Credential leak during testing exposes test-environment accounts | Security incident; test environment compromised | Low | Use ephemeral credentials; rotate after each test cycle; monitor vCenter audit logs |
| R-04 | Atomicity requirement is architecturally infeasible | Feature cannot meet non-functional requirement; redesign needed | Medium | Early architectural review; prototype the transition mechanism before full test execution |
| R-05 | Permission validation matrix is underspecified | Cannot verify provisioning vs. operational privilege separation | High | Engage vSphere SME to enumerate minimum privilege sets before integration testing |
| R-06 | Backward-compatibility regression | Existing (no-component-credential) installs break | Medium | Run backward-compatibility suite on every build before feature-specific tests |

---

## 16. Approvals

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Test Lead | **TBD** | | |
| QE Manager | **TBD** | | |
| Dev Lead (SPLAT) | **TBD** | | |
| Product Owner | **TBD** | | |

---

## 17. Traceability Matrix

This matrix maps each test case to the epic's goals, scope items, non-functional requirements, and acceptance criteria.

**Legend:**

- **G1** = Goal: Privilege separation (provisioning vs. operational credentials)
- **G2** = Goal: Auditability (distinguish installer vs. operator actions in vCenter)
- **S1** = Scope: Component-specific credential support in install-config.yaml
- **S2** = Scope: vsphere-cloud-credentials behavior (component creds provided)
- **S3** = Scope: vsphere-cloud-credentials behavior (component creds NOT provided)
- **S4** = Scope: Validation of provisioning vs. operational permissions
- **NF1** = Non-functional: Do not persist high-privilege creds unless explicitly requested
- **NF2** = Non-functional: Provisioning-to-operational transition must be atomic
- **AC1** = Acceptance: Install vSphere IPI cluster with separate credentials
- **AC2** = Acceptance: Documentation/UX for inputs and lifecycle

| Test Case ID | G1 | G2 | S1 | S2 | S3 | S4 | NF1 | NF2 | AC1 | AC2 |
|--------------|----|----|----|----|----|----|-----|-----|-----|-----|
| TC-SPLAT2724-01-001 | X | | X | | | | | | | |
| TC-SPLAT2724-01-002 | X | | X | | | | | | | |
| TC-SPLAT2724-01-003 | X | | X | | | | | | | |
| TC-SPLAT2724-02-001 | X | | | X | | | X | | | |
| TC-SPLAT2724-02-002 | X | | | X | | | X | | | |
| TC-SPLAT2724-02-003 | | | | X | | | | | | |
| TC-SPLAT2724-03-001 | | | | | X | | | | | |
| TC-SPLAT2724-03-002 | | | | | X | | | | | |
| TC-SPLAT2724-04-001 | | | | | | X | | | | |
| TC-SPLAT2724-04-002 | | | | | | X | | | | |
| TC-SPLAT2724-05-001 | | | X | | | | | | | |
| TC-SPLAT2724-05-002 | | | | X | | | | | | |
| TC-SPLAT2724-05-003 | | | X | X | | | | | | |
| TC-SPLAT2724-05-004 | | | | | | X | | | | |
| TC-SPLAT2724-05-005 | | | X | | | | | | | |
| TC-SPLAT2724-06-001 | | | | | | | X | | | |
| TC-SPLAT2724-06-002 | | | | | | | X | | | |
| TC-SPLAT2724-06-003 | | | | | | | X | | | |
| TC-SPLAT2724-06-004 | | X | | | | | X | | | |
| TC-SPLAT2724-07-001 | | | | | | | | X | | |
| TC-SPLAT2724-07-002 | | | | | | | | X | | |
| TC-SPLAT2724-07-003 | | | | | | | | X | | |
| TC-SPLAT2724-08-001 | X | | | | | | | | X | |
| TC-SPLAT2724-08-002 | | | | | | | | | X | |
| TC-SPLAT2724-09-001 | | | | | | | | | | X |
| TC-SPLAT2724-09-002 | | | | | | | | | | X |
| TC-SPLAT2724-09-003 | | | | | | | | | | X |

---

## 18. Test Scenarios

### Scenario Group 1: Component-Specific Credential Input (Positive)

Verify that the install-config.yaml schema accepts per-component credentials for each of the four components defined in the epic.

### Scenario Group 2: Provisioning Credential Non-Persistence (Security)

Verify that when component-specific credentials are provided, provisioning (high-privilege) credentials are not stored in cluster configuration.

### Scenario Group 3: Backward Compatibility (Fallback)

Verify that when component-specific credentials are NOT provided, provisioning credentials are stored and used exactly as they are today -- no behavioral change.

### Scenario Group 4: Permission Validation

Verify that the system validates provisioning permissions versus minimal operational permissions and rejects configurations that do not meet requirements.

### Scenario Group 5: Negative / Error Handling

Verify proper behavior when invalid, incomplete, or malformed credential configurations are supplied.

### Scenario Group 6: Secret Hygiene and Security

Verify that high-privilege credentials are not leaked, persisted, or exposed beyond the provisioning phase unless explicitly requested.

### Scenario Group 7: Atomicity and Failure Injection

Verify that the transition from provisioning to operational credentials is atomic and handles failures correctly.

### Scenario Group 8: End-to-End IPI Install with Credential Separation

Verify the full acceptance criterion: install a new vSphere IPI cluster with separate provisioning and operational credentials.

### Scenario Group 9: Documentation and UX

Verify that documentation and user experience for credential inputs and lifecycle behavior are complete and accurate.

---

## 19. Detailed Test Cases

### Legend

- **Priority:** P1 (critical), P2 (major), P3 (minor/exploratory)
- **Type:** Positive, Negative, Security, Atomicity, Backward-Compat, Doc/UX
- Parameters in `{BRACES}` are symbolic and must be resolved before execution (see Section 22)

---

### Scenario Group 1: Component-Specific Credential Input

#### TC-SPLAT2724-01-001: Accept component-specific credentials for all four components

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-01-001 |
| **Priority** | P1 |
| **Type** | Positive |
| **Objective** | Verify that install-config.yaml accepts per-component credential entries for machine-api, storage, diagnostics, and cloud controller |
| **Preconditions** | Valid install-config.yaml template; valid credential sets `MAPI_CRED`, `STORAGE_CRED`, `DIAG_CRED`, `CCM_CRED` |
| **Input** | install-config.yaml with `{INSTALL_CONFIG_FIELD}` populated for all four components |
| **Steps** | 1. Construct install-config.yaml with provisioning credential and all four component-specific credentials in `{INSTALL_CONFIG_FIELD}` sections. 2. Run installer validation (or `openshift-install create manifests`). 3. Inspect output for validation errors. |
| **Expected Result** | Validation succeeds; no errors related to credential fields. Installer accepts the configuration. |
| **TBD Dependencies** | `{INSTALL_CONFIG_FIELD}` -- exact schema field names |

#### TC-SPLAT2724-01-002: Accept component-specific credentials for a subset of components

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-01-002 |
| **Priority** | P1 |
| **Type** | Positive |
| **Objective** | Verify that providing credentials for only some components (e.g., machine-api and storage but not diagnostics or cloud controller) is accepted |
| **Preconditions** | Valid install-config.yaml template; credential sets for a subset of components |
| **Input** | install-config.yaml with `{INSTALL_CONFIG_FIELD}` populated for 2 of 4 components |
| **Steps** | 1. Construct install-config.yaml with provisioning credential and credentials for machine-api and storage only. 2. Run installer validation. 3. Inspect output. |
| **Expected Result** | Validation succeeds. Components with credentials get component-specific credentials; components without get provisioning credentials (per backward-compatibility scope). |
| **TBD Dependencies** | `{INSTALL_CONFIG_FIELD}`; behavior when only some components have credentials (confirm: do non-specified components fall back to provisioning creds?) |

#### TC-SPLAT2724-01-003: Reject install-config with invalid credential schema

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-01-003 |
| **Priority** | P2 |
| **Type** | Negative |
| **Objective** | Verify that install-config.yaml with malformed component credential entries is rejected with a clear error |
| **Preconditions** | install-config.yaml template |
| **Input** | install-config.yaml with syntactically invalid content in `{INSTALL_CONFIG_FIELD}` (e.g., wrong data type, missing required subfields) |
| **Steps** | 1. Construct install-config.yaml with malformed credential entries. 2. Run installer validation. 3. Capture error output. |
| **Expected Result** | Validation fails with a clear, actionable error message identifying the malformed field. |
| **TBD Dependencies** | `{INSTALL_CONFIG_FIELD}`; exact error messages (**TBD**) |

---

### Scenario Group 2: Provisioning Credential Non-Persistence

#### TC-SPLAT2724-02-001: Provisioning credentials not stored when component credentials are provided

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-02-001 |
| **Priority** | P1 |
| **Type** | Security |
| **Objective** | Verify that when all four component-specific credentials are provided, the provisioning (high-privilege) credential is NOT persisted in `vsphere-cloud-credentials` in `kube-system` or any other cluster configuration |
| **Preconditions** | Successful IPI install with all four component-specific credentials |
| **Input** | install-config.yaml with `PROV_CRED` and all four component credentials |
| **Steps** | 1. Install cluster with full component credentials. 2. After install completes, inspect `vsphere-cloud-credentials` secret in `kube-system`. 3. Search all secrets in `kube-system` and `openshift-*` namespaces for `PROV_CRED` values. 4. Inspect cluster ConfigMaps and other configuration objects for provisioning credential data. |
| **Expected Result** | Provisioning credential values are not present in any cluster-stored secret or configuration. The `vsphere-cloud-credentials` secret contains only component-specific credential files. |
| **TBD Dependencies** | `{SECRET_KEY_FORMAT}`; `{COMPONENT_CRED_FILE}` names |

#### TC-SPLAT2724-02-002: Each component receives its own credential file in vsphere-cloud-credentials

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-02-002 |
| **Priority** | P1 |
| **Type** | Positive |
| **Objective** | Verify that when component-specific credentials are provided, the `vsphere-cloud-credentials` secret in `kube-system` contains separate files for each component |
| **Preconditions** | Successful IPI install with all four component-specific credentials |
| **Input** | install-config.yaml with all four component credentials |
| **Steps** | 1. Install cluster with all component credentials. 2. Retrieve `vsphere-cloud-credentials` secret from `kube-system`. 3. List the data keys in the secret. 4. Verify each component has a distinct entry. 5. Decode each entry and verify it matches the corresponding component credential (not the provisioning credential). |
| **Expected Result** | Secret contains >= 4 distinct entries (one per component). Each entry's decoded content matches the corresponding `{COMPONENT}_CRED` values. No entry contains `PROV_CRED` values. |
| **TBD Dependencies** | `{SECRET_KEY_FORMAT}`; `{COMPONENT_CRED_FILE}` naming convention |

#### TC-SPLAT2724-02-003: Explicit opt-in to persist provisioning credentials

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-02-003 |
| **Priority** | P2 |
| **Type** | Positive |
| **Objective** | Verify that when an administrator explicitly requests it, provisioning credentials ARE stored in cluster config |
| **Preconditions** | Valid install-config.yaml with component credentials AND an explicit opt-in to persist provisioning credentials |
| **Input** | install-config.yaml with `{EXPLICIT_PERSIST_FLAG}` set to true (or equivalent) |
| **Steps** | 1. Construct install-config.yaml with component credentials AND the explicit persistence opt-in. 2. Install cluster. 3. Inspect `vsphere-cloud-credentials` and other config objects for provisioning credential data. |
| **Expected Result** | Provisioning credentials are stored as requested. Component-specific credentials are also present. |
| **TBD Dependencies** | `{EXPLICIT_PERSIST_FLAG}` -- mechanism for "explicitly requested" persistence is undefined in the epic. **Open question**: what constitutes "explicitly requested"? |

---

### Scenario Group 3: Backward Compatibility

#### TC-SPLAT2724-03-001: No component credentials -- provisioning credentials stored as today

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-03-001 |
| **Priority** | P1 |
| **Type** | Backward-Compat |
| **Objective** | Verify that when no component-specific credentials are provided, provisioning credentials are stored in `vsphere-cloud-credentials` exactly as in current (pre-feature) behavior |
| **Preconditions** | install-config.yaml with only provisioning credentials (no component-specific fields) |
| **Input** | Standard install-config.yaml (current format, no new credential fields) |
| **Steps** | 1. Install vSphere IPI cluster using a standard install-config.yaml with no component-specific credentials. 2. Inspect `vsphere-cloud-credentials` in `kube-system`. 3. Compare secret structure and content to a baseline cluster installed without this feature. |
| **Expected Result** | Secret structure, key names, and encoded values are identical to pre-feature behavior. All components receive the provisioning credential. |

#### TC-SPLAT2724-03-002: Existing install-config format is accepted without errors

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-03-002 |
| **Priority** | P1 |
| **Type** | Backward-Compat |
| **Objective** | Verify that an install-config.yaml in the current (pre-feature) format is accepted without deprecation warnings or errors |
| **Preconditions** | install-config.yaml from current OCP documentation (no component credential fields) |
| **Input** | Current-format install-config.yaml |
| **Steps** | 1. Run installer validation with current-format install-config.yaml on a build containing the SPLAT-2724 feature. 2. Inspect stdout/stderr for warnings or errors. |
| **Expected Result** | No errors or warnings related to credential configuration. Install proceeds normally. |

---

### Scenario Group 4: Permission Validation

#### TC-SPLAT2724-04-001: Provisioning credentials validated for sufficient privileges

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-04-001 |
| **Priority** | P2 |
| **Type** | Positive |
| **Objective** | Verify that the installer validates provisioning credentials have sufficient privileges for the provisioning phase |
| **Preconditions** | vCenter service account with `{PROV_PRIVS}` privileges |
| **Input** | install-config.yaml with provisioning credential that has full provisioning privileges |
| **Steps** | 1. Provide provisioning credential with `{PROV_PRIVS}`. 2. Run installer. 3. Observe validation outcome. |
| **Expected Result** | Validation passes. Install proceeds. |
| **TBD Dependencies** | `{PROV_PRIVS}` -- exact provisioning privilege set |

#### TC-SPLAT2724-04-002: Operational credentials validated for minimal required privileges

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-04-002 |
| **Priority** | P2 |
| **Type** | Positive |
| **Objective** | Verify that the installer validates each component's operational credential has at least the minimal operational privileges required for that component |
| **Preconditions** | vCenter service accounts with `{OP_PRIVS}` per component |
| **Input** | install-config.yaml with component credentials at minimal privilege level |
| **Steps** | 1. Provide component credentials with exactly `{OP_PRIVS}` for each component. 2. Run installer. 3. Observe validation outcome. |
| **Expected Result** | Validation passes for all four components. |
| **TBD Dependencies** | `{OP_PRIVS}` per component -- exact minimal privilege sets |

---

### Scenario Group 5: Negative / Error Handling

#### TC-SPLAT2724-05-001: Missing required field in component credential entry

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-05-001 |
| **Priority** | P2 |
| **Type** | Negative |
| **Objective** | Verify that omitting a required subfield within a component credential entry produces a clear validation error |
| **Preconditions** | install-config.yaml template |
| **Input** | install-config.yaml with a component credential entry missing a required subfield (e.g., password but not username) |
| **Steps** | 1. Construct install-config.yaml with incomplete component credential. 2. Run installer validation. 3. Capture error output. |
| **Expected Result** | Clear error message identifying the missing field and the affected component. |
| **TBD Dependencies** | `{INSTALL_CONFIG_FIELD}` subfield names; exact error messages |

#### TC-SPLAT2724-05-002: Component credential with insufficient vCenter privileges

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-05-002 |
| **Priority** | P2 |
| **Type** | Negative |
| **Objective** | Verify that providing a component credential with insufficient vCenter privileges is detected and reported |
| **Preconditions** | vCenter service account with fewer privileges than `{OP_PRIVS}` |
| **Input** | install-config.yaml with a component credential that lacks required permissions |
| **Steps** | 1. Create a vCenter account with a subset of `{OP_PRIVS}`. 2. Provide it as a component credential. 3. Run installer. 4. Observe result. |
| **Expected Result** | Validation fails (or post-install health check fails) with a message identifying the privilege gap. |
| **TBD Dependencies** | `{OP_PRIVS}`; when validation occurs (pre-install vs. post-install) |

#### TC-SPLAT2724-05-003: Provisioning credential with insufficient privileges

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-05-003 |
| **Priority** | P1 |
| **Type** | Negative |
| **Objective** | Verify that provisioning credentials with insufficient privileges cause a clear failure during install |
| **Preconditions** | vCenter service account with fewer privileges than `{PROV_PRIVS}` |
| **Input** | install-config.yaml with under-privileged provisioning credential |
| **Steps** | 1. Provide provisioning credential lacking required provisioning privileges. 2. Run installer. 3. Capture output. |
| **Expected Result** | Install fails with a clear error message before or during provisioning. No partial cluster state is left. |
| **TBD Dependencies** | `{PROV_PRIVS}`; exact failure behavior |

#### TC-SPLAT2724-05-004: Credential pointing to wrong vCenter / unreachable host

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-05-004 |
| **Priority** | P2 |
| **Type** | Negative |
| **Objective** | Verify that a component credential referencing a wrong or unreachable vCenter endpoint produces a clear error |
| **Preconditions** | Valid install-config.yaml template |
| **Input** | Component credential with incorrect vCenter hostname |
| **Steps** | 1. Provide a component credential with a hostname that does not resolve or points to the wrong vCenter. 2. Run installer. 3. Capture output. |
| **Expected Result** | Clear error message identifying the connectivity or authentication failure and the affected component. |

#### TC-SPLAT2724-05-005: Duplicate component credential entries in install-config

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-05-005 |
| **Priority** | P3 |
| **Type** | Negative |
| **Objective** | Verify behavior when the same component has duplicate credential entries in install-config.yaml |
| **Preconditions** | install-config.yaml template |
| **Input** | install-config.yaml with two different credential entries for the same component |
| **Steps** | 1. Construct install-config.yaml with duplicate entries for one component. 2. Run installer validation. 3. Observe behavior. |
| **Expected Result** | Either validation rejects the duplicate with a clear error, or documented precedence rules apply (first wins, last wins). Behavior must be deterministic. |
| **TBD Dependencies** | Expected behavior for duplicates is undefined in the epic |

---

### Scenario Group 6: Secret Hygiene and Security

#### TC-SPLAT2724-06-001: High-privilege credential absent from cluster secrets post-install

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-06-001 |
| **Priority** | P1 |
| **Type** | Security |
| **Objective** | Comprehensively verify that the high-privilege provisioning credential is not stored anywhere in the running cluster after a component-credential install |
| **Preconditions** | Successful IPI install with all four component credentials |
| **Steps** | 1. Install cluster with all component credentials. 2. Enumerate all Secrets across all namespaces: `oc get secrets -A`. 3. For each secret, decode and search for `PROV_CRED` username and password values. 4. Search ConfigMaps across all namespaces for provisioning credential values. 5. Search etcd (if accessible) for provisioning credential values. |
| **Expected Result** | Zero matches for provisioning credential values in any cluster-stored object. |

#### TC-SPLAT2724-06-002: Component credentials are isolated between components

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-06-002 |
| **Priority** | P1 |
| **Type** | Security |
| **Objective** | Verify that each component only has access to its own credential, not to credentials of other components |
| **Preconditions** | Successful IPI install with all four distinct component credentials |
| **Steps** | 1. Install cluster with four distinct component credentials. 2. For each component, inspect the credential data it receives. 3. Verify that machine-api has only `MAPI_CRED`, storage has only `STORAGE_CRED`, etc. 4. Verify no component can read another component's credential file. |
| **Expected Result** | Each component's credential file contains only its own credential. No cross-component credential access is possible. |
| **TBD Dependencies** | `{COMPONENT_CRED_FILE}` naming; isolation mechanism details |

#### TC-SPLAT2724-06-003: Credential values not present in installer logs

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-06-003 |
| **Priority** | P1 |
| **Type** | Security |
| **Objective** | Verify that neither provisioning nor component credentials are logged in plaintext by the installer |
| **Preconditions** | install-config.yaml with all credentials |
| **Steps** | 1. Run installer with verbose/debug logging enabled. 2. Capture all installer log output. 3. Search logs for any credential username or password values (provisioning and component). |
| **Expected Result** | No credential values appear in installer logs. Credentials may appear as redacted/masked placeholders only. |

#### TC-SPLAT2724-06-004: vCenter audit log distinguishes component identities

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-06-004 |
| **Priority** | P2 |
| **Type** | Security |
| **Objective** | Verify that vCenter audit logs show distinct identities for each component's operations, supporting the compliance/auditability goal |
| **Preconditions** | Successful IPI install with all four distinct component credentials; vCenter audit logging enabled |
| **Steps** | 1. Install cluster with distinct component credentials. 2. Trigger operations for each component (e.g., machine creation for machine-api, volume provisioning for storage). 3. Review vCenter audit/event logs. 4. Verify each operation is attributed to the corresponding component's service account. |
| **Expected Result** | Each component's vCenter API calls are logged under the component's own credential identity, not under the provisioning credential. |

---

### Scenario Group 7: Atomicity and Failure Injection

#### TC-SPLAT2724-07-001: Successful atomic transition from provisioning to operational

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-07-001 |
| **Priority** | P1 |
| **Type** | Atomicity |
| **Objective** | Verify that the transition from provisioning credentials to operational credentials during installation is atomic -- either fully complete or fully rolled back |
| **Preconditions** | Valid install-config.yaml with provisioning and component credentials |
| **Steps** | 1. Install cluster with credential separation. 2. Monitor the credential transition point during installation (mechanism: **TBD**). 3. After install completes, verify the cluster is in a consistent state: all components use their operational credentials, no provisioning credential remnants exist. |
| **Expected Result** | Post-install state shows clean operational credentials for all components. No intermediate/mixed state is observable. |
| **TBD Dependencies** | How to observe the transition point; exact definition of "atomic" in this context |

#### TC-SPLAT2724-07-002: Failure during credential transition -- no partial state

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-07-002 |
| **Priority** | P1 |
| **Type** | Atomicity |
| **Objective** | Verify that if a failure occurs during the provisioning-to-operational credential transition, the cluster does not end up in a partially-transitioned state |
| **Preconditions** | Valid install-config.yaml with credential separation; ability to inject failure during transition |
| **Input** | Simulated failure at the credential transition point (e.g., kill installer process, network interruption to vCenter, revoke provisioning credential mid-transition) |
| **Steps** | 1. Begin cluster installation with credential separation. 2. Inject failure during the credential transition phase (method: **TBD**). 3. Inspect cluster state after failure. 4. Verify credentials are either all-provisioning or all-operational, not a mix. |
| **Expected Result** | Cluster is in a consistent state: either the transition completed fully (all operational) or did not start (all provisioning). No component has a credential from a different phase than other components. |
| **TBD Dependencies** | Failure injection mechanism; how to identify the transition phase; expected recovery behavior |

#### TC-SPLAT2724-07-003: Recovery after failed transition

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-07-003 |
| **Priority** | P2 |
| **Type** | Atomicity |
| **Objective** | Verify that after a failed credential transition, the installation can be retried and completes successfully |
| **Preconditions** | Failed install from TC-SPLAT2724-07-002 |
| **Steps** | 1. After the failure in TC-07-002, clean up as needed (method: **TBD**). 2. Re-run the installer with the same configuration. 3. Verify install completes successfully with proper credential separation. |
| **Expected Result** | Re-run succeeds. All components have their correct operational credentials. No provisioning credential remnants. |
| **TBD Dependencies** | Cleanup/retry procedure after failed transition |

---

### Scenario Group 8: End-to-End IPI Install

#### TC-SPLAT2724-08-001: Full IPI install with all four component credentials

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-08-001 |
| **Priority** | P1 |
| **Type** | Positive |
| **Objective** | Demonstrate the primary acceptance criterion: install a new vSphere IPI cluster with separate provisioning and operational credentials |
| **Preconditions** | vSphere environment; valid credential sets; install-config.yaml with all four component credentials |
| **Steps** | 1. Prepare install-config.yaml with `PROV_CRED` and `MAPI_CRED`, `STORAGE_CRED`, `DIAG_CRED`, `CCM_CRED`. 2. Run `openshift-install create cluster`. 3. Wait for install to complete. 4. Verify cluster is healthy (`oc get clusteroperators`). 5. Verify each component is using its dedicated credential. 6. Verify provisioning credential is not persisted. 7. Perform a basic smoke test (create a machine, provision a volume). |
| **Expected Result** | Cluster installs successfully. All components operational with their own credentials. Provisioning credential not stored. Smoke tests pass. |

#### TC-SPLAT2724-08-002: IPI install with provisioning-only credentials (backward compat E2E)

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-08-002 |
| **Priority** | P1 |
| **Type** | Backward-Compat |
| **Objective** | Verify that the standard (no component credentials) install path still works identically on a build containing this feature |
| **Preconditions** | vSphere environment; standard install-config.yaml |
| **Steps** | 1. Install cluster using current install-config.yaml format (no component credentials). 2. Verify cluster health. 3. Verify `vsphere-cloud-credentials` contains provisioning credential as in pre-feature builds. 4. Smoke test. |
| **Expected Result** | Cluster installs and operates identically to a build without this feature. |

---

### Scenario Group 9: Documentation and UX

#### TC-SPLAT2724-09-001: Documentation covers credential input configuration

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-09-001 |
| **Priority** | P1 |
| **Type** | Doc/UX |
| **Objective** | Verify that user-facing documentation describes how to configure per-component credentials in install-config.yaml |
| **Preconditions** | Published or draft documentation |
| **Steps** | 1. Review documentation for install-config.yaml credential configuration. 2. Verify it describes: field names, required vs. optional fields, supported credential formats. 3. Verify it includes a complete example. 4. Follow the documented procedure on a test cluster. |
| **Expected Result** | Documentation is complete, accurate, and sufficient for a user to configure credential separation without additional guidance. Following the documented steps produces a successful install. |
| **TBD Dependencies** | Documentation location and format |

#### TC-SPLAT2724-09-002: Documentation covers credential lifecycle behavior

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-09-002 |
| **Priority** | P1 |
| **Type** | Doc/UX |
| **Objective** | Verify that documentation explains the credential lifecycle: what happens to provisioning credentials after install, how component credentials are stored, and the atomicity guarantee |
| **Preconditions** | Published or draft documentation |
| **Steps** | 1. Review documentation for lifecycle description. 2. Verify it addresses: provisioning credential fate, component credential storage location, transition behavior, and what "explicitly requested" persistence means. |
| **Expected Result** | Lifecycle behavior is clearly documented and matches observed behavior from test execution. |

#### TC-SPLAT2724-09-003: Error messages are actionable

| Field | Value |
|-------|-------|
| **ID** | TC-SPLAT2724-09-003 |
| **Priority** | P2 |
| **Type** | Doc/UX |
| **Objective** | Verify that error messages produced by validation failures and credential issues are user-friendly and actionable |
| **Preconditions** | Results from negative test cases (TC-05-xxx) |
| **Steps** | 1. Collect all error messages from negative test case executions. 2. Evaluate each for: clarity (identifies the problem), specificity (names the affected field/component), actionability (suggests how to fix). |
| **Expected Result** | All error messages meet minimum UX quality: identify the problem, name the affected component/field, and either suggest a fix or reference documentation. |
| **TBD Dependencies** | Exact error message requirements |

---

## 20. Test Data

### 20.1 Credential Sets

All credentials below are **synthetic test-only values**. Real credentials must never appear in this document or test artifacts.

| Set Name | Purpose | Username Pattern | Password Pattern |
|----------|---------|-----------------|------------------|
| `PROV_CRED` | Provisioning (high privilege) | `test-prov-user@vsphere.local` | Randomly generated, 20+ chars |
| `MAPI_CRED` | machine-api operations | `test-mapi-user@vsphere.local` | Randomly generated, 20+ chars |
| `STORAGE_CRED` | storage operations | `test-storage-user@vsphere.local` | Randomly generated, 20+ chars |
| `DIAG_CRED` | diagnostics operations | `test-diag-user@vsphere.local` | Randomly generated, 20+ chars |
| `CCM_CRED` | cloud controller operations | `test-ccm-user@vsphere.local` | Randomly generated, 20+ chars |
| `INVALID_CRED` | Negative testing | `invalid-user@vsphere.local` | `invalid-password` |

### 20.2 install-config.yaml Templates

Templates will be maintained alongside this test plan. Each template is a parameterized install-config.yaml with placeholders for:

- `{INSTALL_CONFIG_FIELD}` -- component credential field names
- `{VCENTER_HOST}` -- vCenter server hostname
- `{DATACENTER}` -- vSphere datacenter name
- `{CLUSTER_NAME}` -- OCP cluster name
- `{BASE_DOMAIN}` -- DNS base domain

### 20.3 Privilege Sets

| Set Name | Description | Exact Privileges |
|----------|-------------|-----------------|
| `{PROV_PRIVS}` | Full provisioning privileges | **TBD** -- must be enumerated by implementation team |
| `{OP_PRIVS}` | Minimal operational privileges (per component) | **TBD** -- must be enumerated per component |
| `{INSUFFICIENT_PRIVS}` | Deliberately insufficient privileges (for negative tests) | Strict subset of `{OP_PRIVS}` |

---

## 21. Tools

| Tool | Purpose | Version |
|------|---------|---------|
| `openshift-install` | Cluster installation | **TBD** (matching OCP build) |
| `oc` | Cluster inspection, secret retrieval, operator status | **TBD** (matching OCP build) |
| `kubectl` | Supplemental cluster interaction | Compatible with cluster API version |
| `jq` | JSON parsing for secret inspection | >= 1.6 |
| `grep` / `ripgrep` | Credential-leak scanning across logs and secrets | Any recent version |
| Test automation framework | Automated test execution | **TBD** -- framework not specified in epic |
| Go test runner (`go test`) | Unit/component test execution (if applicable) | **TBD** |
| vCenter UI / API | Audit log review, privilege verification | Matching vCenter version |

---

## 22. N/A and TBD Registry

This section consolidates all items that are undefined, not yet decided, or explicitly out of scope. Each entry must be resolved before the corresponding test cases can be executed.

### TBD Items (must be resolved before test execution)

| ID | Item | Affected Test Cases | Resolution Owner | Target Date |
|----|------|---------------------|-----------------|-------------|
| TBD-01 | **Exact install-config.yaml field names and schema shape** for component-specific credentials | TC-01-001, TC-01-002, TC-01-003, TC-05-001, TC-05-005 | Dev team | TBD |
| TBD-02 | **Exact vCenter privilege matrix** -- provisioning privileges (`{PROV_PRIVS}`) and per-component minimal operational privileges (`{OP_PRIVS}`) | TC-04-001, TC-04-002, TC-05-002, TC-05-003 | Dev team + vSphere SME | TBD |
| TBD-03 | **Supported OCP versions/builds** containing this feature | All test cases (environment setup) | Dev team | TBD |
| TBD-04 | **Exact file names, secret keys, and encoding format** within `vsphere-cloud-credentials` for per-component credential files | TC-02-001, TC-02-002, TC-06-001, TC-06-002 | Dev team | TBD |
| TBD-05 | **Implementation test hooks and automation framework** for unit, integration, and E2E tests | All test cases (execution method) | QE + Dev team | TBD |
| TBD-06 | **Exact UX and error-message requirements** for validation failures | TC-01-003, TC-05-001 through TC-05-005, TC-09-003 | UX + Dev team | TBD |
| TBD-07 | **Performance and security acceptance thresholds** (e.g., maximum acceptable transition time, secret-scan coverage requirements) | TC-07-001 | QE + Dev team | TBD |
| TBD-08 | **Environment topology and test-account provisioning process** -- number of vCenters, ESXi hosts, network layout, account creation procedure | All test cases (environment setup) | Infrastructure team | TBD |
| TBD-09 | **Mechanism for "explicitly requested" provisioning credential persistence** (`{EXPLICIT_PERSIST_FLAG}`) | TC-02-003 | Dev team | TBD |
| TBD-10 | **Failure injection mechanism** for atomicity testing -- how to interrupt the credential transition mid-install | TC-07-002, TC-07-003 | QE + Dev team | TBD |
| TBD-11 | **Minimum supported vCenter version** | All test cases (environment setup) | Dev team | TBD |
| TBD-12 | **Staffing assignments** | All | Test Lead | TBD |
| TBD-13 | **Schedule dates** | All milestones | Project Manager | TBD |
| TBD-14 | **Behavior when only some components have credentials** -- do non-specified components fall back to provisioning credentials? | TC-01-002 | Dev team | TBD |

### N/A Items (explicitly out of scope for this plan)

| ID | Item | Rationale |
|----|------|-----------|
| NA-01 | **Credential rotation after install** | Not defined in epic scope. Rotation behavior is not addressed by SPLAT-2724 acceptance criteria. |
| NA-02 | **Upgrade from pre-feature OCP version** | Not defined in epic scope. Upgrade behavior is not specified. |
| NA-03 | **Reinstall / re-provisioning behavior** | Not defined in epic scope. |
| NA-04 | **Non-vSphere platforms** | Epic is explicitly vSphere-specific. |

### Open Questions (from epic and enhancement, unresolved)

| ID | Question | Impact on Testing |
|----|----------|-------------------|
| OQ-01 | What constitutes "explicitly requested" persistence of provisioning credentials? Is it a flag in install-config.yaml, a CLI option, or something else? | Affects TC-02-003 design |
| OQ-02 | When component-specific credentials are provided for only a subset of components, do the remaining components receive the provisioning credential or no credential? | Affects TC-01-002 expected result |
| OQ-03 | At what point is permission validation performed -- pre-install (dry-run) or during provisioning? | Affects TC-04-001, TC-04-002, TC-05-002 step design |
| OQ-04 | What is the exact definition of "atomic" for the credential transition? Is it transactional (all-or-nothing) or idempotent-retry? | Affects TC-07-001 through TC-07-003 expected results |

---

*End of Test Plan TP-SPLAT-2724-001*
