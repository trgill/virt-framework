# virt-framework

VM-based end-to-end testing framework for [Snapshot Manager](https://github.com/snapshotmanager/snapm).

## Overview

virt-framework provides automated end-to-end testing for snapm using libvirt/qemu virtual machines. It provisions test VMs, installs snapm from a git repository, and executes a comprehensive test suite that validates snapshot creation, boot-to-snapshot, and rollback functionality.

## Features

- **Multi-OS Support**: Fedora 42/43, CentOS Stream 9/10
- **Storage Layouts**: LVM and LVM-thin configurations
- **Firmware Modes**: BIOS and UEFI boot support
- **Automated Testing**: Complete lifecycle testing from provisioning to validation
- **CI/CD Integration**: Designed for GitHub Actions workflows

## System Requirements

The following packages must be installed on the host system:

- `libvirt-daemon-system` / `libvirt-daemon`
- `libvirt-clients`
- `qemu-kvm`
- `qemu-utils`
- `virtinst` (virt-install)
- `libosinfo-bin`
- `seabios` (for BIOS boot)
- `ovmf` (for UEFI boot)
- `wget`
- `expect`
- `python3` (>= 3.9)
- `pip`

## Installation

Install from the local repository:

```bash
pip install /path/to/virt-framework
```

Or install in development mode:

```bash
pip install -e /path/to/virt-framework
```

## Usage

Basic usage:

```bash
virt-framework fedora42
```

With options:

```bash
# Test with UEFI firmware and LVM-thin storage
virt-framework --uefi --storage lvm-thin fedora43

# Test a specific snapm repository and branch
virt-framework --repo myuser/snapm --ref-name feature-branch centos-stream9

# Keep VM running after test completes
virt-framework --keep fedora42
```

### Command-Line Options

- `BASE_OS`: Operating system to install (required)
  - Supported: `fedora42`, `fedora43`, `centos-stream9`, `centos-stream10`
- `--uefi` / `--bios`: Boot firmware mode (default: BIOS)
- `--storage`, `-s`: Storage layout (default: lvm)
  - Choices: `lvm`, `lvm-thin`
- `--repo`: GitHub repository to test (default: snapshotmanager/snapm)
- `--ref-name`: Git ref/branch to test (default: main)
- `--keep`: Keep VM running after test completion
- `--allow-root`: Enable root account access on test VM
- `--json`: Output results in JSON format

### JSON Output

Use the `--json` flag to output structured test results:

```bash
virt-framework --json fedora42
```

Example JSON output:

```json
{
  "vm_name": "snapm-test-fedora42-12345-abc12",
  "base_os": "fedora42",
  "storage": "lvm",
  "uefi": false,
  "repo": "snapshotmanager/snapm",
  "ref_name": "main",
  "start_time": "2026-06-25T10:30:00Z",
  "end_time": "2026-06-25T10:45:30Z",
  "duration_seconds": 930.5,
  "success": true,
  "vm_ip": "192.168.122.100",
  "test_results": {
    "test_10_verify_system_ready": true,
    "test_20_create_baseline": true,
    "test_30_create_snapset": true,
    "test_40_apply_system_changes": true,
    "test_50_verify_changes_applied": true,
    "test_60_reboot_to_snapshot": true,
    "test_70_verify_snapshot_state": true,
    "test_80_reboot_to_updated_system": true,
    "test_90_verify_back_in_updated_system": true,
    "test_95_initiate_revert": true,
    "test_99_verify_final_rollback_success": true
  }
}
```

### Test Workflow

The framework executes the following test steps:

1. **System Ready**: Verify snapm installation and LVM configuration
2. **Create Baseline**: Record initial system state and create test files
3. **Create Snapset**: Create bootable snapshot before system changes
4. **Apply Changes**: Install package updates and modify test files
5. **Verify Changes**: Confirm system mutations took effect
6. **Reboot to Snapshot**: Boot into snapshot state
7. **Verify Snapshot State**: Confirm system reverted to baseline
8. **Reboot to Updated System**: Return to updated state
9. **Verify Updated System**: Confirm back in updated state
10. **Initiate Revert**: Start rollback process
11. **Verify Rollback**: Confirm complete rollback to snapshot state

## Integration with snapm CI/CD

Example GitHub Actions workflow integration:

```yaml
- name: Install system dependencies
  run: |
    sudo apt-get update
    sudo apt-get install -y libvirt-daemon-system qemu-kvm virtinst

- name: Install virt-framework
  run: |
    pip install /path/to/virt-framework

- name: Run end-to-end tests
  run: |
    sudo virt-framework \
      --uefi \
      --storage lvm \
      --repo ${{ github.repository }} \
      --ref-name ${{ github.ref_name }} \
      fedora42
```

## Architecture

The framework consists of three main modules:

- **testvm.py**: VM provisioning and infrastructure management
  - Creates VMs using virt-install with kickstart automation
  - Manages SSH connectivity and command execution
  - Handles LVM setup verification
  - Installs snapm and boom-boot from git

- **strategy.py**: Test execution framework
  - Defines abstract test strategy pattern
  - OS-specific strategy implementations (Fedora, CentOS, RHEL)
  - Test step orchestration and validation

- **util.py**: Helper utilities
  - Logging functions for stdout/stderr

## Development

To contribute or modify the framework:

1. Clone the repository
2. Install in development mode: `pip install -e .`
3. Make changes to the source code
4. Test with: `virt-framework --help`

## License

Apache-2.0

## Authors

- Bryn M. Reeves <bmr@redhat.com>
- Todd Gill <tgill@redhat.com>

## History

This framework was originally developed as part of the snapm project and extracted into a standalone repository to enable reuse across snapm-ecosystem projects.
