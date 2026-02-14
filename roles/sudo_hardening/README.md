sudo_hardening
==============

Harden sudo on Debian-based servers. This role deploys a secure sudoers
drop-in configuration with sane session defaults, full I/O logging,
strict file permissions on all sudoers files, and an audit of NOPASSWD
entries.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

### Environment and Timing

| Variable | Default | Description |
|---|---|---|
| `sudo_timestamp_timeout` | `15` | Minutes of inactivity before sudo requires password again. |
| `sudo_passwd_timeout` | `1` | Minutes allowed to enter password before the prompt times out. |
| `sudo_timestamp_type` | `global` | Timestamp scope — `global` uses one prompt per session, not per terminal. |

### Logging

| Variable | Default | Description |
|---|---|---|
| `sudo_logfile` | `/var/log/sudo.log` | Dedicated log file for all sudo commands. |
| `sudo_iolog_dir` | `/var/log/sudo-io` | Directory for storing I/O session logs. |

### Security

| Variable | Default | Description |
|---|---|---|
| `sudo_secure_path` | `/usr/local/sbin:…:/bin` | Restricted PATH to prevent execution of malicious binaries. |
| `sudo_passwd_tries` | `3` | Maximum number of password attempts before failing. |

### Alerts

| Variable | Default | Description |
|---|---|---|
| `sudo_mailto` | `itstodperson@gymnasium.ax` | Email address to receive sudo security alerts on failed attempts. |

### Service Accounts

| Variable | Default | Description |
|---|---|---|
| `sudo_service_accounts` | *(see below)* | List of `{name, privileges, comment}` dicts for accounts that require passwordless sudo. |

Default service accounts:

```yaml
sudo_service_accounts:
  - name: ansible
    privileges: "ALL=(ALL) NOPASSWD: ALL"
    comment: Ansible automation — requires passwordless sudo for remote management
```

### What the role configures (non-variable)

- `/etc/sudoers.d/10_sudo-hardening` — deployed from template with `visudo` validation, `root:root 0440`
- `/etc/sudoers` — enforced `root:root 0440`
- All files in `/etc/sudoers.d/` — enforced `root:root 0440`
- `Defaults env_reset` — reset environment to prevent malicious exploitation
- `Defaults use_pty` — force pseudo-terminal allocation
- `Defaults mail_badpass` — email alert on failed password attempts
- `Defaults log_input, log_output` — full I/O audit trail
- NOPASSWD audit — reports all NOPASSWD entries found across sudoers files

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all sudo hardening with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - sudo_hardening
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Change session timeout and password tries

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: sudo_hardening
      sudo_timestamp_timeout: 5
      sudo_passwd_tries: 5
```

### 3. Add additional service accounts

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: sudo_hardening
      sudo_service_accounts:
        - name: ansible
          privileges: "ALL=(ALL) NOPASSWD: ALL"
          comment: Ansible automation
        - name: monitoring
          privileges: "ALL=(ALL) NOPASSWD: /usr/bin/systemctl status *"
          comment: Monitoring agent — limited passwordless access
```

### 4. Run only a specific subsystem using tags

Apply only the sudoers configuration:

```bash
ansible-playbook -i inventory site.yml --tags sudo_configuration
```

Apply only the file permission hardening:

```bash
ansible-playbook -i inventory site.yml --tags sudo_permissions
```

Apply only the NOPASSWD audit:

```bash
ansible-playbook -i inventory site.yml --tags sudo_audit
```

Available tags: `sudo`, `sudo_configuration`, `sudo_permissions`,
`sudo_audit`.

### 5. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 6. Override variables from the command line

```bash
ansible-playbook -i inventory site.yml \
  -e sudo_timestamp_timeout=5 \
  -e sudo_passwd_tries=5
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
    - grub_hardening
    - kernel_hardening
    - accounts_hardening
    - pam_hardening
    - sudo_hardening
    - ssh_hardening
```

### 9. Set variables per environment in group_vars

```yaml
# group_vars/production.yml
sudo_timestamp_timeout: 5
sudo_passwd_tries: 3

# group_vars/development.yml
sudo_timestamp_timeout: 30
sudo_passwd_tries: 5
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
