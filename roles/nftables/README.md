#SPDX-License-Identifier: MIT-0
# nftables

Manage the nftables firewall on Debian-based servers. The base template is
always deployed, providing common hardening rules (early connection-state
filtering, ICMP limits, mangle/prerouting defrag, and a REJECT-for-internals
/ DROP-for-externals verdict split). Per-host service rules are pulled in
automatically via `{% include ... ignore missing %}` when a matching snippet
exists under `templates/per_host/<hostname>/`.

## Requirements

- Ansible ≥ 2.15
- Debian 12 (Bookworm) targets
- `ansible` SSH user with sudo privileges

## Role variables

### Network interfaces

| Variable | Default | Description |
|---|---|---|
| `nft_ingress_device` | `["ens18"]` | List of ingress network interfaces |

### Ansible control node (always allowed SSH)

| Variable | Default | Description |
|---|---|---|
| `nft_ansible_server_v4` | `"10.0.99.22"` | IPv4 of the Ansible controller |
| `nft_ansible_server_v6` | `"2a00:5500:c00a:99:..."` | IPv6 of the Ansible controller |

### SSH allowed networks

| Variable | Default | Description |
|---|---|---|
| `nft_ssh_teacher_v4` | `["10.0.20.0/24", "192.168.20.0/24"]` | Teacher/staff IPv4 networks |
| `nft_ssh_teacher_v6` | `["2a00:5500:c00a:20::/64"]` | Teacher/staff IPv6 networks |
| `nft_ssh_student_v4` | `["10.0.30.0/24", "192.168.30.0/24"]` | Student IPv4 networks |
| `nft_ssh_student_v6` | `["2a00:5500:c00a:30::/64"]` | Student IPv6 networks |
| `nft_ssh_allowed_v4` | `{{ nft_ssh_teacher_v4 }}` | Per-host active IPv4 SSH allowlist |
| `nft_ssh_allowed_v6` | `{{ nft_ssh_teacher_v6 }}` | Per-host active IPv6 SSH allowlist |

Override `nft_ssh_allowed_v4` / `nft_ssh_allowed_v6` in `host_vars/` to
combine groups. The Ansible control node is always added automatically by
the base template — do not include it in these lists.

### Internal networks (REJECT vs DROP verdict)

| Variable | Default | Description |
|---|---|---|
| `nft_internal_nets_v4` | `[10.0.x.0/24 ranges]` | Internal IPv4 subnets that get REJECT |
| `nft_internal_nets_v6` | `[2a00:5500:c00a:x::/64 ranges]` | Internal IPv6 subnets that get REJECT |

## Per-host service rules

Create snippet templates under:

```
roles/nftables/templates/per_host/<inventory_hostname>/
```

The base template includes them automatically. No task changes needed — add
the template file and re-run the role.

## Example playbook

```yaml
- hosts: debian_servers
  roles:
    - role: nftables
      tags: nftables
```

## License

MIT-0
