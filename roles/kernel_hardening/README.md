# kernel_hardening

Hardens the Linux kernel on Debian servers: blacklists dangerous or unnecessary
kernel modules via modprobe rules, deploys hardened sysctl parameters, restricts
compiler binary access to root only (Lynis HRDN-7222), and sets hard `nproc`
limits.

## Requirements

- Ansible ≥ 2.15
- Debian 12 (Bookworm)
- `become: true`

## Role variables

All variables are defined in `defaults/main.yml` and can be overridden.

### Process limits

| Variable | Default | Description |
|---|---|---|
| `nproc_hard_limit` | `100` | Hard limit on number of processes per user |

### Compiler lockdown

| Variable | Default | Description |
|---|---|---|
| `compiler_binaries` | `/usr/bin/{as,cc,gcc,g++,make}` | Paths to restrict to `root:root 0700` |

### Module blacklist

| Variable | Default | Description |
|---|---|---|
| `blacklisted_modules` | see defaults | List of `{name, comment}` dicts for modules to blacklist |

Default blacklisted modules include: `afs`, `atm`, `can`, `ceph`, `cifs`,
`cramfs`, `dccp`, `exfat`, `freevxfs`, `fscache`, `fuse`, `gfs2`, `hfs`,
`hfsplus`, `jffs2`, `overlayfs`, `rds`, `sctp`, `squashfs`, `tipc`, `udf`,
`usb_storage`, `firewire_ohci`, `firewire_core`, `firewire_sbp2`.

## Tags

| Tag | Tasks |
|---|---|
| `kernel` | All tasks in this role |
| `kernel_sysctl` | Sysctl hardening configuration |
| `kernel_modules` | Kernel module blacklisting |
| `kernel_compilers` | Compiler binary permission restriction |
| `kernel_limits` | Process and resource limits |

## Dependencies

None.

## Example playbook

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: kernel_hardening
      vars:
        nproc_hard_limit: 200
        compiler_binaries:
          - /usr/bin/gcc
          - /usr/bin/g++
          - /usr/bin/cc
```

```bash
ansible-playbook playbooks/security-hardening.yml --tags kernel
ansible-playbook playbooks/security-hardening.yml --tags kernel_sysctl
```

## License

MIT-0
