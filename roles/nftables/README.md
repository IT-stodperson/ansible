nftables
========

Manage nftables firewall on Debian-based servers. Deploys a per-host nftables
configuration when a matching template exists, otherwise falls back to a
secure base ruleset.

Template Lookup
---------------

The role checks for templates in this order:

1. `templates/per_host/{{ inventory_hostname }}.conf.j2` — host-specific
2. `templates/base.conf.j2` — default ruleset (SSH-only, Teacher networks)

To add firewall rules for a new host, create a Jinja2 template at
`roles/nftables/templates/per_host/<hostname>.conf.j2`.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

SSH allowed networks are split into **Teacher** (staff/management) and
**Student** groups. Override these in `group_vars` or `host_vars`.

| Variable                | Default                                              | Description                  |
|-------------------------|------------------------------------------------------|------------------------------|
| `nft_ssh_teacher_v4`    | `["10.0.20.0/24", "192.168.20.0/24"]`                | Teacher IPv4 SSH networks (incl. VPN)  |
| `nft_ssh_teacher_v6`    | `["2a00:5500:c00a:20::/64"]`                         | Teacher IPv6 SSH networks              |
| `nft_ssh_student_v4`    | `["10.0.30.0/24", "192.168.30.0/24"]`                | Student IPv4 SSH networks (incl. VPN)  |
| `nft_ssh_student_v6`    | `["2a00:5500:c00a:30::/64"]`                         | Student IPv6 SSH networks              |

Templates can reference these variables to build the SSH ACL sets. For example:

- **Teacher only:** `{{ nft_ssh_teacher_v4 | join(', ') }}`
- **Teacher + Student:** `{{ (nft_ssh_teacher_v4 + nft_ssh_student_v4) | join(', ') }}`

Existing Host Templates
-----------------------

| Template                          | SSH access          | Extra ports  |
|-----------------------------------|---------------------|--------------|
| `per_host/webftp.its.ax.conf.j2`  | Teacher + Student   | 80, 443      |

Dependencies
------------

None.

Use Cases
---------

### 1. Deploy nftables to all servers (host-specific where available)

```yaml
- hosts: debian_servers
  become: true
  roles:
    - nftables
```

```bash
ansible-playbook playbooks/nftables-hardening.yml
```

### 2. Deploy to a single host

```bash
ansible-playbook playbooks/nftables-hardening.yml --limit webftp.its.ax
```

### 3. Dry-run (check mode)

```bash
ansible-playbook playbooks/nftables-hardening.yml --check --diff
```

### 4. Add a new host-specific template

1. Copy the base template or an existing per-host template:
   ```bash
   cp roles/nftables/templates/base.conf.j2 \
      roles/nftables/templates/per_host/newhost.its.ax.conf.j2
   ```
2. Edit the new template to add host-specific rules (extra ports, different
   SSH network sets, etc.).
3. Run the playbook with `--limit newhost.its.ax`.

### 5. Use in the master hardening playbook

```yaml
- hosts: debian_servers
  become: true
  roles:
    - kernel_hardening
    - nftables
    - ssh_hardening
    - logging_auditing
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
