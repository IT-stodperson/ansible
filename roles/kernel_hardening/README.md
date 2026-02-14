kernel_hardening
================

Harden the Linux kernel on Debian-based servers. This role blacklists
dangerous or unnecessary kernel modules via modprobe rules, deploys
comprehensive sysctl parameters for kernel, filesystem, and network
hardening, and restricts process limits and core dumps.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

### Process limits

| Variable | Default | Description |
|---|---|---|
| `nproc_hard_limit` | `100` | Hard limit for number of processes per user (fork bomb protection). Increase for busy services. |

### Module blacklist

| Variable | Default | Description |
|---|---|---|
| `blacklisted_modules` | *(25 modules)* | List of `{name, comment}` dicts for kernel modules to blacklist. See `defaults/main.yml` for the full list. |

Default blacklisted modules include: `afs`, `atm`, `can`, `ceph`,
`cifs`, `cramfs`, `dccp`, `exfat`, `freevxfs`, `fscache`, `fuse`,
`gfs2`, `hfs`, `hfsplus`, `jffs2`, `overlayfs`, `rds`, `sctp`,
`squashfs`, `tipc`, `udf`, `usb_storage`, `firewire_ohci`,
`firewire_core`, `firewire_sbp2`.

### What the role configures (non-variable, hardcoded in templates)

**Sysctl parameters** (`/etc/sysctl.d/60-kernel-hardening.conf`):
- Kernel: ASLR, dmesg restriction, kptr restriction, SysRq disabled,
  unprivileged BPF disabled, unprivileged user namespaces disabled,
  ptrace scope 3, BPF JIT hardening
- Filesystem: protected FIFOs/hardlinks/regular/symlinks, suid_dumpable
  disabled, core dumps disabled, minimal swappiness
- Network: ICMP broadcast ignore, bogus error ignore, forwarding
  disabled, martian logging, SYN cookies, redirect rejection, source
  routing disabled, reverse path filtering

**Module blacklist** (`/etc/modprobe.d/kernel-hardening.conf`):
- Each module is both blacklisted and set to `install /bin/false`

**Process limits** (`/etc/security/limits.d/10-coredump-debian.conf`):
- Hard nproc limit (configurable)
- Core dumps disabled (`hard core 0`)

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all kernel hardening with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - kernel_hardening
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Increase process limit for application servers

```yaml
- hosts: app_servers
  become: true
  roles:
    - role: kernel_hardening
      nproc_hard_limit: 500
```

### 3. Customise the module blacklist

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: kernel_hardening
      blacklisted_modules:
        - { name: cramfs, comment: "Compressed ROM filesystem" }
        - { name: freevxfs, comment: "VERITAS File System" }
        - { name: usb_storage, comment: "USB storage" }
```

### 4. Run only a specific subsystem using tags

Apply only the module blacklist:

```bash
ansible-playbook -i inventory site.yml --tags kernel_modules
```

Apply only the sysctl parameters:

```bash
ansible-playbook -i inventory site.yml --tags kernel_sysctl
```

Apply only the process/core dump limits:

```bash
ansible-playbook -i inventory site.yml --tags kernel_limits
```

Available tags: `kernel`, `kernel_modules`, `kernel_sysctl`,
`kernel_limits`.

### 5. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 6. Override variables from the command line

```bash
ansible-playbook -i inventory site.yml \
  -e nproc_hard_limit=500
```

### 7. Dry-run (check mode)

Preview changes without modifying the system:

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
    - accounts_hardening
    - auditd
```

### 9. Set variables per environment in group_vars

```yaml
# group_vars/production.yml
nproc_hard_limit: 200

# group_vars/development.yml
nproc_hard_limit: 500
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
