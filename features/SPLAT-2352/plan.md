# Configuring Additional etcd Disk for Azure

**Test Plan Identifier:** SPLAT-2352-TP-001

**Plan Status:** Draft  

**Feature:** [SPLAT-2352](https://redhat.atlassian.net/browse/SPLAT-2352) — [GA] Configuring Additional etcd Disk for Azure  

**Priority:** Critical  

**Feature Status:** Planning; the feature has been reported as partially included in a 5.1 nightly, but is not yet considered GA.  

**Primary Components:** OpenShift Installer, Machine API, Machine Config Operator, Azure platform, etcd  

## 1. Introduction

This test plan defines the verification strategy for configuring an additional Azure data disk dedicated to etcd during OpenShift installation. The plan verifies the complete path from install configuration through generated machine resources, MachineConfig and Ignition delivery, node-level disk setup, etcd operation, and cluster upgrade.

The plan also covers regression coverage for the shared multi-disk setup mechanism used by etcd, swap, and user-defined disks. The primary acceptance target remains a reliable Azure cluster with etcd data isolated on the configured additional disk.

## 2. References

- [SPLAT-2352](https://redhat.atlassian.net/browse/SPLAT-2352) — Feature epic
- [SPLAT-2374](https://redhat.atlassian.net/browse/SPLAT-2374) — Pre-merge testing
- [SPLAT-2375](https://redhat.atlassian.net/browse/SPLAT-2375) — E2E testing automation
- [SPLAT-2376](https://redhat.atlassian.net/browse/SPLAT-2376) — CI implementation
- [SPLAT-2419](https://redhat.atlassian.net/browse/SPLAT-2419) — Upgrade CI implementation
- [SPLAT-2510](https://redhat.atlassian.net/browse/SPLAT-2510) — Machine Config Operator e2e coverage
- [OCPBUGS-59520](https://redhat.atlassian.net/browse/OCPBUGS-59520) — Managed disk configuration validation
- [OCPBUGS-59521](https://redhat.atlassian.net/browse/OCPBUGS-59521) — Disk Encryption Set handling
- [OCPBUGS-59522](https://redhat.atlassian.net/browse/OCPBUGS-59522) — Unsupported data-disk security encryption configuration
- [OCPBUGS-59743](https://redhat.atlassian.net/browse/OCPBUGS-59743) — Azure Stack Hub behavior
- [OCPBUGS-61892](https://redhat.atlassian.net/browse/OCPBUGS-61892) — Swap disk regression
- [openshift/installer#9947](https://github.com/openshift/installer/pull/9947) — Azure data-disk fixes
- [openshift/installer#10776](https://github.com/openshift/installer/pull/10776) — Disk Encryption Set handling
- [openshift/installer#10777](https://github.com/openshift/installer/pull/10777) — Validation for unsupported disk security encryption
- [openshift/installer#10823](https://github.com/openshift/installer/pull/10823) — Azure Stack Hub rejection behavior

## 3. Test Items

The following items are under test:
- Azure install-config fields for additional data disks and disk setup.
- The MultiDiskSetup feature-gate path in the installer.
- Generated Azure machine provider specifications, including disk name, size, LUN, caching, managed-disk type, and disk-encryption-set references where supported.
- Generated MachineConfig and Ignition configuration for control-plane disk setup.
- Machine Config Operator rollout and reconciliation of disk configuration.
- Node-level partitioning, filesystem creation, mount or swap activation, and persistence.
- etcd startup, health, data placement, and normal operation after disk setup.
- Pre-merge, e2e, CI, and upgrade jobs associated with the feature.

## 4. Features to Be Tested

### 4.1 Configuration and validation

- Valid etcd disk configuration is accepted when the feature gates and required Azure fields are present.
- The configured disk is matched to the requested platformDiskID and disk setup type.
- Invalid or incomplete managed-disk configuration fails deterministically with an actionable error.
- Invalid configuration must not cause an installer panic or leave a partially created cluster.
- Unsupported Azure Stack Hub data-disk configuration is rejected explicitly rather than attempted silently.
- Unsupported data-disk security-encryption configuration is rejected explicitly until support is delivered by the separately tracked work.

### 4.2 Azure resource delivery

- Every control-plane machine receives the additional disk.
- The attached disk has the configured size, LUN, caching mode, name suffix, and managed-disk settings.
- Disk attachment is consistent across all control-plane replicas.
- A configuration with additional user-defined compute disks does not prevent the etcd disk from being provisioned.
- Disk resources follow the expected lifecycle and deletion policy when the cluster is removed, subject to the product design.

### 4.3 MachineConfig and node behavior

- The expected disk-setup MachineConfig is generated only when the relevant feature gate is enabled.
- The MachineConfig targets the correct machine role.
- The disk is partitioned and labeled as expected.
- The etcd disk is formatted as XFS and mounted at the intended etcd path.
- The mount unit has the correct dependency on filesystem checks and is enabled for boot.
- The disk remains correctly mounted after node reboot and MachineConfig reconciliation.
- No existing OS, container-storage, or unrelated mount is overwritten.

### 4.4 etcd behavior

- All etcd members become healthy after installation.
- etcd data is stored on the additional disk, not on the operating-system disk.
- etcd remains healthy during a controlled reboot of each control-plane node.
- Normal etcd read/write operations continue after disk setup and after reconciliation.
- A full control-plane rollout does not cause unrecoverable etcd quorum loss.

### 4.5 Upgrade and performance

- The upgrade job provisions and uses the etcd disk configuration.
- The cluster upgrades from the supported previous version to the target version successfully.
- The disk remains attached, mounted, and used by etcd throughout the upgrade.
- The job records upgrade duration and etcd latency metrics for later comparison.
- Results document the comparison between the etcd-disk configuration and the control or baseline configuration.
- CI execution is repeatable and does not require a flaky-test exemption.

### 4.6 Shared multi-disk regression coverage

The shared disk setup path must retain coverage for:

- etcd disks on control-plane nodes;
- user-defined data disks on compute nodes;
- swap disks where the platform and feature support them;
- multiple disk definitions with distinct names and LUNs; and
- disk labels and mount paths containing characters that require sanitization.

## 5. Features Not to Be Tested
Successful use of data-disk securityProfile.securityEncryptionType where the current scope explicitly rejects the configuration; full support is tracked separately under SPLAT-2897.
- Successful data-disk provisioning on Azure Stack Hub when the expected behavior is explicit rejection.
- Disk behavior on non-Azure platforms, except for regression checks proving that unrelated platforms are not changed.
- Absolute performance or latency improvement targets, until product and QE owners define measurable thresholds.
- Azure infrastructure behavior outside the installer, Machine API, Machine Config Operator, and etcd integration boundaries.
- Customer-specific Azure subscriptions, quotas, or networking topologies.

## 6. Test Strategy and Approach

### 6.1 Pre-merge and unit-level testing

- Add installer tests for valid and invalid Azure disk configurations.
- Verify generated machine manifests contain the expected data-disk fields.
- Verify generated MachineConfig and Ignition content for etcd, user-defined, and swap setup types.
- Verify no-panic behavior for missing or incompatible nested configuration.
- Verify feature-gate-off behavior preserves the baseline installation path.

### 6.2 Machine Config Operator e2e testing

Implement at least five e2e scenarios as required by SPLAT-2510:

- **MCO-E2E-01 — etcd disk:** attach, partition, format, mount, and verify etcd data placement.
- **MCO-E2E-02 — user-defined disk:** attach and verify the requested filesystem and mount path on compute nodes.
- **MCO-E2E-03 — swap disk:** verify the swap partition and activation behavior where supported.
- **MCO-E2E-04 — reboot and reconciliation:** reboot an affected node and verify disk persistence, mount persistence, and MCO convergence.
- **MCO-E2E-05 — multiple disks:** configure more than one disk and verify independent LUN, label, filesystem, and mount behavior without cross-wiring.

### 6.3 Azure installation e2e testing

- Install a multi-control-plane Azure cluster with a dedicated etcd disk.
- Validate Azure resources and node state on every control-plane replica.
- Validate etcd health and data placement before and after a controlled node reboot.
- Install with both control-plane etcd disks and compute user-defined disks to verify coexistence.
- Execute negative configuration cases and verify early, actionable failure.

### 6.4 Upgrade testing

- Run the dedicated upgrade CI job from the agreed previous version to the target version.
- Collect upgrade completion status, elapsed time, etcd latency, and relevant node/MCO conditions.
- Repeat with a baseline configuration without a dedicated etcd disk when the same payload and environment are available.
- Preserve the measurements as CI artifacts and summarize the comparison in the test report.

### 6.5 CI validation

- Add or update the pre-merge, e2e, and CI job definitions associated with SPLAT-2374, SPLAT-2375, and SPLAT-2376.
- Ensure the jobs select an Azure environment with VM types that support the configured disk layout.
- Ensure job artifacts include install configuration, generated manifests, MachineConfig status, node disk/mount evidence, etcd health, and failure diagnostics.
- Run the jobs repeatedly enough to establish reliability before making them gating.

## 7. Test Case Definitions

### TP-01 — Feature gate disabled

**Setup:** Use an otherwise valid Azure install configuration without enabling the multi-disk setup feature.  

**Expected result:** The installer follows the baseline path and does not create disk-setup MachineConfigs unexpectedly.

### TP-02 — Dedicated etcd disk installation

**Setup:** Enable the supported feature gates and configure one additional disk for each control-plane replica with diskSetup.type: etcd.  

**Expected result:** Installation succeeds; every control-plane node has the configured disk, the disk is mounted at the intended etcd path, and etcd is healthy.

### TP-03 — Manifest-to-resource traceability
**Setup:** Generate installation manifests from a valid etcd-disk configuration.  

**Expected result:** The configured disk name, size, LUN, caching, and managed-disk settings are represented consistently in the generated machine resources and the corresponding disk setup is linked by platformDiskID.

### TP-04 — MachineConfig and Ignition correctness

**Setup:** Apply the generated MachineConfig to a test machine pool.  

**Expected result:** The disk is partitioned, labeled, formatted as XFS, and mounted using the expected systemd unit; the MachineConfig becomes applied and the node returns to a healthy state.

### TP-05 — etcd persistence and reboot

**Setup:** Run a healthy cluster with etcd on the additional disk, then reboot one control-plane node at a time.  

**Expected result:** The disk remounts automatically, etcd data remains available, and cluster health is restored without loss of quorum.

### TP-06 — Multiple disk coexistence

**Setup:** Configure an etcd disk on control-plane nodes and a user-defined disk on compute nodes, with distinct names and LUNs.  

**Expected result:** Both disk classes are provisioned and configured independently; neither disk is mounted at the other disk's path.

### TP-07 — Managed-disk validation

**Setup:** Configure managedDisk without the required storage-account type.  

**Expected result:** The configuration is rejected with a clear validation error before successful cluster provisioning; no panic and no partially usable cluster are produced.

### TP-08 — Disk Encryption Set negative case

**Setup:** Configure a Disk Encryption Set without the other required managed-disk fields.  

**Expected result:** The installer returns an actionable validation error and does not panic during manifest generation.

### TP-09 — Unsupported disk security encryption

**Setup:** Configure an unsupported data-disk security-encryption type.  

**Expected result:** The installer rejects the configuration explicitly and does not silently omit the setting or provision a disk with different security semantics.

### TP-10 — Azure Stack Hub negative case

**Setup:** Attempt the data-disk configuration on Azure Stack Hub.  

**Expected result:** The installer rejects the unsupported combination with a clear error and documents the limitation.

### TP-11 — Upgrade with dedicated etcd disk

**Setup:** Run the upgrade workflow with the dedicated etcd disk configuration.  

**Expected result:** The upgrade completes, etcd remains healthy, the disk remains attached and mounted, and performance evidence is collected.

## 8. Item Pass/Fail Criteria

An item passes only when:

- the expected result for every applicable test case is met;
- no critical or major defect remains open against the in-scope behavior;
- invalid configurations fail safely and diagnostically;
- no installer panic, silent field loss, or partially provisioned success is observed;
- all control-plane replicas use the configured etcd disk correctly;
- the cluster remains healthy through reboot, MCO reconciliation, and upgrade; and
- required CI artifacts and test results are available for review.

An item fails when any expected behavior is not met, when configuration is silently ignored, when an unsupported input causes a panic, or when the upgrade or etcd health criteria are not satisfied.

## 9. Suspension Criteria and Resumption Requirements

Suspend execution when:

- Azure credentials, quota, region capacity, or required VM types are unavailable;
- the test environment has an unrelated platform outage or a failed prerequisite operator;
- the test payload does not contain the implementation under test; or
- required observability artifacts cannot be collected.

Resume only after the blocker is documented, the environment is healthy, and a baseline installation or equivalent control has passed. Retain the original failure evidence and mark reruns separately.

## 10. Test Deliverables

- This test plan in Markdown.
- Installer unit and validation tests.
- At least five Machine Config Operator e2e tests.
- Azure installation e2e coverage for dedicated etcd storage.
- Pre-merge, e2e, CI, and upgrade job definitions and execution results.
- CI artifacts containing configuration, manifests, MachineConfig status, disk/mount evidence, etcd health, and performance measurements.
- A final test report with pass/fail status, known limitations, defect references, and environment details.

## 11. Test Tasks

1. Refine the supported API fields, feature-gate prerequisites, platform/version matrix, and performance thresholds.
2. Implement pre-merge validation and manifest-generation coverage.
3. Implement the five required Machine Config Operator e2e scenarios.
4. Implement Azure installation e2e coverage for dedicated etcd storage.
5. Implement the upgrade workflow and performance artifact collection.
6. Add and validate CI job configuration.
7. Execute repeated runs and classify failures as product defects, test defects, or infrastructure failures.
8. Review results with Installer, Machine Config Operator, CI, and QE owners.
9. Publish the final report and update the linked Jira stories and defects.

## 12. Environmental Needs

- Azure public-cloud test subscription with permissions to create and inspect the cluster's VM and disk resources.
- Supported Azure VM types with capacity for the configured additional disks.
- Multi-control-plane cluster topology; three replicas are recommended for the primary e2e scenario.
- Test payloads containing the relevant installer, Machine API, Machine Config Operator, and etcd changes.
- Access to node-level disk, mount, filesystem, systemd, MCO, and etcd health evidence.
- Separate baseline environment or comparable run for upgrade and performance comparison.
- Azure Stack Hub environment only if needed to verify the negative/rejection case.

The exact OCP release versions, architectures, regions, VM types, and CI environments are **TBD** and must be recorded when the child stories are refined.

## 13. Responsibilities

- **Installer engineering:** installer API, validation, manifest generation, and unit tests.
- **Machine API engineering:** Azure machine resource delivery and disk attachment behavior.
- **Machine Config Operator engineering:** MachineConfig reconciliation and node-level disk setup behavior.
- **QE:** test implementation, execution, defect validation, and final test report.
- **CI owners:** pre-merge, e2e, upgrade job configuration, reliability, and artifact retention.
- **Documentation/product owners:** supported configuration, limitations, troubleshooting, and performance-result review.

## 14. Staffing and Training Needs

- Engineers familiar with Azure VM and managed-disk resources.
- QE engineers familiar with OpenShift installation, MachineConfig rollout, node storage inspection, and etcd health validation.
- CI owners familiar with Azure job configuration and artifact collection.
- No additional training requirement identified; **N/A** pending team review.

## 15. Schedule

- Test-plan review: **TBD**
- Pre-merge implementation: tracked by SPLAT-2374; date **TBD**
- E2E automation: tracked by SPLAT-2375; date **TBD**
- CI implementation: tracked by SPLAT-2376; date **TBD**
- Upgrade implementation: tracked by SPLAT-2419; date **TBD**
- Machine Config Operator e2e implementation: tracked by SPLAT-2510; date **TBD**
- Final sign-off: **TBD**, after exit criteria are met

## 16. Risks and Contingencies

- **Feature maturity:** SPLAT-2352 is still in Planning and has only been reported as partially included in a nightly. Re-baseline the plan against the release payload used for each test run.
- **Configuration drift:** Installer and Machine API field support may differ across payloads. Archive generated manifests and record the exact payload for every run.- **Disk ordering and LUN differences:** Azure VM types may expose different device paths. Validate by stable disk identity, LUN, and partition label rather than assuming a device name.
- **MCO rollout impact:** Disk setup changes can temporarily make nodes unavailable. Test one control-plane node at a time and preserve quorum-health evidence.
- **Encryption scope:** Unsupported security-encryption inputs must remain negative tests until the dedicated support work is complete.
- **Azure capacity and permissions:** If the environment cannot provide the required VM or disk resources, mark the run blocked rather than treating it as a product pass.
- **Performance interpretation:** Do not claim an improvement until comparable baseline and etcd latency data are collected under equivalent conditions.

## 17. Approvals

- Installer engineering: **TBD**
- Machine API engineering: **TBD**
- Machine Config Operator engineering: **TBD**
- QE: **TBD**
- CI/release owner: **TBD**
- Product/documentation owner: **TBD**

## 18. Traceability Summary

- Configuration and pre-merge coverage: SPLAT-2374
- E2E automation: SPLAT-2375
- CI implementation: SPLAT-2376
- Upgrade and performance coverage: SPLAT-2419
- Machine Config Operator e2e coverage: SPLAT-2510
- Known Azure data-disk defect themes: OCPBUGS-59520, OCPBUGS-59521, OCPBUGS-59522, OCPBUGS-59743, and OCPBUGS-61892