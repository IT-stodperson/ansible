grub_hardening
==============

Harden GRUB on Debian-based servers. This role sets a GRUB superuser
password (PBKDF2), adds `--unrestricted` to normal boot entries so
regular booting is unaffected, deploys hardened kernel command-line
parameters (AppArmor, audit, USB restrictions, memory allocator
hardening, CPU mitigations, kernel attack surface reduction), and
locks down `grub.cfg` file permissions.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system with GRUB 2
  (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).
- `vault_grub_password` must be defined in your Ansible Vault or
  host/group vars (the role does **not** provide a default).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

### GRUB password

| Variable | Default | Description |
|---|---|---|
| `grub_superuser` | `root` | The GRUB superuser account name |
| `grub_pbkdf2_iterations` | `600000` | PBKDF2 iteration count (higher = slower brute force) |
| `grub_pbkdf2_salt_length` | `64` | PBKDF2 salt length in bytes |

### Kernel boot parameters

| Variable | Default | Description |
|---|---|---|
| `grub_cmdline_linux_hardening` | See `defaults/main.yml` | Parameters appended to `GRUB_CMDLINE_LINUX` |

Default boot parameters include:

- `apparmor=1 security=apparmor` — enable AppArmor MAC
- `audit=1 audit_backlog_limit=8192` — enable kernel audit subsystem
- `usbcore.nousb` — disable USB at boot
- `slab_nomerge` — prevent slab merging (hardens heap isolation)
- `init_on_alloc=1 init_on_free=1` — zero memory on alloc/free
- `slub_debug=ZF` — enable red-zoning and sanity checks on SLUB
- `page_alloc.shuffle=1` — randomize page allocator free lists
- `randomize_kstack_offset=on` — randomize kernel stack offset per syscall
- `pti=on` — force page table isolation (Meltdown mitigation)
- `mitigations=auto,nosmt` — enable CPU mitigations, disable SMT
- `vsyscall=none` — disable legacy vsyscall interface
- `debugfs=off` — disable debugfs mount
- `oops=panic` — panic on kernel oops (prevent exploitation of corrupted state)
- `kfence.sample_interval=100` — enable KFENCE sampling for heap bug detection
- `loglevel=0` — suppress kernel log messages at boot

### Required vault / host variables

| Variable | Description |
|---|---|
| `vault_grub_password` | Plaintext GRUB password (stored in Ansible Vault) |

### What the role configures (non-variable)

- `/etc/grub.d/50_hardening` — superuser + PBKDF2 hash
- `/etc/grub.d/10_linux` — adds `--unrestricted` to the CLASS variable
- `/etc/default/grub.d/60-hardening.cfg` — kernel boot parameters
- `/boot/grub/grub.cfg` — permissions set to `0600 root:root`

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all GRUB hardening with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - grub_hardening
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Override GRUB superuser and PBKDF2 settings

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: grub_hardening
      grub_superuser: admin
      grub_pbkdf2_iterations: 1000000
```

### 3. Customize boot parameters

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: grub_hardening
      grub_cmdline_linux_hardening:
        - apparmor=1
        - security=apparmor
        - audit=1
        - audit_backlog_limit=8192
```

### 4. Run only a specific subsystem using tags

Apply only the GRUB password protection:

```bash
ansible-playbook -i inventory site.yml --tags grub_password
```

Apply only the boot parameters:

```bash
ansible-playbook -i inventory site.yml --tags grub_boot_params
```

Apply only file permission lockdown:

```bash
ansible-playbook -i inventory site.yml --tags grub_permissions
```

Available tags: `grub`, `grub_password`, `grub_boot_params`,
`grub_permissions`, `grub_verify`.

### 5. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 6. Override variables from the command line

```bash
ansible-playbook -i inventory site.yml \
  -e grub_superuser=admin \
  -e grub_pbkdf2_iterations=1000000
```

### 7. Dry-run (check mode)

Preview what the role would change without modifying the system:

```bash
ansible-playbook -i inventory site.yml --check --diff
```

### 8. Use in a larger playbook with other hardening roles

```yaml
- hosts: debian_servers
  become: true
  roles:
    - common
    - grub_hardening
    - kernel_hardening
    - pam_hardening
    - auditd
```

### 9. Set variables per environment in group_vars

```yaml
# group_vars/production.yml
grub_pbkdf2_iterations: 1000000
# Uses all defaults from the role (full hardening)

# group_vars/development.yml
grub_pbkdf2_iterations: 300000
grub_cmdline_linux_hardening:
  - apparmor=1
  - security=apparmor
  - audit=1
  - audit_backlog_limit=8192
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
