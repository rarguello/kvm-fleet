# kvm_hypervisor

Bootstraps a bare-metal (or nested) host as a KVM hypervisor: installs virtualization packages, enables libvirt daemons, runs `virt-host-validate`, ensures the image storage pool, and optionally configures nested virtualization.

Idempotent and cheap to re-run — day-to-day VM changes should skip it entirely with `--tags vms`.

## Requirements

EL9/EL10 family host (RHEL, Rocky Linux, AlmaLinux, CentOS Stream) reachable over SSH with sudo.

## Role Variables

```yaml
# Set false to skip this role (hypervisor managed elsewhere, or a faster
# --tags vms/networks/images run).
kvm_hypervisor_bootstrap: true

kvm_image_dir: /var/lib/libvirt/images
kvm_pool_name: images

# Set false on image-mode/bootc hosts, where packages are already baked
# into the image and dnf/package management doesn't apply.
kvm_hypervisor_manage_packages: true

kvm_hypervisor_packages:
  - qemu-kvm
  - libvirt
  - libvirt-client
  - virt-install
  - python3-libvirt
  - python3-lxml

kvm_hypervisor_install_cockpit: false
kvm_hypervisor_cockpit_packages: [cockpit, cockpit-machines]

# libguestfs (virt-cat, guestfish, virt-df, ...) for offline disk inspection.
kvm_hypervisor_install_libguestfs: false
kvm_hypervisor_libguestfs_packages: [libguestfs-tools-c]

# EL9/EL10 ship modular libvirt daemons by default. Set false for EL8 and
# earlier (monolithic libvirtd).
kvm_hypervisor_modular_daemons: true
kvm_hypervisor_monolithic_services: [libvirtd, libvirtd.socket]
kvm_hypervisor_modular_services: [...]  # see defaults/main.yml

# Fails the play if virt-host-validate reports a hard FAIL.
kvm_hypervisor_require_preflight_pass: true

# Enables nested=1 for kvm_intel/kvm_amd. Off by default. If the module is
# already in use (VMs running), the setting is written to modprobe.d but
# only takes effect on the hypervisor's next reboot.
kvm_hypervisor_nested_virt: false
```

## Example

```yaml
kvm_hypervisor_bootstrap: true
kvm_hypervisor_nested_virt: true
```
