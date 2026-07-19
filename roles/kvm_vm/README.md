# kvm_vm

Declarative VM lifecycle: creates VMs with `virt-install` + cloud-init (NoCloud, static networking), destroys them cleanly, and reconciles state/autostart on ones that already exist.

## Behavior

- **`state: present`** (default) — VM exists and is running.
- **`state: stopped`** — VM exists, powered off.
- **`state: absent`** — VM is force-stopped, undefined, and its disk files removed (read from `virsh domblklist`, not guessed from current vars).
- **Already exists:** never recreated or resized. If declared `memory_mb`/`vcpus` no longer match, a warning is printed; state and autostart are still reconciled.
- **`mac`** is optional — if unset, libvirt/virt-install assigns one. If set, it must start with `52:54` (the QEMU/KVM locally-administered OUI).

## Requirements

`community.libvirt` collection. A `kvm_images` entry matching `image` and a `kvm_networks` entry matching `network` must already be declared (usually processed by those two roles earlier in the same play).

## Role Variables

```yaml
kvm_vm_default_memory_mb: 2048
kvm_vm_default_vcpus: 2
kvm_vm_default_disk_size: 20G
kvm_vm_default_device: eth0
kvm_vm_default_cpu_mode: host-passthrough

kvm_ssh_public_key_file: "~/.ssh/id_ed25519.pub"
kvm_cloud_user: cloud-user   # fallback if the image doesn't set cloud_user

kvm_vms:
  - name: web01
    state: present              # present | stopped | absent
    autostart: true
    image: rocky9                 # a kvm_images name
    network: lab                   # a kvm_networks name
    ip: 192.168.100.10
    mac: "52:54:00:aa:bb:cc"        # optional
    device: eth0                     # optional, overrides the image's default
    memory_mb: 2048
    vcpus: 2
    disks: ["20G"]                     # first entry is the OS disk (overlay);
                                         # extras are blank data disks
    swap_size_gb: 0                      # cloud-init swapfile, optional
    cloud_user: rocky                      # optional, overrides the image's default
    cloud_user_password: ""                  # optional, overrides vault default
    ssh_public_key_file: ""                    # optional, overrides the default
```

## Example

```yaml
kvm_vms:
  - name: web01
    state: present
    autostart: true
    image: rocky9
    network: lab
    ip: 192.168.100.10
    memory_mb: 4096
    vcpus: 2
    disks: ["20G", "50G"]
    swap_size_gb: 2
```
