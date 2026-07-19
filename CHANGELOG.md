# Changelog

## 0.2.0

- `kvm_vm`: `gateway` and `dns_nameservers` are now optional per-VM overrides — both default to the network's own `gateway`, as before, but a VM can now declare a different default route or resolver.
- `examples/`: use FQCN role names instead of the `collections:` play keyword (matches `ansible-lint`'s `fqcn` rule); CI now lints `examples/` too.
- `README.md`: installation and consumption examples updated to install from Galaxy.

## 0.1.0

Initial release.

- `kvm_hypervisor`: bootstrap a bare-metal or nested host as a KVM hypervisor.
- `kvm_images`: maintain a base qcow2 image catalog.
- `kvm_network`: declarative libvirt networks with live DHCP/DNS reconciliation.
- `kvm_vm`: declarative VM lifecycle via virt-install + cloud-init.
