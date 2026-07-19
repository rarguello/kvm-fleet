# kvm_images

Maintains the base qcow2 image catalog on a hypervisor. Each `kvm_images` entry resolves against `kvm_known_images` first, so common distros don't need their URL/os_variant/device repeated in every inventory.

Two ways to provide an image:

- **`url` set** — downloaded with `get_url`. The on-disk filename defaults to the URL's basename (matching the mirror's own naming) unless `filename` is set explicitly.
- **`url` unset, `filename` set** — not downloaded; the role just verifies `<kvm_image_dir>/<filename>` already exists. This is the RHEL path: KVM Guest Images require Red Hat portal authentication and can't be fetched by URL, so copy the file there by hand first.

URLs in `kvm_known_images` are pinned to a specific dated build, never a `latest` symlink — VM overlay disks reference the base image by path, so if that path's content silently changes underneath an existing overlay, every VM built from it becomes unreproducible.

## Requirements

None beyond what `kvm_hypervisor` installs.

## Role Variables

```yaml
kvm_image_dir: /var/lib/libvirt/images

kvm_known_images:
  rocky9:
    url: https://dl.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud-Base-9.8-20260525.0.x86_64.qcow2
    checksum: sha256:...
    os_variant: rocky9
    device: eth0          # guest NIC name cloud-init writes the static config for
    cloud_user: rocky      # the image's default cloud-init user

kvm_images:
  - name: rocky9           # references kvm_known_images.rocky9
    state: present          # present | absent
  - name: rhel9
    filename: rhel-9.8-x86_64-kvm.qcow2   # copied there by hand, no url
    os_variant: rhel9.8
    device: eth0
    state: present
```

## Example

```yaml
kvm_images:
  - name: rocky9
    state: present
  - name: centos-stream9
    state: absent
```
