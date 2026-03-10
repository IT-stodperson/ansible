# grub_hardening

Hardens GRUB on Debian servers: sets a PBKDF2 superuser password, adds
`--unrestricted` to normal boot entries so users are not asked for the
password on every boot, appends hardened kernel boot parameters, locks down
GRUB file permissions, and verifies that the generated `grub.cfg` contains
the expected superuser and password entries.

## Requirements

- Ansible ≥ 2.15
- Debian 12 (Bookworm)
- `become: true`
- `vault_grub_password` must be defined in Ansible Vault

## Role variables

All variables are defined in `defaults/main.yml` and can be overridden.

### GRUB password

| Variable | Default | Description |
|---|---|---|
| `grub_superuser` | `root` | GRUB superuser account name |
| `grub_pbkdf2_iterations` | `600000` | PBKDF2 iteration count (higher = slower brute force) |
| `grub_pbkdf2_salt_length` | `64` | PBKDF2 salt length in bytes |
| `vault_grub_password` | *(no default)* | **Required.** GRUB password — define in Ansible Vault |

### Kernel boot parameters

| Variable | Default | Description |
|---|---|---|
| `grub_cmdline_linux_hardening` | see defaults | List of extra kernel parameters appended to `GRUB_CMDLINE_LINUX` |

Default hardening parameters include: `apparmor=1 security=apparmor audit=1
usbcore.nousb slab_nomerge init_on_alloc=1 init_on_free=1 pti=on
mitigations=auto,nosmt vsyscall=none debugfs=off oops=panic loglevel=0`
(among others).

## Tags

| Tag | Tasks |
|---|---|
| `grub` | All tasks in this role |
| `grub_password` | GRUB superuser password configuration |
| `grub_boot_params` | Kernel boot parameter hardening |
| `grub_permissions` | GRUB file permission lockdown |
| `grub_verify` | Verification that `grub.cfg` is correct |

## Dependencies

None.

## Example playbook

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: grub_hardening
      vars:
        grub_superuser: grubadmin
        grub_pbkdf2_iterations: 1000000
```

Define the password in `inventory/group_vars/all/vault.yml` (Ansible Vault):

```yaml
vault_grub_password: "your-secure-password"
```

```bash
ansible-playbook playbooks/security-hardening.yml --tags grub
ansible-playbook playbooks/security-hardening.yml --tags grub --check --diff
```

## License

MIT-0
