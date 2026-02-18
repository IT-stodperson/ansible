apt_hardening
=============

Harden APT (Advanced Package Tool) on Debian-based servers.
This role disables weak dependency installation (Recommends/Suggests),
disables translation downloads, enforces strict ownership and
permissions on GPG keyrings, source lists, and authentication configs,
installs package integrity tools (`debsums`, `apt-show-versions`),
and configures automatic security upgrades via `unattended-upgrades`.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

### APT configuration

| Variable | Default | Description |
|---|---|---|
| `apt_install_recommends` | `"0"` | Whether to install recommended packages (`0` = disabled) |
| `apt_install_suggests` | `"0"` | Whether to install suggested packages (`0` = disabled) |
| `apt_acquire_languages` | `"none"` | Languages for translation downloads (`none` = disabled) |

### Package integrity and patch management

| Variable | Default | Description |
|---|---|---|
| `apt_install_debsums` | `true` | Install `debsums` for verifying installed package file integrity |
| `apt_install_apt_show_versions` | `true` | Install `apt-show-versions` for patch management reporting |
| `apt_debsums_cron_check` | `"daily"` | Debsums cron frequency: `daily`, `weekly`, `monthly`, or `never` (Lynis PKGS-7370) |

### Unattended upgrades

| Variable | Default | Description |
|---|---|---|
| `apt_unattended_upgrades_enabled` | `true` | Install and configure `unattended-upgrades` |
| `apt_unattended_upgrades_origins` | Debian security + stable | Origins-Pattern entries for eligible repositories |
| `apt_unattended_upgrades_blacklist` | `[]` | Packages to exclude from automatic upgrades (regex) |
| `apt_unattended_upgrades_mail` | `""` | Email address for notifications (empty = disabled) |
| `apt_unattended_upgrades_mail_report` | `"on-change"` | When to send mail: `always`, `only-on-error`, `on-change` |
| `apt_unattended_upgrades_remove_unused_kernel` | `true` | Remove unused kernel packages after upgrade |
| `apt_unattended_upgrades_remove_new_unused_deps` | `true` | Remove new unused dependencies after upgrade |
| `apt_unattended_upgrades_remove_unused_deps` | `true` | Remove all unused dependencies after upgrade |
| `apt_unattended_upgrades_auto_reboot` | `false` | Automatically reboot when required (disabled for safety) |
| `apt_unattended_upgrades_auto_reboot_time` | `"02:00"` | Scheduled reboot time if auto-reboot is enabled |
| `apt_unattended_upgrades_syslog_enabled` | `true` | Log to syslog |
| `apt_unattended_upgrades_syslog_facility` | `"daemon"` | Syslog facility to log to |

### APT periodic intervals

| Variable | Default | Description |
|---|---|---|
| `apt_periodic_update_package_lists` | `1` | Days between package list updates (`0` = disabled) |
| `apt_periodic_download_upgradeable` | `1` | Days between downloading upgradeable packages |
| `apt_periodic_unattended_upgrade` | `1` | Days between running unattended upgrades |
| `apt_periodic_autoclean_interval` | `7` | Days between auto-cleaning the package cache |

### Directory permissions

| Variable | Default | Description |
|---|---|---|
| `apt_directories` | see `defaults/main.yml` | List of APT directories to enforce `root:root 0755` on |

### What the role hardens (non-variable)

- GPG keyring files in `/usr/share/keyrings` and `/etc/apt/trusted.gpg.d` — enforced `root:root 0644`
- Source files in `/etc/apt/sources.list.d` — enforced `root:root 0644`
- Auth files in `/etc/apt/auth.conf.d` — enforced `root:root 0644`
- Verification tasks confirm no files with bad permissions remain after hardening

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all APT hardening with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - apt_hardening
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Override APT configuration settings

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: apt_hardening
      apt_install_recommends: "1"
      apt_acquire_languages: "en"
```

### 3. Run only a specific subsystem using tags

Apply only the APT configuration (weak deps, translations):

```bash
ansible-playbook -i inventory site.yml --tags apt_config
```

Apply only the file and directory permission hardening:

```bash
ansible-playbook -i inventory site.yml --tags apt_permissions
```

Run only the verification checks:

```bash
ansible-playbook -i inventory site.yml --tags apt_verify
```

Install only the hardening packages (debsums, apt-show-versions, unattended-upgrades):

```bash
ansible-playbook -i inventory site.yml --tags apt_packages
```

Available tags: `apt`, `apt_packages`, `apt_config`, `apt_permissions`, `apt_verify`.

### 4. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 5. Override variables from the command line

```bash
ansible-playbook -i inventory site.yml \
  -e apt_install_recommends=1 \
  -e apt_acquire_languages=en
```

### 6. Dry-run (check mode)

Preview what the role would change without modifying the system:

```bash
ansible-playbook -i inventory site.yml --check --diff
```

### 7. Use in a larger playbook with other hardening roles

```yaml
- hosts: debian_servers
  become: true
  roles:
    - common
    - apt_hardening
    - accounts_hardening
    - pam_hardening
    - auditd
```

### 8. Configure unattended-upgrades with email notifications and auto-reboot

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: apt_hardening
      apt_unattended_upgrades_mail: "admin@example.com"
      apt_unattended_upgrades_auto_reboot: true
      apt_unattended_upgrades_auto_reboot_time: "03:00"
```

### 9. Disable specific packages from automatic upgrades

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: apt_hardening
      apt_unattended_upgrades_blacklist:
        - "linux-image"
        - "postgresql-*"
```

### 10. Use on Ubuntu hosts (override origins)

```yaml
- hosts: ubuntu_servers
  become: true
  roles:
    - role: apt_hardening
      apt_unattended_upgrades_origins:
        - "origin=Ubuntu,archive=${distro_codename}-security,label=Ubuntu"
```

### 11. Set variables per environment in group_vars

```yaml
# group_vars/production.yml
apt_install_recommends: "0"
apt_install_suggests: "0"
apt_unattended_upgrades_auto_reboot: false

# group_vars/development.yml
apt_install_recommends: "1"
apt_install_suggests: "0"
apt_unattended_upgrades_auto_reboot: true
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
