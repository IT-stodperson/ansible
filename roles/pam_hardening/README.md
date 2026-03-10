#SPDX-License-Identifier: MIT-0
# pam_hardening

Harden PAM (Pluggable Authentication Modules) on Debian-based servers.
The role restricts `su` access to a dedicated group, enforces password
complexity and history requirements via `pwquality` and `pwhistory`, and
configures `faillock` for account lockout after repeated failed logins.
All PAM configuration is managed via `pam-auth-update` drop-ins to remain
compatible with Debian's PAM management infrastructure.

## Requirements

- Ansible ≥ 2.15
- Debian 12 (Bookworm) targets
- `ansible` SSH user with sudo privileges

## Role variables

### SU restriction

| Variable | Default | Description |
|---|---|---|
| `pam_su_group` | `"nosu"` | Only members of this group may use `su`; all others are denied |

### Password quality (`pwquality`)

| Variable | Default | Description |
|---|---|---|
| `pam_pwquality_difok` | `2` | Minimum number of characters different from old password |
| `pam_pwquality_minlen` | `8` | Minimum password length |
| `pam_pwquality_dcredit` | `-1` | Require at least 1 digit |
| `pam_pwquality_ucredit` | `-1` | Require at least 1 uppercase letter |
| `pam_pwquality_lcredit` | `-1` | Require at least 1 lowercase letter |
| `pam_pwquality_ocredit` | `-1` | Require at least 1 special character |
| `pam_pwquality_minclass` | `3` | Minimum number of character classes required |
| `pam_pwquality_maxrepeat` | `3` | Maximum consecutive identical characters allowed |
| `pam_pwquality_maxsequence` | `3` | Maximum length of monotonic character sequence |
| `pam_pwquality_dictcheck` | `1` | Check against dictionary words |
| `pam_pwquality_usercheck` | `1` | Check if password contains the username |
| `pam_pwquality_usersubstr` | `3` | Minimum substring length checked against username |
| `pam_pwquality_enforcing` | `1` | Enforce the policy (0 = warn only) |
| `pam_pwquality_enforce_for_root` | `true` | Apply password quality checks to root |

### Account lockout (`faillock`)

| Variable | Default | Description |
|---|---|---|
| `pam_faillock_audit` | `true` | Log failed authentication attempts to the audit log |
| `pam_faillock_deny` | `3` | Lock account after this many consecutive failures |
| `pam_faillock_fail_interval` | `900` | Window (seconds) in which failures are counted |
| `pam_faillock_unlock_time` | `900` | Lock duration (seconds); 0 = permanent until admin reset |
| `pam_faillock_even_deny_root` | `true` | Apply lockout to root account |
| `pam_faillock_root_unlock_time` | `900` | Lock duration for root (seconds) |

### Password history (`pwhistory`)

| Variable | Default | Description |
|---|---|---|
| `pam_pwhistory_remember` | `24` | Number of previous passwords to remember and reject |
| `pam_pwhistory_enforce_for_root` | `true` | Apply password history enforcement to root |

## Example playbook

```yaml
- hosts: debian_servers
  roles:
    - role: pam_hardening
      tags: pam
```

## License

MIT-0
