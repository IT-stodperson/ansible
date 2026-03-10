# kanidm

Installs and configures the Kanidm Unix daemon (`kanidm-unixd`) for centralised
identity resolution and PAM authentication via a Kanidm server. Handles PPA
installation, daemon configuration, NSSwitch integration, and verifies the
daemon is online after deployment.

## Requirements

- Ansible ≥ 2.15
- Debian 12 (Bookworm)
- `become: true`
- `vault_kanidm_unixd_token` must be defined in Ansible Vault

## Role variables

All variables are defined in `defaults/main.yml` and can be overridden.

### Server connection

| Variable | Default | Description |
|---|---|---|
| `kanidm_uri` | `https://kanidm.its.ax` | URI of the Kanidm server |
| `kanidm_verify_ca` | `true` | Whether to verify the Kanidm server CA certificate |

### Unix daemon

| Variable | Default | Description |
|---|---|---|
| `kanidm_default_shell` | `/bin/bash` | Default shell for Kanidm-provisioned users |
| `kanidm_home_alias` | `name` | How to derive the home directory name |
| `kanidm_uid_attr_map` | `name` | Attribute used for UID mapping |
| `kanidm_gid_attr_map` | `name` | Attribute used for GID mapping |
| `kanidm_use_etc_skel` | `true` | Whether to use `/etc/skel` for new home directories |
| `kanidm_pam_allowed_login_groups` | `[teachers, ansible]` | Kanidm POSIX groups allowed to log in via PAM |

### Group mapping

| Variable | Default | Description |
|---|---|---|
| `kanidm_group_mappings` | `[{local: sudo, with: teachers}]` | Map Kanidm groups to local system groups |

### NSSwitch

| Variable | Default | Description |
|---|---|---|
| `kanidm_configure_nsswitch` | `true` | Whether to configure `nsswitch.conf` |
| `kanidm_nsswitch_passwd` | `"kanidm compat"` | `passwd` source order |
| `kanidm_nsswitch_group` | `"kanidm compat"` | `group` source order |

### Secrets

| Variable | Description |
|---|---|
| `vault_kanidm_unixd_token` | **Required.** Service account token — define in Ansible Vault |
| `kanidm_unixd_token` | Defaults to `{{ vault_kanidm_unixd_token }}` |

## Tags

| Tag | Tasks |
|---|---|
| `kanidm` | All tasks in this role |
| `kanidm_install` | PPA and package installation |
| `kanidm_configure` | Daemon configuration files |
| `kanidm_nsswitch` | NSSwitch configuration |
| `kanidm_services` | Service enable and start |
| `kanidm_verify` | Daemon connectivity verification |

## Dependencies

None.

## Example playbook

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: kanidm
      vars:
        kanidm_uri: "https://auth.example.com"
        kanidm_pam_allowed_login_groups:
          - admins
          - ansible
```

Define the service token in `inventory/group_vars/all/vault.yml`:

```yaml
vault_kanidm_unixd_token: "your-service-token"
```

```bash
ansible-playbook playbooks/bootstrap.yml
ansible-playbook playbooks/security-hardening.yml --tags kanidm
```

## License

MIT-0
