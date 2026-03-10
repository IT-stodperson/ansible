# apt_hardening

Hardens APT on Debian servers: disables weak dependency installation
(Recommends/Suggests), restricts language downloads, enforces directory
permissions, installs integrity-checking tools (`debsums`, `apt-show-versions`),
configures unattended security upgrades, and verifies APT directory permissions.

## Requirements

- Ansible ≥ 2.15
- Debian 12 (Bookworm)
- `become: true`

## Role variables

All variables are defined in `defaults/main.yml` and can be overridden.

### APT configuration

| Variable | Default | Description |
|---|---|---|
| `apt_install_recommends` | `"0"` | Disable automatic installation of Recommends |
| `apt_install_suggests` | `"0"` | Disable automatic installation of Suggests |
| `apt_acquire_languages` | `"none"` | Disable translation file downloads |

### Package integrity

| Variable | Default | Description |
|---|---|---|
| `apt_install_debsums` | `true` | Install `debsums` for package file integrity checking |
| `apt_install_apt_show_versions` | `true` | Install `apt-show-versions` |
| `apt_install_apt_transport_https` | `true` | Install `apt-transport-https` |
| `apt_debsums_cron_check` | `"daily"` | Frequency for debsums cron job (`daily`, `weekly`, `monthly`, `never`) |

### Unattended upgrades

| Variable | Default | Description |
|---|---|---|
| `apt_unattended_upgrades_enabled` | `true` | Enable automatic security updates |
| `apt_unattended_upgrades_origins` | Debian security origins | Origins eligible for automatic upgrade |
| `apt_unattended_upgrades_blacklist` | `[]` | Packages excluded from auto-upgrade (regex) |
| `apt_unattended_upgrades_mail` | `""` | Email for upgrade notifications (empty = disabled) |
| `apt_unattended_upgrades_mail_report` | `"on-change"` | When to send mail: `always`, `only-on-error`, `on-change` |
| `apt_unattended_upgrades_remove_unused_kernel` | `true` | Remove old kernel images |
| `apt_unattended_upgrades_remove_new_unused_deps` | `true` | Remove newly-unused dependencies |
| `apt_unattended_upgrades_remove_unused_deps` | `true` | Remove previously-unused dependencies |
| `apt_unattended_upgrades_auto_reboot` | `true` | Allow automatic reboot |
| `apt_unattended_upgrades_auto_reboot_with_users` | `true` | Allow reboot even when users are logged in |
| `apt_unattended_upgrades_auto_reboot_time` | `"02:00"` | Time for scheduled automatic reboots |
| `apt_unattended_upgrades_syslog_enabled` | `true` | Log to syslog |
| `apt_unattended_upgrades_syslog_facility` | `"daemon"` | Syslog facility |
| `apt_periodic_update_package_lists` | `1` | Days between package list updates |
| `apt_periodic_download_upgradeable` | `1` | Days between downloading upgradeable packages |
| `apt_periodic_unattended_upgrade` | `1` | Days between unattended upgrade runs |
| `apt_periodic_autoclean_interval` | `7` | Days between autoclean runs |

### Directory permissions

| Variable | Default | Description |
|---|---|---|
| `apt_directories` | see defaults | APT directories to lock down to `root:root 0755` |

## Tags

| Tag | Tasks |
|---|---|
| `apt` | All tasks in this role |
| `apt_packages` | Install hardening packages |
| `apt_config` | Deploy APT configuration templates |
| `apt_permissions` | Enforce directory permissions |
| `apt_verify` | Verify permissions |

## Dependencies

None.

## Example playbook

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: apt_hardening
      vars:
        apt_unattended_upgrades_auto_reboot_time: "04:00"
        apt_unattended_upgrades_mail: "admin@example.com"
```

```bash
ansible-playbook playbooks/security-hardening.yml --tags apt
ansible-playbook playbooks/security-hardening.yml --tags apt --check --diff
```

## License

MIT-0
