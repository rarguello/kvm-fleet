# kvm_network

Manages libvirt networks declaratively: creation, DHCP reservations, and internal DNS (including OpenShift-UPI-style `api`/`api-int`/`*.apps` records), reconciled live where possible.

## Behavior

- **New network:** defined and started in one step, with every matching VM's DHCP+DNS entry already baked in.
- **Existing network, per-VM DHCP/DNS changes** (a VM added, removed, or its `ip`/`mac` changed): reconciled live via `virsh net-update`, without restarting the network or disrupting other VMs on it.
- **Existing network, base attributes changed** (`mode`, `cidr`, `dhcp_start`/`dhcp_end`, `domain`, `dns_aliases`, `dnsmasq_options`): **not** applied automatically. Add the network's name to `kvm_network_recreate` to authorize a destroy+redefine — this briefly disrupts every VM on that network, so it's opt-in only.

DHCP reservations require the VM to declare `mac`; DNS entries work from `ip` alone (a VM without `mac` still gets a DNS record, just no fixed DHCP lease).

## Requirements

`community.libvirt` collection, `python3-libvirt` on the managed host (installed by `kvm_hypervisor`).

## Role Variables

```yaml
kvm_networks:
  - name: lab
    state: present            # present | absent
    mode: nat                  # nat (recommended) | route | isolated
    cidr: 192.168.100.0/24
    gateway: 192.168.100.1
    dhcp_start: 192.168.100.100
    dhcp_end: 192.168.100.200
    domain: example.internal   # also used to build each VM's FQDN
    dns_forwarders: [1.1.1.1]
    autostart: true
    bridge: virbr-lab           # optional; omit to let libvirt pick one
                                  # (avoids the 15-char Linux ifname limit)
    hosts: []                    # extra DHCP+DNS reservations beyond kvm_vms
    dns_aliases: []               # DNS-only aliases, no DHCP — e.g. OCP api/api-int
    dnsmasq_options: []            # raw dnsmasq passthrough — e.g. wildcard *.apps

kvm_network_recreate: []   # network names authorized to be destroyed + redefined
```

## Example

```yaml
kvm_networks:
  - name: ocp4
    state: present
    mode: nat
    cidr: 192.168.150.0/24
    gateway: 192.168.150.1
    dhcp_start: 192.168.150.100
    dhcp_end: 192.168.150.200
    domain: ocp4.example.internal
    dns_aliases:
      - ip: 192.168.150.50
        hostnames: [api, api-int]
    dnsmasq_options:
      - "address=/.apps.ocp4.example.internal/192.168.150.51"
```
