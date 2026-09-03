# Additional Disk Support for AWS — Dedicated etcd and Other Block Devices (Day-0)

**Test Plan Identifier:** SPLAT-2841-TP-001

**Plan Status:** Draft  

**Feature:** [SPLAT-2841](https://redhat.atlassian.net/browse/SPLAT-2841) — [Tech Preview][AWS] Additional Disk Support — Dedicated etcd and Other Block Devices (Day-0)  

**Priority:** Major  

**Feature Status:** New; the epic and all child stories are unresolved. Implementation, target release, supported EC2 instance-family matrix, exact EBS parameters, performance thresholds, and CI job names are TBD.  

**Primary Components:** OpenShift Installer, Cluster API Provider AWS (CAPA), Machine API, Machine Config Operator, etcd, AWS EC2/EBS  

## 1. Introduction

This test plan defines the verification strategy for adding AWS EBS volumes as additional block devices during OpenShift cluster installation (day-0). The feature uses the platform-agnostic `controlPlane.diskSetup` and `compute.diskSetup` install-config fields with an AWS-specific binding that maps `PlatformDiskID` to CAPA `NonRootVolumes` and corresponding EBS volumes.

Three disk-setup types are in scope: `etcd` (dedicated etcd data directory on control-plane nodes), `swap` (swap partition activation), and `user-defined` (arbitrary mount points on compute nodes for workloads such as `/var/lib/containers`, `/var/log/containers`, or `/var/lib/kubelet`). The feature targets IPI, BYO VPC, and UPI installation topologies.

Because the epic is at Tech Preview maturity and all child stories remain in "New" status, this plan distinguishes between:

- **Research and design verification** — confirming that documented findings, design decisions, and enhancement proposals satisfy acceptance criteria (SPLAT-2842, SPLAT-2843, SPLAT-2848).
- **Executable product tests** — functional, integration, performance, and CI tests that require a working implementation (SPLAT-2844, SPLAT-2845, SPLAT-2846, SPLAT-2849, SPLAT-2855, SPLAT-2850, SPLAT-2851).

Test scenarios and acceptance criteria will be refined as child stories are implemented. Conservative `TBD` markers are used wherever implementation details, target versions, or thresholds have not yet been decided.

## 2. References

### 2.1 Jira

- [SPLAT-2841](https://redhat.atlassian.net/browse/SPLAT-2841) — Feature epic
- [SPLAT-2842](https://redhat.atlassian.net/browse/SPLAT-2842) — CAPA NonRootVolumes research
- [SPLAT-2843](https://redhat.atlassian.net/browse/SPLAT-2843) — Platform-agnostic enhancement and AWS binding
- [SPLAT-2844](https://redhat.atlassian.net/browse/SPLAT-2844) — AWS installer IPI implementation
- [SPLAT-2845](https://redhat.atlassian.net/browse/SPLAT-2845) — BYO VPC validation
- [SPLAT-2846](https://redhat.atlassian.net/browse/SPLAT-2846) — UPI CloudFormation template updates
- [SPLAT-2848](https://redhat.atlassian.net/browse/SPLAT-2848) — HyperShift research
- [SPLAT-2849](https://redhat.atlassian.net/browse/SPLAT-2849) — AWS swap via diskSetup
- [SPLAT-2855](https://redhat.atlassian.net/browse/SPLAT-2855) — AWS user-defined disks via compute.diskSetup
- [SPLAT-2850](https://redhat.atlassian.net/browse/SPLAT-2850) — E2E day-0/day-2 testing
- [SPLAT-2851](https://redhat.atlassian.net/browse/SPLAT-2851) — Performance benchmarking
- [OCPSTRAT-2546](https://redhat.atlassian.net/browse/OCPSTRAT-2546) — Parent initiative
- [SPLAT-2133](https://redhat.atlassian.net/browse/SPLAT-2133) — Azure Additional Disk Tech Preview (related)
- [SPLAT-2352](https://redhat.atlassian.net/browse/SPLAT-2352) — Azure Additional Disk GA planning (related)

### 2.2 Enhancement proposals

- [openshift/enhancements#1805](https://github.com/openshift/enhancements/pull/1805) — Installer Multiple Disk Setup
- [openshift/enhancements#1779](https://github.com/openshift/enhancements/pull/1779) — Azure Multiple Data Disks
- [openshift/enhancements#2017](https://github.com/openshift/enhancements/pull/2017) — Azure Data Disk IOPS/Throughput

### 2.3 Precedent

- [SPLAT-2352 test plan](https://github.com/openshift-splat-team/test-plans/blob/main/features/SPLAT-2352/plan.md) — Azure additional etcd disk test plan

## 3. Test Items

The following items are under test:

- AWS install-config fields for additional EBS volumes and disk setup (`controlPlane.diskSetup`, `compute.diskSetup`).
- The `MultiDiskSetup` feature-gate path in the installer.
- `PlatformDiskID` mapping from the platform-agnostic disk-setup schema to CAPA `AWSMachineSpec.NonRootVolumes`.
- Generated CAPA machine specifications, including EBS volume type (e.g., gp3, io2), size, IOPS, throughput, encryption, and KMS key references.
- Generated MachineConfig and Ignition configuration for control-plane and compute disk setup.
- Machine Config Operator rollout and reconciliation of disk configuration.
- Node-level disk discovery (stable labels/udev, not hardcoded device paths), partitioning, filesystem creation, mount or swap activation, and persistence.
- etcd startup, health, data placement, and normal operation on a dedicated EBS volume.
- Day-2 MachineSet scaling via MAPI/CAPI-portable abstraction and CPMS operations.
- BYO VPC IAM permissions, security-group considerations, AZ alignment, and private subnet/proxy configurations.
- UPI CloudFormation template optional control-plane EBS volume parameters.
- Research and design deliverables for CAPA `NonRootVolumes` coverage, NVMe naming, and HyperShift applicability.
- Performance benchmarking comparing shared-root-disk versus dedicated-etcd-disk configurations.
- E2E and CI jobs associated with the feature (names and identifiers TBD).

## 4. Features to Be Tested

### 4.1 Configuration and validation

- Valid etcd disk configuration is accepted when the `MultiDiskSetup` feature gate and required AWS fields are present.
- The configured disk is matched to the requested `PlatformDiskID` and disk-setup type (`etcd`, `swap`, or user-defined).
- The `PlatformDiskID` mapping correctly translates to CAPA `NonRootVolumes` entries.
- EBS volume parameters (volume type, size, IOPS, throughput, encryption, KMS key) are propagated from install-config through to CAPA `AWSMachineSpec.NonRootVolumes` and to actual EBS volumes.
- Invalid or incomplete disk configuration fails deterministically with an actionable error.
- Invalid configuration must not cause an installer panic or leave a partially created cluster.
- The feature gate disabled path preserves the baseline installation behavior with no unexpected disk-setup MachineConfigs.

### 4.2 AWS resource delivery — IPI

- Every control-plane machine receives the additional EBS volume(s) as specified.
- The attached EBS volume has the configured volume type, size, IOPS, throughput, and encryption settings.
- Volume attachment is consistent across all control-plane replicas.
- A configuration with additional user-defined compute disks does not prevent the etcd disk from being provisioned.
- EBS volumes follow the expected lifecycle and deletion policy when the cluster is removed.
- Disk discovery on the node uses stable labels/udev rules rather than hardcoded device paths, accounting for AWS NVMe versus non-NVMe device naming differences.

### 4.3 AWS resource delivery — BYO VPC

- IPI installation succeeds in a BYO VPC environment with the additional disk configuration.
- Required IAM permissions (`ec2:CreateVolume`, `ec2:AttachVolume`) are documented and validated.
- Security-group considerations for EBS operations are identified and documented.
- AZ alignment between subnets and EBS volumes is correct.
- Private subnet and proxy configurations do not prevent EBS volume provisioning or attachment.

### 4.4 AWS resource delivery — UPI

- CloudFormation templates include optional parameters for control-plane EBS volumes.
- The MachineConfig pattern for disk formatting and mounting is consistent with IPI.
- UPI documentation is updated to reflect the optional EBS volume parameters.

### 4.5 MachineConfig and node behavior

- The expected disk-setup MachineConfig is generated only when the `MultiDiskSetup` feature gate is enabled.
- The MachineConfig targets the correct machine role (master or worker).
- The disk is discovered by stable identity (udev labels), not by hardcoded `/dev/` path.
- The etcd disk is formatted as XFS and mounted at the intended etcd data directory path.
- The mount unit has the correct dependency on filesystem checks and is enabled for boot.
- The disk remains correctly mounted after node reboot and MachineConfig reconciliation.
- No existing OS, container-storage, or unrelated mount is overwritten.

### 4.6 etcd behavior

- All etcd members become healthy after installation with dedicated disk.
- etcd data is stored on the additional EBS volume, not on the root volume.
- etcd remains healthy during a controlled reboot of each control-plane node.
- Normal etcd read/write operations continue after disk setup and after MCO reconciliation.
- A full control-plane rollout does not cause unrecoverable etcd quorum loss.

### 4.7 Swap disk behavior

- A swap partition is provisioned and activated when `diskSetup[].type: swap` is configured.
- Kubelet swap configuration is applied correctly.
- The node remains stable under load with swap active.

### 4.8 User-defined disk behavior

- User-defined disks on compute nodes are provisioned and mounted at the requested mount points (e.g., `/var/lib/containers`, `/var/log/containers`, `/var/lib/kubelet`).
- Mount points are validated against allowed and restricted paths (open question: exact allowed mount points and control-plane versus compute restrictions are TBD per SPLAT-2855).
- The node remains stable under load with user-defined mounts active.

### 4.9 Day-2 operations

- Day-2 MachineSet scaling via MAPI/CAPI-portable abstraction provisions new nodes with the expected additional disk configuration.
- CPMS operations with dedicated-etcd configuration complete successfully.
- The design is CAPI-migration aware; day-2 abstractions do not assume MAPI-only internals.

### 4.10 EC2 instance-family and EBS type coverage

- The feature is validated on both NVMe and non-NVMe EC2 instance families (exact family matrix TBD; to be defined in SPLAT-2842).
- At minimum, gp3 and io2 EBS volume types are validated.
- Encryption configurations (default encryption, customer-managed KMS keys) are validated.

### 4.11 Performance

- Controlled benchmarks compare shared root disk versus dedicated etcd disk across multiple EBS configurations (gp3 default, gp3 tuned IOPS, io2).
- Metrics collected include etcd WAL fsync p50/p99/p999, commit latency, backend commit duration, I/O throughput/IOPS, leader-election stability, API latency, and boot/provisioning overhead.
- Scenarios include idle, sustained API load, and upgrade/node drain.

## 5. Features Not to Be Tested

- Disk behavior on non-AWS platforms, except for regression checks proving that unrelated platforms are not affected.
- HyperShift disk-setup integration if the research (SPLAT-2848) determines that `diskSetup` does not apply to HyperShift's PV-backed etcd storage model and no alternative interface has been designed. See section 16 for the decision-gate risk.
- Absolute performance or latency improvement targets until product and QE owners define measurable thresholds (SPLAT-2851 will propose thresholds as an output, not as a precondition).
- AWS infrastructure behavior outside the installer, CAPA, Machine API, Machine Config Operator, and etcd integration boundaries.
- Customer-specific AWS accounts, quotas, or networking topologies.
- Day-2 disk resize, reconfiguration, or hot-attach scenarios unless explicitly added to a child story scope.
- GA-readiness criteria; this plan covers Tech Preview scope only.

## 6. Test Strategy and Approach

### 6.1 Research and design verification (Phase 1)

These verifications confirm that documented findings and design deliverables satisfy acceptance criteria. They are review-based, not automated-test-based.

- **SPLAT-2842 — CAPA research:** Verify that the deliverable documents CAPA `NonRootVolumes` coverage (volume type, size, IOPS, throughput, encryption, KMS), NVMe versus non-NVMe naming across supported EC2 families, and `PlatformDiskID` mapping. Verify that a tested EC2 family list is included.
- **SPLAT-2843 — Enhancement:** Verify that a platform-agnostic enhancement for `diskSetup` and the AWS binding is documented, referencing EP #1805, EP #1779, and EP #2017. Verify that the document states whether EP #1805 is revived or a unified EP is created.
- **SPLAT-2848 — HyperShift research:** Verify that the deliverable assesses StorageClass customization for volume type/IOPS/throughput, resize/reconfiguration, and whether `diskSetup` applies or another interface is needed. This is a **decision gate**: if the research concludes that `diskSetup` does not apply to HyperShift, no HyperShift functional tests will be authored under this epic. See section 16 for risk treatment.

### 6.2 Pre-merge and unit-level testing

- Add installer unit tests for valid and invalid AWS disk configurations.
- Verify generated CAPA machine manifests contain the expected `NonRootVolumes` entries with correct EBS parameters.
- Verify generated MachineConfig and Ignition content for etcd, swap, and user-defined setup types.
- Verify `PlatformDiskID` mapping consistency between install-config, CAPA spec, and node-level udev identity.
- Verify no-panic behavior for missing or incompatible nested configuration.
- Verify feature-gate-off behavior preserves the baseline installation path.
- Verify that NVMe versus non-NVMe device naming differences are handled by stable-label discovery, not hardcoded paths.

### 6.3 AWS installation E2E testing — IPI

- Install a multi-control-plane AWS cluster with a dedicated etcd disk (gp3, then io2).
- Validate EBS volumes and node state on every control-plane replica.
- Validate etcd health and data placement before and after a controlled node reboot.
- Install with both control-plane etcd disks and compute user-defined disks to verify coexistence.
- Execute negative configuration cases and verify early, actionable failure.

### 6.4 AWS installation E2E testing — BYO VPC

- Install in at least one BYO VPC configuration with additional disks.
- Validate that required IAM permissions are documented and that the install fails with an actionable error when permissions are missing.
- Validate AZ alignment, private subnet, and proxy compatibility.

### 6.5 AWS installation E2E testing — UPI

- Deploy using updated CloudFormation templates with optional control-plane EBS volume parameters.
- Validate that MachineConfig disk-setup behavior is consistent with IPI.
- Validate that documentation updates are accurate.

### 6.6 Swap and user-defined disk testing

- Provision swap disks and verify activation, kubelet configuration, and stability under load.
- Provision user-defined compute disks at target mount points and verify mounting, filesystem, and stability.
- These are optional tests, depending on PM priority (SPLAT-2849) and resolution of open questions (SPLAT-2855).

### 6.7 Day-2 and scaling testing

- Scale up a MachineSet and verify that new nodes receive the expected additional disk configuration.
- Run CPMS operations with dedicated-etcd configuration.
- Verify CAPI migration awareness in day-2 abstractions.
- Validate on both NVMe and non-NVMe EC2 instance families.

### 6.8 Performance benchmarking

- Establish a baseline with shared root disk configuration.
- Run dedicated-etcd variants: gp3 default, gp3 tuned IOPS, io2.
- Collect the metrics defined in section 4.11 under idle, sustained API load, and upgrade/node drain scenarios.
- Document performance numbers, cost implications, stability implications, and a recommendation.

### 6.9 CI validation

- Add or update pre-merge, E2E, and CI job definitions (job names and identifiers TBD).
- Ensure the jobs select AWS environments with EC2 instance types that support the configured disk layout.
- Ensure job artifacts include install configuration, generated manifests, MachineConfig status, node disk/mount evidence, etcd health, and failure diagnostics.
- Run the jobs repeatedly to establish reliability before making them gating.

## 7. Test Case Definitions

### Research and Design Verification Tests

#### RD-01 — CAPA NonRootVolumes research completeness

**Traceability:** SPLAT-2842  
**Setup:** Review the SPLAT-2842 deliverable document.  
**Expected result:** The document covers: CAPA `NonRootVolumes` field coverage (volume type, size, IOPS, throughput, encryption, KMS); NVMe versus non-NVMe naming behavior across EC2 families; `PlatformDiskID` mapping design; and a tested EC2 family list. Gaps or unsupported CAPA fields are explicitly identified.

#### RD-02 — Platform-agnostic enhancement documentation

**Traceability:** SPLAT-2843  
**Setup:** Review the SPLAT-2843 deliverable document or enhancement PR.  
**Expected result:** The document references EP #1805, EP #1779, and EP #2017. It states whether EP #1805 is revived or a unified EP is created. The AWS binding design for `PlatformDiskID` to CAPA `NonRootVolumes` is described. The `diskSetup` schema for types `etcd`, `swap`, and `user-defined` is specified.

#### RD-03 — HyperShift applicability assessment (decision gate)

**Traceability:** SPLAT-2848  
**Setup:** Review the SPLAT-2848 deliverable document.  
**Expected result:** The document assesses whether `diskSetup` applies to HyperShift (where etcd runs as management-cluster pods with PV-backed storage, typically EBS CSI). It evaluates StorageClass customization for volume type, IOPS, and throughput; resize/reconfiguration; and identifies whether a separate interface is needed. A clear recommendation is documented. If `diskSetup` does not apply, functional HyperShift test cases under this epic are not required.

### Installer Configuration and Validation Tests

#### IC-01 — Feature gate disabled

**Traceability:** SPLAT-2844  
**Setup:** Use an otherwise valid AWS install configuration without enabling the `MultiDiskSetup` feature gate.  
**Expected result:** The installer follows the baseline path and does not create disk-setup MachineConfigs unexpectedly.

#### IC-02 — Valid etcd disk configuration accepted

**Traceability:** SPLAT-2844  
**Setup:** Enable the `MultiDiskSetup` feature gate and configure one additional disk for each control-plane replica with `diskSetup[].type: etcd` and valid EBS parameters (volume type, size).  
**Expected result:** The install-config is accepted without validation errors. The generated CAPA `AWSMachineSpec.NonRootVolumes` contains an entry matching the configured disk.

#### IC-03 — PlatformDiskID mapping correctness

**Traceability:** SPLAT-2844, SPLAT-2842  
**Setup:** Generate installation manifests from a valid etcd-disk configuration.  
**Expected result:** The configured disk parameters (volume type, size, IOPS, throughput, encryption, KMS key) are represented consistently in the generated CAPA machine spec, and the corresponding disk setup is linked by `PlatformDiskID`.

#### IC-04 — Invalid disk configuration rejection

**Traceability:** SPLAT-2844  
**Setup:** Provide an install-config with missing or invalid EBS parameters (e.g., io2 without IOPS, negative volume size, invalid volume type).  
**Expected result:** The installer returns an actionable validation error before cluster provisioning begins. No installer panic or partially created cluster results.

#### IC-05 — MachineConfig and Ignition correctness

**Traceability:** SPLAT-2844  
**Setup:** Generate and apply MachineConfig from a valid etcd-disk configuration.  
**Expected result:** The disk is discovered by stable udev label (not hardcoded device path), partitioned, formatted as XFS, and mounted at the intended etcd data directory. The systemd mount unit has correct dependencies and is enabled for boot. The MachineConfig becomes applied and the node reaches a healthy state.

### IPI Installation and etcd Tests

#### IPI-01 — Dedicated etcd disk installation (gp3)

**Traceability:** SPLAT-2844, SPLAT-2850  
**Setup:** Install a multi-control-plane AWS cluster (3 replicas) with `diskSetup[].type: etcd` using a gp3 EBS volume.  
**Expected result:** Installation succeeds. Every control-plane node has the configured EBS volume attached. The volume is discovered by stable identity, formatted, and mounted at the etcd data directory. All etcd members are healthy and etcd data resides on the dedicated volume.

#### IPI-02 — Dedicated etcd disk installation (io2)

**Traceability:** SPLAT-2844, SPLAT-2850  
**Setup:** Same as IPI-01 but with io2 EBS volume type and configured IOPS/throughput.  
**Expected result:** Same as IPI-01; additionally, the EBS volume has the configured IOPS and throughput settings.

#### IPI-03 — etcd persistence and reboot

**Traceability:** SPLAT-2850  
**Setup:** Run a healthy cluster with etcd on a dedicated EBS volume. Reboot one control-plane node at a time.  
**Expected result:** The EBS volume remounts automatically via the systemd unit, etcd data remains available, and cluster health is restored without loss of quorum.

#### IPI-04 — Multiple disk coexistence

**Traceability:** SPLAT-2844, SPLAT-2855, SPLAT-2850  
**Setup:** Configure an etcd disk on control-plane nodes and a user-defined disk on compute nodes.  
**Expected result:** Both disk classes are provisioned and configured independently. Neither disk is mounted at the other disk's path. Both node roles reach healthy state.

#### IPI-05 — Encrypted EBS volume

**Traceability:** SPLAT-2850  
**Setup:** Configure a dedicated etcd disk with AWS default encryption or a customer-managed KMS key.  
**Expected result:** The EBS volume is created with the specified encryption settings. etcd functions correctly on the encrypted volume.

#### IPI-06 — NVMe instance family

**Traceability:** SPLAT-2842, SPLAT-2850  
**Setup:** Install on an EC2 instance type that uses NVMe device naming (exact family TBD from SPLAT-2842 research).  
**Expected result:** The disk is discovered by stable label/udev, not by NVMe device path. Installation succeeds and etcd operates correctly.

#### IPI-07 — Non-NVMe instance family

**Traceability:** SPLAT-2842, SPLAT-2850  
**Setup:** Install on an EC2 instance type that uses non-NVMe device naming (exact family TBD from SPLAT-2842 research).  
**Expected result:** Same as IPI-06; the stable-label discovery mechanism works regardless of device naming convention.

### BYO VPC Tests

#### BYO-01 — BYO VPC with additional disk

**Traceability:** SPLAT-2845  
**Setup:** Install in a pre-existing VPC with the dedicated etcd-disk configuration. Ensure IAM role includes `ec2:CreateVolume` and `ec2:AttachVolume`.  
**Expected result:** Installation succeeds. EBS volumes are created and attached in the correct AZs aligned with the VPC subnets. etcd operates correctly.

#### BYO-02 — BYO VPC missing IAM permissions

**Traceability:** SPLAT-2845  
**Setup:** Install in a BYO VPC where the IAM role is missing `ec2:CreateVolume` or `ec2:AttachVolume`.  
**Expected result:** The installation fails with an actionable error message identifying the missing permission. No partial cluster is left behind.

#### BYO-03 — BYO VPC private subnet with proxy

**Traceability:** SPLAT-2845  
**Setup:** Install in a BYO VPC with private subnets and an HTTP proxy configured.  
**Expected result:** EBS volume provisioning and attachment succeed through the proxy. etcd operates correctly on the dedicated volume.

### UPI Tests

#### UPI-01 — CloudFormation template with optional EBS volume

**Traceability:** SPLAT-2846  
**Setup:** Deploy a UPI cluster using the updated CloudFormation templates with the optional control-plane EBS volume parameters specified.  
**Expected result:** The CloudFormation stack creates EBS volumes and attaches them to control-plane instances. The MachineConfig disk-setup behavior is consistent with IPI (same filesystem, mount unit, udev discovery).

#### UPI-02 — CloudFormation template without optional EBS volume

**Traceability:** SPLAT-2846  
**Setup:** Deploy a UPI cluster using the updated CloudFormation templates without specifying optional EBS volume parameters.  
**Expected result:** The cluster installs normally without additional disks. No unexpected disk-setup MachineConfigs are created. Backward compatibility is preserved.

### Swap Disk Tests

#### SW-01 — Swap partition provisioning and activation

**Traceability:** SPLAT-2849  
**Setup:** Configure `diskSetup[].type: swap` on compute or control-plane nodes (scope TBD).  
**Expected result:** A swap partition is created on the additional EBS volume, formatted, and activated. `swapon -s` or equivalent confirms swap is active. Kubelet swap configuration is applied.

#### SW-02 — Swap stability under load

**Traceability:** SPLAT-2849  
**Setup:** Run a sustained workload that triggers swap usage on nodes with swap configured.  
**Expected result:** The node remains stable. No OOM kills or kubelet crashes occur that are attributable to the swap configuration. System metrics confirm swap usage.

### User-Defined Disk Tests

#### UD-01 — User-defined disk provisioning and mount

**Traceability:** SPLAT-2855  
**Setup:** Configure `compute.diskSetup` with a user-defined mount point (e.g., `/var/lib/containers`).  
**Expected result:** The EBS volume is provisioned, formatted, and mounted at the specified path on all compute nodes. The mount persists across reboot.

#### UD-02 — Multiple user-defined mount points

**Traceability:** SPLAT-2855  
**Setup:** Configure multiple user-defined disks on compute nodes with distinct mount points (e.g., `/var/lib/containers` and `/var/log/containers`).  
**Expected result:** Each disk is independently provisioned, formatted, and mounted at its respective path. No cross-wiring between disks.

#### UD-03 — User-defined disk stability under load

**Traceability:** SPLAT-2855  
**Setup:** Run a sustained container workload using the user-defined mount point.  
**Expected result:** The node remains stable. Container runtime I/O is served from the dedicated volume.

#### UD-04 — Restricted mount point handling

**Traceability:** SPLAT-2855  
**Setup:** Attempt to configure a user-defined disk at a mount point that conflicts with a system path or is otherwise restricted (exact restricted paths TBD per SPLAT-2855 open question).  
**Expected result:** The installer rejects the configuration with an actionable error, or the behavior matches the documented allowed-mount-point policy.

### Day-2 and Scaling Tests

#### D2-01 — MachineSet scale-up with additional disk

**Traceability:** SPLAT-2850  
**Setup:** Scale up a compute MachineSet on a cluster with user-defined disk configuration.  
**Expected result:** New compute nodes receive the additional EBS volume, formatted and mounted identically to the original nodes.

#### D2-02 — CPMS with dedicated etcd disk

**Traceability:** SPLAT-2850  
**Setup:** Trigger a control-plane MachineSet (CPMS) operation on a cluster with dedicated etcd disk configuration.  
**Expected result:** The replacement control-plane node receives the dedicated etcd disk. etcd data is available and the etcd member becomes healthy without quorum loss.

#### D2-03 — CAPI migration awareness

**Traceability:** SPLAT-2850  
**Setup:** Review the day-2 MachineSet scaling abstraction for CAPI portability.  
**Expected result:** The abstraction does not assume MAPI-only internals. The design is documented as CAPI-migration aware per the SPLAT-2850 acceptance criteria.

### Performance Benchmarking Tests

#### PB-01 — Baseline: shared root disk

**Traceability:** SPLAT-2851  
**Setup:** Install an AWS cluster with default configuration (no dedicated etcd disk). Run the benchmark scenarios: idle, sustained API load, upgrade/node drain.  
**Expected result:** Metrics are collected: etcd WAL fsync p50/p99/p999, commit latency, backend commit duration, I/O throughput/IOPS, leader-election stability, API latency, boot/provisioning overhead. These serve as the control baseline.

#### PB-02 — Dedicated etcd disk: gp3 default

**Traceability:** SPLAT-2851  
**Setup:** Install with a dedicated etcd disk (gp3, default IOPS/throughput). Run the same benchmark scenarios as PB-01.  
**Expected result:** Same metrics collected. Results are compared against the PB-01 baseline.

#### PB-03 — Dedicated etcd disk: gp3 tuned IOPS

**Traceability:** SPLAT-2851  
**Setup:** Install with a dedicated etcd disk (gp3, tuned IOPS/throughput values TBD). Run the same benchmark scenarios.  
**Expected result:** Same metrics collected. Results are compared against PB-01 and PB-02.

#### PB-04 — Dedicated etcd disk: io2

**Traceability:** SPLAT-2851  
**Setup:** Install with a dedicated etcd disk (io2, IOPS value TBD). Run the same benchmark scenarios.  
**Expected result:** Same metrics collected. Results are compared against PB-01, PB-02, and PB-03.

#### PB-05 — Performance report and recommendation

**Traceability:** SPLAT-2851  
**Setup:** Aggregate results from PB-01 through PB-04.  
**Expected result:** A written report documenting performance numbers, cost implications, stability implications, and a recommendation. No performance improvement claim is made until comparable baseline and dedicated-disk data are collected under equivalent conditions.

## 8. Item Pass/Fail Criteria

An item passes only when:

- the expected result for every applicable test case is met;
- no critical or major defect remains open against the in-scope behavior;
- invalid configurations fail safely and diagnostically;
- no installer panic, silent field loss, or partially provisioned success is observed;
- all control-plane replicas use the configured etcd EBS volume correctly;
- disk discovery uses stable identity (udev labels), not hardcoded device paths;
- the cluster remains healthy through reboot, MCO reconciliation, scaling, and (when applicable) upgrade;
- research and design deliverables (RD-01 through RD-03) address all acceptance criteria in their respective child stories;
- required CI artifacts and test results are available for review.

An item fails when any expected behavior is not met, when configuration is silently ignored, when an unsupported input causes a panic, or when etcd health or performance criteria are not satisfied.

## 9. Suspension Criteria and Resumption Requirements

Suspend execution when:

- AWS credentials, quota, region capacity, or required EC2 instance types are unavailable;
- the test environment has an unrelated platform outage or a failed prerequisite operator;
- the test payload does not contain the implementation under test;
- required observability artifacts (etcd metrics, disk/mount evidence) cannot be collected;
- a research or design deliverable (SPLAT-2842, SPLAT-2843, SPLAT-2848) has not been completed when downstream functional tests depend on it; or
- the HyperShift decision gate (SPLAT-2848) blocks HyperShift-specific test planning.

Resume only after the blocker is documented, the environment is healthy, and a baseline installation or equivalent control has passed. Retain the original failure evidence and mark reruns separately.

## 10. Test Deliverables

- This test plan in Markdown.
- Installer unit and validation tests for AWS disk configuration.
- AWS IPI, BYO VPC, and UPI installation E2E tests for dedicated etcd storage.
- Swap and user-defined disk functional tests (pending PM priority decisions).
- Day-2 MachineSet scaling and CPMS test results.
- Performance benchmarking report with baseline comparison.
- Pre-merge, E2E, and CI job definitions and execution results (job names TBD).
- CI artifacts containing install configuration, generated CAPA manifests, MachineConfig status, node disk/mount evidence, etcd health, and performance measurements.
- Research and design review records for SPLAT-2842, SPLAT-2843, and SPLAT-2848.
- A final test report with pass/fail status, known limitations, defect references, and environment details.

## 11. Test Tasks

1. Complete research deliverables (SPLAT-2842, SPLAT-2843, SPLAT-2848) and review against acceptance criteria (RD-01, RD-02, RD-03).
2. Refine the supported API fields, feature-gate prerequisites, EC2 instance-family matrix, EBS parameter ranges, and performance thresholds based on research outcomes.
3. Implement pre-merge validation and manifest-generation unit test coverage.
4. Implement AWS IPI installation E2E coverage for dedicated etcd storage.
5. Implement BYO VPC validation tests (at least one BYO VPC configuration).
6. Implement UPI CloudFormation template tests.
7. Implement swap disk tests (pending PM priority decision on SPLAT-2849).
8. Implement user-defined disk tests (pending resolution of SPLAT-2855 open questions).
9. Implement day-2 MachineSet scaling and CPMS tests.
10. Execute performance benchmarking and produce the comparison report.
11. Add and validate CI job configuration.
12. Execute repeated runs and classify failures as product defects, test defects, or infrastructure failures.
13. Review results with Installer, CAPA, Machine Config Operator, CI, and QE owners.
14. Publish the final report and update the linked Jira stories.

## 12. Environmental Needs

- AWS public-cloud test account with permissions to create and inspect EC2 instances, EBS volumes, VPCs, and IAM roles.
- EC2 instance types from both NVMe and non-NVMe families (exact types TBD from SPLAT-2842 research).
- Multi-control-plane cluster topology; three replicas are recommended for the primary E2E scenario.
- At least one pre-existing VPC for BYO VPC testing (with configurable IAM permissions for negative tests).
- Test payloads containing the relevant installer, CAPA, Machine API, Machine Config Operator, and etcd changes.
- Access to node-level disk, mount, filesystem, systemd, udev, MCO, and etcd health evidence.
- Separate baseline environment or comparable run for performance benchmarking comparison.
- AWS regions with sufficient capacity for the configured instance types and EBS volumes.

The exact OCP release versions, architectures, regions, EC2 instance types, and CI environments are **TBD** and must be recorded when the child stories are refined.

## 13. Responsibilities

- **Installer engineering:** installer API, validation, manifest generation, `PlatformDiskID` mapping, CAPA integration, and unit tests.
- **CAPA/Machine API engineering:** CAPA `NonRootVolumes` delivery and EBS volume attachment behavior.
- **Machine Config Operator engineering:** MachineConfig reconciliation and node-level disk setup behavior (udev discovery, formatting, mounting).
- **QE:** test implementation, execution, defect validation, performance benchmarking, and final test report.
- **CI owners:** pre-merge, E2E, CI job configuration, reliability, and artifact retention.
- **Documentation/product owners:** supported configuration, limitations, troubleshooting, UPI CloudFormation documentation, and performance-result review.

## 14. Staffing and Training Needs

- Engineers familiar with AWS EC2 instance types, EBS volume management, and NVMe device naming.
- QE engineers familiar with OpenShift installation, MachineConfig rollout, node storage inspection, udev rules, and etcd health validation.
- CI owners familiar with AWS job configuration and artifact collection.
- Engineers familiar with CAPA `AWSMachineSpec` and `NonRootVolumes` API.
- No additional training requirement identified; **N/A** pending team review.

## 15. Schedule

- CAPA research completion (SPLAT-2842): **TBD**
- Enhancement documentation (SPLAT-2843): **TBD**
- HyperShift research and decision gate (SPLAT-2848): **TBD**
- IPI implementation and pre-merge tests (SPLAT-2844): **TBD**
- BYO VPC validation (SPLAT-2845): **TBD**
- UPI template updates (SPLAT-2846): **TBD**
- Swap disk implementation (SPLAT-2849): **TBD** (optional, pending PM priority)
- User-defined disk implementation (SPLAT-2855): **TBD**
- E2E test implementation (SPLAT-2850): **TBD**
- Performance benchmarking (SPLAT-2851): **TBD**
- Test-plan review: **TBD**
- Final sign-off: **TBD**, after exit criteria are met

## 16. Risks and Contingencies

- **Feature maturity:** SPLAT-2841 and all child stories are in "New" status. No implementation exists yet. Re-baseline the plan against the actual implementation and release payload used for each test run.
- **Research dependencies:** Functional tests (IC-*, IPI-*, BYO-*, UPI-*) depend on research deliverables (SPLAT-2842, SPLAT-2843). Do not author or execute functional tests until the research phase confirms the API surface, EC2 family matrix, and `PlatformDiskID` mapping design.
- **HyperShift decision gate:** SPLAT-2848 is a research story that will determine whether `diskSetup` applies to HyperShift. If the research concludes that `diskSetup` does not apply (because HyperShift etcd uses PV-backed storage on the management cluster), no HyperShift functional tests will be authored under this epic. If an alternative interface is recommended, new test cases must be added in a plan revision.
- **NVMe device naming non-determinism:** AWS API device names (e.g., `/dev/sdf`) differ from OS-visible NVMe names (e.g., `/dev/nvme1n1`). Disk discovery must use stable labels/udev rather than hardcoded paths. Validate by stable identity on both NVMe and non-NVMe families.
- **EC2 family matrix uncertainty:** The exact set of supported EC2 instance families is not yet defined. SPLAT-2842 will produce this list. Until then, test coverage is limited to the families available in the CI environment.
- **Swap and user-defined disk priority:** SPLAT-2849 (swap) is optional depending on PM priority. SPLAT-2855 (user-defined) has open questions about allowed mount points and control-plane versus compute restrictions. These test cases may be deferred or modified based on PM and engineering decisions.
- **CAPI migration awareness:** The design must be CAPI-migration aware. Day-2 test abstractions must not assume MAPI-only internals. Verify portability during SPLAT-2850 implementation.
- **Configuration drift:** Installer and CAPA field support may differ across payloads. Archive generated manifests and record the exact payload for every run.
- **MCO rollout impact:** Disk setup changes can temporarily make nodes unavailable. Test one control-plane node at a time and preserve quorum-health evidence.
- **AWS capacity and permissions:** If the environment cannot provide the required EC2 instances or EBS volumes, mark the run blocked rather than treating it as a product pass.
- **Performance interpretation:** Do not claim an improvement until comparable baseline and dedicated-disk latency data are collected under equivalent conditions. Cost implications must be documented alongside performance numbers.
- **Enhancement proposal status:** It is not yet determined whether EP #1805 will be revived or a unified EP will be created (SPLAT-2843). Test plan updates may be required based on the enhancement outcome.

## 17. Approvals

- Installer engineering: **TBD**
- CAPA/Machine API engineering: **TBD**
- Machine Config Operator engineering: **TBD**
- QE: **TBD**
- CI/release owner: **TBD**
- Product/documentation owner: **TBD**

## 18. Traceability Summary

| Child Story | Description | Test Case IDs | Category |
|---|---|---|---|
| SPLAT-2842 | CAPA NonRootVolumes research | RD-01, IC-03, IPI-06, IPI-07 | Research / Design Verification |
| SPLAT-2843 | Platform-agnostic enhancement and AWS binding | RD-02 | Research / Design Verification |
| SPLAT-2844 | AWS installer IPI implementation | IC-01, IC-02, IC-03, IC-04, IC-05, IPI-01, IPI-02, IPI-04 | Executable Product Test |
| SPLAT-2845 | BYO VPC validation | BYO-01, BYO-02, BYO-03 | Executable Product Test |
| SPLAT-2846 | UPI CloudFormation template updates | UPI-01, UPI-02 | Executable Product Test |
| SPLAT-2848 | HyperShift research (decision gate) | RD-03 | Research / Design Verification |
| SPLAT-2849 | AWS swap via diskSetup (optional) | SW-01, SW-02 | Executable Product Test |
| SPLAT-2855 | AWS user-defined disks | UD-01, UD-02, UD-03, UD-04, IPI-04 | Executable Product Test |
| SPLAT-2850 | E2E day-0/day-2 testing | IPI-01 through IPI-07, D2-01, D2-02, D2-03 | Executable Product Test |
| SPLAT-2851 | Performance benchmarking | PB-01, PB-02, PB-03, PB-04, PB-05 | Executable Product Test |
