nftables
========

Manage nftables firewall on Debian-based servers. The base template is
always deployed, providing common hardening (early filtering, mangle,
SSH rate limiting, Ansible server access). Per-host service rules are
layered on top via snippet includes.

Template Architecture
---------------------

A single `base.conf.j2` template is deployed to every host. It contains:

- **NETDEV early_filter** — DDoS mitigation at ingress (before conntrack)
- **inet mangle** — TCP sanity, scan signatures, spoofed loopback
- **inet filter** — SSH access control with rate limiting, ICMP, DHCP,
  hybrid REJECT/DROP final verdict

The base template includes per-host snippets from
`templates/per_host/<hostname>_*.j2` at five extension points:

| Snippet suffix    | Purpose                                          |
|-------------------|--------------------------------------------------|
| `_defines.j2`     | Extra nft `define` directives (before tables)    |
| `_sets.j2`        | Service-specific sets and rate-limit sets         |
| `_chains.j2`      | Service-specific guard chains                    |
| `_inbound_v4.j2`  | IPv4 inbound rules for the service               |
| `_inbound_v6.j2`  | IPv6 inbound rules for the service               |

All snippets use `ignore missing`, so hosts without snippets get only the
base ruleset.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

### Ansible Control Node

The Ansible server is always granted SSH access on every host.

| Variable                | Default                                              | Description                  |
|-------------------------|------------------------------------------------------|------------------------------|
| `nft_ansible_server_v4` | `"10.0.99.22"`                                       | Ansible server IPv4 address  |
| `nft_ansible_server_v6` | `"2a00:5500:c00a:99:be24:11ff:feb3:33d0"`            | Ansible server IPv6 address  |

### SSH Networks

SSH allowed networks are split into **Teacher** (staff/management) and
**Student** groups. The `nft_ssh_allowed_v4/v6` variables control which
groups are added to the SSH ACL. Override in `host_vars` to combine groups.

| Variable                | Default                                              | Description                  |
|-------------------------|------------------------------------------------------|------------------------------|
| `nft_ssh_teacher_v4`    | `["10.0.20.0/24", "192.168.20.0/24"]`                | Teacher IPv4 SSH networks    |
| `nft_ssh_teacher_v6`    | `["2a00:5500:c00a:20::/64"]`                         | Teacher IPv6 SSH networks    |
| `nft_ssh_student_v4`    | `["10.0.30.0/24", "192.168.30.0/24"]`                | Student IPv4 SSH networks    |
| `nft_ssh_student_v6`    | `["2a00:5500:c00a:30::/64"]`                         | Student IPv6 SSH networks    |
| `nft_ssh_allowed_v4`    | `"{{ nft_ssh_teacher_v4 }}"`                         | SSH-allowed IPv4 (override per host) |
| `nft_ssh_allowed_v6`    | `"{{ nft_ssh_teacher_v6 }}"`                         | SSH-allowed IPv6 (override per host) |

Example host_vars override for Teacher + Student SSH access:

```yaml
# inventory/host_vars/webftp.its.ax.yml
nft_ssh_allowed_v4: "{{ nft_ssh_teacher_v4 + nft_ssh_student_v4 }}"
nft_ssh_allowed_v6: "{{ nft_ssh_teacher_v6 + nft_ssh_student_v6 }}"
```

Existing Host Snippets
----------------------

| Host                    | SSH access          | Extra ports  |
|-------------------------|---------------------|--------------|
| `webftp.its.ax`         | Teacher + Student   | 80, 443      |
| `mariadb.its.ax`        | Teacher (default)   | 3306         |

Dependencies
------------

None.

Use Cases
---------

### 1. Deploy nftables to all servers

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

### 4. Add service rules for a new host

1. Create snippet files under `roles/nftables/templates/per_host/`:
   ```
   newhost.its.ax_defines.j2      # optional: extra nft defines
   newhost.its.ax_sets.j2         # service sets and rate-limit sets
   newhost.its.ax_chains.j2       # service guard chains
   newhost.its.ax_inbound_v4.j2   # IPv4 inbound rules
   newhost.its.ax_inbound_v6.j2   # IPv6 inbound rules
   ```
2. If the host needs non-default SSH access, add an override in
   `inventory/host_vars/newhost.its.ax.yml`:
   ```yaml
   nft_ssh_allowed_v4: "{{ nft_ssh_teacher_v4 + nft_ssh_student_v4 }}"
   nft_ssh_allowed_v6: "{{ nft_ssh_teacher_v6 + nft_ssh_student_v6 }}"
   ```
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
