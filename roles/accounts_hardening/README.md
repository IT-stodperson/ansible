accounts_hardening
=========

Harden user accounts and shell environment on Debian-based servers.
This role configures `/etc/login.defs` (password aging, encryption,
umask), enforces aging policies on existing accounts with exemption
support, and deploys a `/etc/profile.d` script that sets session
timeouts, history hardening, and safety aliases.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

### Password aging

| Variable | Default | Description |
|---|---|---|
| `shadow_pass_max_days` | `365` | Maximum days a password may be used |
| `shadow_pass_min_days` | `1` | Minimum days between password changes |
| `shadow_pass_warn_age` | `35` | Days of warning before password expiry |

### Session timeout

| Variable | Default | Description |
|---|---|---|
| `session_timeout` | `900` | Idle timeout in seconds (15 min). Applied to non-root, non-sudo users. |

### Exemptions

| Variable | Default | Description |
|---|---|---|
| `aging_exempt_users` | `[ansible, root]` | Users excluded from password aging enforcement |
| `aging_exempt_groups` | `[teachers]` | Groups whose members are excluded from password aging |

### What login.defs configures (non-variable, hardcoded in template)

- `ENCRYPT_METHOD YESCRYPT` (strongest available hash)
- `UMASK 027` (files 640, directories 750)
- `HOME_MODE 0700`
- `LOGIN_RETRIES 3`, `LOGIN_TIMEOUT 60`
- `LOG_OK_LOGINS yes`, `FAILLOG_ENAB yes`

### What the profile script configures

- `umask 027` enforced in shell
- `TMOUT` (read-only) for non-root, non-sudo users
- Core dumps disabled (`ulimit -c 0`)
- `noclobber` enabled
- History: 10 000 entries, timestamped, `ignoreboth`, append mode
- Safety aliases: `rm -I`, `cp -i`, `mv -i`

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all account hardening with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - accounts_hardening
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Override password aging policy

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: accounts_hardening
      shadow_pass_max_days: 90
      shadow_pass_min_days: 7
      shadow_pass_warn_age: 14
```

### 3. Customize session timeout and exemptions

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: accounts_hardening
      session_timeout: 600
      aging_exempt_users:
        - ansible
        - root
        - monitoring
      aging_exempt_groups:
        - teachers
        - admins
```

### 4. Run only a specific subsystem using tags

Deploy only the login.defs configuration:

```bash
ansible-playbook -i inventory site.yml --tags accounts_login_defs
```

Enforce password aging only (without touching login.defs or shell):

```bash
ansible-playbook -i inventory site.yml --tags accounts_aging
```

Deploy only the shell hardening profile:

```bash
ansible-playbook -i inventory site.yml --tags accounts_shell
```

Available tags: `accounts`, `accounts_login_defs`, `accounts_aging`,
`accounts_shell`.

### 5. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 6. Override variables from the command line

```bash
ansible-playbook -i inventory site.yml \
  -e shadow_pass_max_days=90 \
  -e session_timeout=600
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
    - accounts_hardening
    - pam_hardening
    - ssh_hardening
    - auditd
```

### 9. Set variables per environment in group_vars

```yaml
# group_vars/production.yml
shadow_pass_max_days: 90
shadow_pass_warn_age: 14
session_timeout: 900

# group_vars/development.yml
shadow_pass_max_days: 365
shadow_pass_warn_age: 35
session_timeout: 0
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
