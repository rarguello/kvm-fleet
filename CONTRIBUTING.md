# Contributing

Issues and pull requests are welcome at https://github.com/rarguello/kvm-fleet.

Before submitting a change:

- `yamllint` and `ansible-lint` should pass clean (`.ansible-lint` documents the two rules that are intentionally disabled, and why).
- `ansible-playbook --syntax-check playbooks/site.yml` should pass.
- If you can test against a real KVM hypervisor, do — this project has repeatedly found real bugs (silent disk truncation, lost DHCP updates, broken shutdown races) that static checks alone missed.

## Releasing

1. Bump `version` in `galaxy.yml` and add an entry to `CHANGELOG.md`.
2. `git tag vX.Y.Z && git push origin vX.Y.Z` — the `Release` workflow builds the collection tarball, publishes it to Ansible Galaxy (using the `GALAXY_API_KEY` repo secret), and creates the GitHub Release with the tarball attached.
