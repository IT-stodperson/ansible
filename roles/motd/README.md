motd
====

Deploy dynamic MOTD infrastructure on Debian-based servers. This role
sets up a cache update script for package update counts, a systemd
timer to refresh the cache periodically, an APT hook to trigger cache
updates after package operations, and supports deploying host-specific
MOTD scripts.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

### Cache timer settings

| Variable | Default | Description |
|---|---|---|
| `motd_cache_timer_boot_delay` | `2min` | Delay after boot before the first cache update |
| `motd_cache_timer_interval` | `6h` | Interval between periodic cache updates |

### MOTD script deployment

| Variable | Default | Description |
|---|---|---|
| `motd_script` | *(undefined)* | Name of a host-specific MOTD script to deploy from `templates/scripts/<name>.j2`. Only used via the `motd-deploy` playbook. |

### What the role configures (non-variable)

- `/usr/local/bin/update-motd-cache.sh` — cache update script
- `/etc/systemd/system/motd-update-cache.service` — systemd oneshot service
- `/etc/systemd/system/motd-update-cache.timer` — systemd timer (boot delay + interval)
- `/etc/apt/apt.conf.d/99motd-cache` — APT hook to refresh cache after installs

Dependencies
------------

None.

Use Cases
---------

### 1. Deploy MOTD cache infrastructure with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - motd
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Override cache timer settings

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: motd
      motd_cache_timer_boot_delay: "5min"
      motd_cache_timer_interval: "12h"
```

### 3. Deploy a host-specific MOTD script

```bash
ansible-playbook playbooks/motd-deploy.yml --limit wazuh.its.ax \
  -e motd_script=10-custom-wazuh
```

### 4. Run only a specific subsystem using tags

Deploy only the cache infrastructure:

```bash
ansible-playbook -i inventory site.yml --tags motd_cache
```

Available tags: `motd`, `motd_cache`.

### 5. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 6. Dry-run (check mode)

Preview changes without modifying the system:

```bash
ansible-playbook -i inventory site.yml --check --diff
```

### 7. Use in a larger playbook with other roles

```yaml
- hosts: debian_servers
  become: true
  roles:
    - login_banners
    - motd
    - accounts_hardening
    - ssh_hardening
```

### 8. Set variables per environment in group_vars

```yaml
# group_vars/production.yml
motd_cache_timer_boot_delay: "2min"
motd_cache_timer_interval: "6h"

# group_vars/development.yml
motd_cache_timer_boot_delay: "5min"
motd_cache_timer_interval: "12h"
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
