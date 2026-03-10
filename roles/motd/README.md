motd
====

Deploy dynamic MOTD infrastructure on Debian-based servers. This role
deploys a unified MOTD script with a common base (system info, uptime,
memory, disk, security) and per-host extensions (services, connections,
usage guides). Per-host sections are included automatically based on
`inventory_hostname`, similar to the nftables role's snippet architecture.

The role also sets up a cache update script for package update counts,
a systemd timer to refresh the cache periodically, and an APT hook to
trigger cache updates after package operations.

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

### MOTD display settings

| Variable | Default | Description |
|---|---|---|
| `motd_role` | `Server` | Server role label shown in the MOTD header. Override in `host_vars/` per server. |

### What the role configures (non-variable)

- `/etc/update-motd.d/10-sysinfo` — combined base + per-host MOTD script
- `/usr/local/bin/update-motd-cache.sh` — cache update script
- `/etc/systemd/system/motd-update-cache.service` — systemd oneshot service
- `/etc/systemd/system/motd-update-cache.timer` — systemd timer (boot delay + interval)
- `/etc/apt/apt.conf.d/99motd-cache` — APT hook to refresh cache after installs

Architecture
------------

The MOTD uses a base + per-host snippet design (like the nftables role):

```
templates/scripts/
├── 10-sysinfo.j2                  # Base template (all servers)
└── per_host/
    ├── ansible.its.ax_init.j2     # Per-host background jobs
    ├── ansible.its.ax_output.j2   # Per-host output sections
    ├── wazuh.its.ax_init.j2
    ├── wazuh.its.ax_process.j2    # Per-host result processing
    ├── wazuh.its.ax_output.j2
    └── ...
```

**Three include points** in the base template:

1. `_init.j2` — Start host-specific background jobs and collect data
2. `_process.j2` — Wait for background jobs and process results
3. `_output.j2` — Display host-specific sections (services, connections, etc.)

All includes use `ignore missing`, so hosts without snippets get only the
base MOTD. To add a new server, create its snippet files — no task or
playbook changes needed.

Dependencies
------------

None.

Use Cases
---------

### 1. Deploy MOTD to all servers (auto-selects per-host sections)

```bash
ansible-playbook playbooks/motd-deploy.yml
```

### 2. Deploy to a single host

```bash
ansible-playbook playbooks/motd-deploy.yml --limit wazuh.its.ax
```

### 3. Override cache timer settings

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: motd
      motd_cache_timer_boot_delay: "5min"
      motd_cache_timer_interval: "12h"
```

### 4. Run only a specific subsystem using tags

```bash
# Cache infrastructure only
ansible-playbook playbooks/motd-deploy.yml --tags motd_cache

# MOTD script only
ansible-playbook playbooks/motd-deploy.yml --tags motd_script
```

Available tags: `motd`, `motd_cache`, `motd_script`.

### 5. Dry-run (check mode)

```bash
ansible-playbook playbooks/motd-deploy.yml --check --diff
```

### 6. Add MOTD for a new server

1. Set `motd_role` in `inventory/host_vars/<hostname>.yml`
2. Create snippet files in `templates/scripts/per_host/`:
   - `<hostname>_init.j2` — background jobs / data collection
   - `<hostname>_process.j2` — wait for jobs / process results
   - `<hostname>_output.j2` — display service-specific sections
3. Run: `ansible-playbook playbooks/motd-deploy.yml --limit <hostname>`

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
