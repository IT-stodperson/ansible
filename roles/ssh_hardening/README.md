ssh_hardening
=============

Harden SSH on Debian-based servers. This role filters weak
Diffie-Hellman moduli, deploys a secure `sshd_config` drop-in with
hardened cipher/KEX/MAC settings, and enforces strict file permissions
on SSH configuration files and host keys.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).
- An `ansible` user and group must exist on the target (the role
  creates them if missing).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

### Diffie-Hellman moduli

| Variable | Default | Description |
|---|---|---|
| `ssh_moduli_min_bits` | `3071` | Minimum bit size for DH moduli. Entries below this threshold are removed from `/etc/ssh/moduli`. |

### What the role configures (non-variable, hardcoded in template)

**SSH configuration** (`/etc/ssh/sshd_config.d/ssh-hardening.conf`):
- Hardened cipher, KEX, and MAC algorithm selection
- Root login restrictions, authentication settings
- Banner, idle timeout, and session limits

**File permissions**:
- `/etc/ssh/sshd_config` — `root:root 0600`
- `/etc/ssh/sshd_config.d/*.conf` — `root:root 0600`
- SSH private host keys — `root:root 0600`
- SSH public host keys — `root:root 0644`

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all SSH hardening with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - ssh_hardening
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Adjust minimum DH moduli bit size

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: ssh_hardening
      ssh_moduli_min_bits: 4095
```

### 3. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 4. Override variables from the command line

```bash
ansible-playbook -i inventory site.yml \
  -e ssh_moduli_min_bits=4095
```

### 5. Dry-run (check mode)

Preview changes without modifying the system:

```bash
ansible-playbook -i inventory site.yml --check --diff
```

### 6. Use in a larger playbook with other hardening roles

```yaml
- hosts: debian_servers
  become: true
  roles:
    - accounts_hardening
    - pam_hardening
    - sudo_hardening
    - ssh_hardening
    - logging_auditing
```

### 7. Set variables per environment in group_vars

```yaml
# group_vars/production.yml
ssh_moduli_min_bits: 4095

# group_vars/development.yml
ssh_moduli_min_bits: 3071
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
