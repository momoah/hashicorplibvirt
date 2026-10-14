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
ansible-playbook playbooks/03_install_vault.yml   # HashiCorp repo + vault-{{ vault_version }}, version-locked
ansible-playbook playbooks/04_configure_vault.yml # Raft + TLS config, init, unseal, KV v2 at secret/
ansible-playbook playbooks/99_destroy_vm.yml      # Tear down the VM to rebuild
```

After a VM reboot Vault starts sealed. Re-run `04_configure_vault.yml` to unseal it.

Unseal key and root token: `~/vault-init.json` on the controller (mode 0600). Never commit it.

Upgrade Vault: change `vault_version`, then remove the lock with
`sudo dnf versionlock delete vault` on the VM and re-run 03.

## Author(s)

- Mohammad Ahmad
