# vaultlibvirt

Build a single RHEL 9 VM on libvirt and install HashiCorp Vault (Community Edition).

## Prerequisites (hypervisor)

- libvirt + `virt-install` + `qemu-img` installed, bridge `br0` up
- RHEL 9 KVM guest image (qcow2) on disk, path set in `qcow_source`
- `~/mypassword` (hash from `openssl passwd -6 'yourpassword'`), `~/.ssh/id_rsa.pub`
- `~/activationkey` and `~/organization` (used from step 3)

## DNS records

```
; forward (local.labmesh.org)
vault   IN A    192.168.1.220
; reverse (192.168.1.db)
220     IN PTR  vault.local.labmesh.org.
```

## Run

```
ansible-playbook playbooks/01_build_vm.yml     # Create the VM (virt-install + cloud-init)
ansible-playbook playbooks/99_destroy_vm.yml   # Tear it down to rebuild
```

## Author(s)

- Mohammad Ahmad
