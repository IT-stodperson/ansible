# login_banners

Deploys a static login banner to `/etc/issue` (local console) and
`/etc/issue.net` (SSH pre-authentication), and optionally removes the default
Debian dynamic MOTD files.

## Requirements

- Ansible ≥ 2.20
- Debian 13 (Trixie)
- `become: true`

## Role variables

All variables are defined in `defaults/main.yml` and can be overridden.

| Variable | Default | Description |
|---|---|---|
| `login_banners_src` | `banner.txt` | Source file in the role's `files/` directory |
| `login_banners_remove_motd` | `true` | Whether to remove default MOTD files |
| `login_banners_motd_files` | `[/etc/motd, /etc/update-motd.d/10-uname]` | Files to remove when `login_banners_remove_motd` is `true` |

## Tags

| Tag | Tasks |
|---|---|
| `login_banners` | All tasks in this role |
| `login_banners_banners` | Deploy `/etc/issue` and `/etc/issue.net` |
| `login_banners_motd` | Remove static MOTD files |

## Dependencies

None.

## Example playbook

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: login_banners
      vars:
        login_banners_remove_motd: true
```

```bash
ansible-playbook playbooks/general-settings.yml --tags login_banners
```

## License

MIT-0
