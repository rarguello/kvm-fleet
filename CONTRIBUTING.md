# Contributing

Issues and pull requests are welcome at https://github.com/rarguello/kvm-fleet.

Before submitting a change:

- `yamllint` and `ansible-lint` should pass clean (`.ansible-lint` documents the two rules that are intentionally disabled, and why).
- `ansible-playbook --syntax-check playbooks/site.yml` should pass.
- If you can test against a real KVM hypervisor, do — this project has repeatedly found real bugs (silent disk truncation, lost DHCP updates, broken shutdown races) that static checks alone missed.

## Releasing

1. Bump `version` in `galaxy.yml` and add an entry to `CHANGELOG.md`.
2. `git tag vX.Y.Z && git push origin vX.Y.Z` — the `Release` workflow builds the collection tarball and creates the GitHub Release automatically.
3. Publish to Galaxy by hand: `ansible-galaxy collection publish rarguello-kvm_fleet-X.Y.Z.tar.gz`.
