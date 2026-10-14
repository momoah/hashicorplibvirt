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
ansible-playbook playbooks/01_build_vm.yml        # Create the VM (virt-install + cloud-init seed ISO)
ansible-playbook playbooks/02_rhel_updates.yml    # Register, pin release, enable repos, update, reboot if needed
ansible-playbook playbooks/03_install_vault.yml   # HashiCorp repo + pinned Vault, version-locked
ansible-playbook playbooks/04_configure_vault.yml # Raft + TLS config, init, unseal, KV v2 at secret/
ansible-playbook playbooks/05_harden_vault.yml    # Audit log, admin policy, userpass login, revoke root
ansible-playbook playbooks/99_destroy_vm.yml      # Tear down the VM to rebuild
```

After a VM reboot Vault starts sealed: re-run `04_configure_vault.yml` to unseal.

Controller files (mode 0600, never commit):
- `~/vault-init.json` – unseal key (its root token is revoked by step 05)
- `~/vault-admin-password` – password for the `admin` userpass login

## Author(s)

- Mohammad Ahmad
