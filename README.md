# kvm-fleet

Declarative management of a KVM/libvirt hypervisor fleet with Ansible. Hypervisors, networks, base images, and VMs are all data — lists with `state: present` or `state: absent` — reconciled by four focused roles.

## Concepts

| Concept | Declared as | Role |
|---|---|---|
| Hypervisor bootstrap | `kvm_hypervisor_bootstrap: true/false` | `kvm_hypervisor` |
| Base images | `kvm_images: []` | `kvm_images` |
| libvirt networks | `kvm_networks: []` | `kvm_network` |
| VMs | `kvm_vms: []` | `kvm_vm` |

Everything is declared per-hypervisor in inventory `host_vars`. Running the playbook reconciles the declared state against what's actually on each hypervisor. Adding a VM, network, or image is a one-line inventory change — no other file needs to change.

## Requirements

- Controller: ansible-core >= 2.15, `community.libvirt` collection.
- Managed hosts: EL9/EL10 family (RHEL, Rocky Linux, AlmaLinux, CentOS Stream), reachable over SSH with sudo. Bare metal or nested KVM both work.

## Installation

```bash
ansible-galaxy collection install git+https://github.com/rarguello/kvm-fleet.git
ansible-galaxy collection install -r requirements.yml   # community.libvirt
```

## Quickstart

```yaml
# inventory/hosts.ini
[hypervisors]
hypervisor1  ansible_host=hypervisor1.example.com ansible_user=admin
```

```yaml
# inventory/host_vars/hypervisor1.yml
kvm_hypervisor_bootstrap: true

kvm_networks:
  - name: lab
    state: present
    mode: nat
    cidr: 192.168.100.0/24
    gateway: 192.168.100.1
    dhcp_start: 192.168.100.100
    dhcp_end: 192.168.100.200
    domain: example.internal
    autostart: true

kvm_images:
  - name: rocky9
    state: present

kvm_vms:
  - name: web01
    state: present
    autostart: true
    image: rocky9
    network: lab
    ip: 192.168.100.10
    memory_mb: 2048
    vcpus: 2
    disks: ["20G"]
```

```bash
ansible-playbook playbooks/site.yml
```

Day-to-day VM changes don't need to re-run the hypervisor bootstrap or image catalog:

```bash
ansible-playbook playbooks/site.yml --tags vms
```

Other tags: `hypervisor`, `images`, `networks`.

More worked examples in [`examples/`](examples/): an OpenShift UPI network with `api`/`api-int`/`*.apps` DNS, static DHCP reservations with forward/reverse DNS, and handing a freshly-provisioned VM off to another role (`rhc` for RHSM/Insights registration, freeipa, a Satellite installer, ...) over SSH.

## Using this from your own inventory repo

The `inventory/` in this repo is a generic example only — your real fleet's inventory (real hostnames, IPs, vaulted secrets) belongs in a separate, private repo that consumes this collection:

```yaml
# your-repo/collections/requirements.yml
collections:
  - name: https://github.com/rarguello/kvm-fleet.git
    type: git
    version: main
```

```yaml
# your-repo/playbooks/site.yml
- hosts: hypervisors
  collections:
    - rarguello.kvm_fleet
  roles:
    - kvm_hypervisor
    - kvm_images
    - kvm_network
    - kvm_vm
```

## Roles

Each role documents its own variables in full:

- [`kvm_hypervisor`](roles/kvm_hypervisor/README.md) — installs virtualization packages, enables libvirt daemons, storage pool, optional nested virtualization.
- [`kvm_images`](roles/kvm_images/README.md) — downloads/removes base qcow2 images, or verifies manually-placed ones (RHEL).
- [`kvm_network`](roles/kvm_network/README.md) — libvirt networks with DHCP reservations and internal DNS, reconciled live.
- [`kvm_vm`](roles/kvm_vm/README.md) — VM lifecycle via virt-install + cloud-init.

## Known limitations

- **No live resize.** If an existing VM's `memory_mb`/`vcpus` no longer match what's declared, `kvm_vm` warns but does not apply the change. Recreate (`state: absent` then `present`) or resize manually.
- **OS disk must be at least as large as the base image's virtual size.** `qemu-img` silently truncates a smaller overlay — no error — which cuts off whatever partition sits at the end (usually root) and leaves the guest unable to boot. `kvm_vm` checks this and fails loudly before creating anything.
- **Network base-attribute changes require opt-in.** Changing an existing network's `mode`/`cidr`/`dhcp_start`/`dhcp_end`/`domain`/`dns_aliases`/`dnsmasq_options` after creation needs the network's name added to `kvm_network_recreate` — recreating a network briefly disrupts every VM on it, so it's never automatic. Per-VM DHCP/DNS reservations (adding/removing a VM) *are* reconciled live, no opt-in needed.
- **Guest NIC device name.** `cloud-init` writes the static network config for a specific device name (`eth0` or `enp1s0` depending on the image). A mismatch silently falls back to DHCP. Verified defaults are documented in `kvm_known_images`; check with `ip -brief addr show` on first boot if a VM doesn't come up on its declared IP.
- **RHEL images** require Red Hat portal authentication and can't be fetched by URL. Set `filename` instead of `url` on the `kvm_images` entry and copy the qcow2 into place by hand.
- **Image-mode (bootc) hypervisors.** `dnf`/package management doesn't apply the usual way when packages are baked into the image. Set `kvm_hypervisor_manage_packages: false` — everything else (libvirt daemons, storage pool, networks, VMs) works normally.

## License

Apache-2.0
