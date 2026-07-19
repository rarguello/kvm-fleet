# kvm_vm

Declarative VM lifecycle: creates VMs with `virt-install` + cloud-init (NoCloud, static networking), destroys them cleanly, and reconciles state/autostart on ones that already exist.

## Behavior

- **`state: present`** (default) — VM exists and is running.
- **`state: stopped`** — VM exists, powered off.
- **`state: absent`** — VM is force-stopped, undefined, and its disk files removed (read from `virsh domblklist`, not guessed from current vars).
- **Already exists:** never recreated or resized. If declared `memory_mb`/`vcpus` no longer match, a warning is printed; state and autostart are still reconciled.
- **`mac`** is optional — if unset, libvirt/virt-install assigns one. If set, it must start with `52:54` (the QEMU/KVM locally-administered OUI).
- **Access**: `cloud_user_password` is per-VM only — there's no fleet-wide default, so one leaked password can't unlock every VM. The ssh key (per-VM `ssh_public_key_file`, or the `kvm_ssh_public_key_file` default) is optional too. At least one of the two is required, or creation fails before touching anything.

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
    cloud_user_password: ""                  # optional, no fleet-wide fallback
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

### Per-VM password, encrypted inline

`cloud_user_password` takes a plain string or an inline-vault-encrypted one — no
separate `vault.yml` needed. Generate it once per VM:

```bash
ansible-vault encrypt_string --vault-password-file .vault_password_file \
  --stdin-name 'cloud_user_password' <<< 'S3cur3P@ss!'
```

Paste the output straight into the VM's entry:

```yaml
kvm_vms:
  - name: web01
    state: present
    image: rocky9
    network: lab
    ip: 192.168.100.10
    cloud_user_password: !vault |
      $ANSIBLE_VAULT;1.1;AES256
      62386565313534623535303331643738373138346331623162393333363936376339336332383365
      3066306664393936363661346663666666396166383837320a366630353230333133643861393761
      35623564326135323564346463396431363766343363306439353231313936376232666239383764
      3232646232396138360a643766353332363234626563663134393338323839663661656265663231
      3833
```
