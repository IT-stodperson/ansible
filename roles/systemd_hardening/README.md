systemd_hardening
=================

Harden systemd services on Debian-based servers by deploying drop-in
`override.conf` files that restrict capabilities, address families,
namespace access, and kernel interfaces. The role auto-discovers which
target services are installed and only deploys overrides for those that
are present.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).
- systemd 252+ (ships with Debian bookworm).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

| Variable | Default | Description |
|---|---|---|
| `systemd_hardening_services` | see below | List of service names to harden (without `.service` suffix) |

Default services:

```yaml
systemd_hardening_services:
  - auditd
  - cron
  - ssh
  - systemd-journald
  - systemd-timesyncd
```

### Shipped override templates

| Service | Override scope | Notes |
|---|---|---|
| `auditd` | Full hardening — capabilities, network, namespaces, kernel protections, UMask | Assumes Wazuh reads `/var/log/audit/audit.log` (file-based, not af_unix) |
| `ssh` | Capabilities, network families, kernel protections | `NoNewPrivileges`, `ProtectHome`, `PrivateDevices` intentionally omitted — PAM and PTY allocation require them |
| `cron` | Conservative kernel protections only | Cron executes arbitrary jobs; most directives would propagate to children and break user cron entries |
| `systemd-journald` | Supplemental — UMask only | Stock unit (systemd 252+) already includes extensive hardening |
| `systemd-timesyncd` | Supplemental — ProtectProc, ProcSubset, UMask | Stock unit (systemd 252+) already includes extensive hardening |

### Adding a new service

1. Create `templates/overrides/<service-name>.conf.j2` with the desired
   `[Service]` directives.
2. Append the service name to `systemd_hardening_services` (in defaults
   or in your inventory overrides).
3. Re-run the playbook. The role will detect the new template and deploy
   it if the service is installed on the target host.

### Post-deployment

After the role runs, overrides are deployed and `systemctl daemon-reload`
is triggered automatically. However, **services are not restarted** by
default — the new restrictions take effect after the next service restart
or host reboot.

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all systemd hardening with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - systemd_hardening
```

```bash
ansible-playbook -i inventory site.yml --tags systemd
```

### 2. Harden only specific services

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: systemd_hardening
      systemd_hardening_services:
        - auditd
        - ssh
```

### 3. Run only systemd hardening in a larger playbook

```bash
ansible-playbook -i inventory playbooks/security-hardening.yml --tags systemd
```

### 4. Run against a single host

```bash
ansible-playbook -i inventory playbooks/security-hardening.yml --tags systemd --limit wazuh.its.ax
```

### 5. Dry-run (check mode)

Preview changes without modifying the system:

```bash
ansible-playbook -i inventory playbooks/security-hardening.yml --tags systemd --check --diff
```

### 6. Use in a larger playbook with other hardening roles

```yaml
- hosts: debian_servers
  become: true
  roles:
    - kernel_hardening
    - ssh_hardening
    - logging_auditing
    - systemd_hardening
```

### 7. Verify deployed overrides on a host

```bash
# Show effective configuration for a service:
systemctl cat auditd.service

# Analyse security score (0 = fully hardened, 10 = no hardening):
systemd-analyze security auditd.service
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
