pam_hardening
=============

Harden PAM (Pluggable Authentication Modules) on Debian-based servers.
This role restricts `su` access to a dedicated group, enforces password
quality via pwquality, configures account lockout with faillock, enables
password history via pwhistory, and removes nullok from the unix PAM
profile.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

### SU restriction

| Variable | Default | Description |
|---|---|---|
| `pam_su_group` | `nosu` | Group allowed to use `su`. Members of this group can switch users; all others are denied. |

### Password quality (pwquality)

| Variable | Default | Description |
|---|---|---|
| `pam_pwquality_difok` | `2` | Minimum number of characters that must differ from the old password |
| `pam_pwquality_minlen` | `8` | Minimum password length |
| `pam_pwquality_dcredit` | `-1` | Require at least 1 digit |
| `pam_pwquality_ucredit` | `-1` | Require at least 1 uppercase letter |
| `pam_pwquality_lcredit` | `-1` | Require at least 1 lowercase letter |
| `pam_pwquality_ocredit` | `-1` | Require at least 1 special character |
| `pam_pwquality_minclass` | `3` | Minimum number of character classes required |
| `pam_pwquality_maxrepeat` | `3` | Maximum consecutive identical characters |
| `pam_pwquality_maxsequence` | `3` | Maximum length of monotonic character sequences |
| `pam_pwquality_dictcheck` | `1` | Enable dictionary check |
| `pam_pwquality_usercheck` | `1` | Check if password contains the username |
| `pam_pwquality_usersubstr` | `3` | Length of username substrings to check |
| `pam_pwquality_enforcing` | `1` | Enforce quality checks (reject non-compliant passwords) |
| `pam_pwquality_enforce_for_root` | `true` | Apply quality rules to the root user as well |

### Account lockout (faillock)

| Variable | Default | Description |
|---|---|---|
| `pam_faillock_audit` | `true` | Log failed authentication attempts |
| `pam_faillock_deny` | `3` | Number of failed attempts before lockout |
| `pam_faillock_fail_interval` | `900` | Time window in seconds for counting failures |
| `pam_faillock_unlock_time` | `900` | Seconds until a locked account is automatically unlocked |
| `pam_faillock_even_deny_root` | `true` | Apply lockout to the root account |
| `pam_faillock_root_unlock_time` | `900` | Unlock time for the root account specifically |

### Password history (pwhistory)

| Variable | Default | Description |
|---|---|---|
| `pam_pwhistory_remember` | `24` | Number of previous passwords to remember |
| `pam_pwhistory_enforce_for_root` | `true` | Enforce password history for the root user |

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all PAM hardening with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - pam_hardening
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Customize password quality settings

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: pam_hardening
      pam_pwquality_minlen: 12
      pam_pwquality_minclass: 4
      pam_pwquality_maxrepeat: 2
```

### 3. Adjust account lockout policy

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: pam_hardening
      pam_faillock_deny: 5
      pam_faillock_unlock_time: 1800
      pam_faillock_even_deny_root: false
```

### 4. Run only a specific subsystem using tags

Restrict only `su` access:

```bash
ansible-playbook -i inventory site.yml --tags pam_su
```

Apply only password quality settings:

```bash
ansible-playbook -i inventory site.yml --tags pam_pwquality
```

Apply only account lockout:

```bash
ansible-playbook -i inventory site.yml --tags pam_faillock
```

Apply only password history:

```bash
ansible-playbook -i inventory site.yml --tags pam_pwhistory
```

Harden only the unix PAM profile:

```bash
ansible-playbook -i inventory site.yml --tags pam_unix
```

Available tags: `pam`, `pam_su`, `pam_pwquality`, `pam_faillock`,
`pam_pwhistory`, `pam_unix`.

### 5. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 6. Override variables from the command line

```bash
ansible-playbook -i inventory site.yml \
  -e pam_pwquality_minlen=12 \
  -e pam_faillock_deny=5
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
    - accounts_hardening
    - pam_hardening
    - sudo_hardening
    - ssh_hardening
```

### 9. Set variables per environment in group_vars

```yaml
# group_vars/production.yml
pam_pwquality_minlen: 12
pam_faillock_deny: 3
pam_pwhistory_remember: 24

# group_vars/development.yml
pam_pwquality_minlen: 8
pam_faillock_deny: 10
pam_pwhistory_remember: 5
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
