# locale_settings

Configures the system locale on Debian servers: generates required locale
definitions via `locale-gen`, then sets the system default locale via
`update-locale` (writing `/etc/default/locale`). Changes are only applied
when the current locale differs from the desired configuration.

## Requirements

- Ansible ≥ 2.15
- Debian 12 (Bookworm)
- `become: true`
- Collection: `community.general` (for `community.general.locale_gen`)

Install the collection:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Role variables

All variables are defined in `defaults/main.yml` and can be overridden.

### Locale generation

| Variable | Default | Description |
|---|---|---|
| `locale_generate` | `[en_US.UTF-8, sv_FI.UTF-8]` | Locales to generate via `locale-gen` |

### System default locale (`/etc/default/locale`)

| Variable | Default | Description |
|---|---|---|
| `locale_lang` | `en_US.UTF-8` | `LANG` — fallback for any `LC_*` variable not explicitly set |
| `locale_lc_address` | `sv_FI.UTF-8` | `LC_ADDRESS` override (empty string to omit) |
| `locale_lc_collate` | `sv_FI.UTF-8` | `LC_COLLATE` override |
| `locale_lc_ctype` | `sv_FI.UTF-8` | `LC_CTYPE` override |
| `locale_lc_identification` | `sv_FI.UTF-8` | `LC_IDENTIFICATION` override |
| `locale_lc_measurement` | `sv_FI.UTF-8` | `LC_MEASUREMENT` override |
| `locale_lc_monetary` | `sv_FI.UTF-8` | `LC_MONETARY` override |
| `locale_lc_name` | `sv_FI.UTF-8` | `LC_NAME` override |
| `locale_lc_numeric` | `sv_FI.UTF-8` | `LC_NUMERIC` override |
| `locale_lc_paper` | `sv_FI.UTF-8` | `LC_PAPER` override |
| `locale_lc_telephone` | `sv_FI.UTF-8` | `LC_TELEPHONE` override |
| `locale_lc_time` | `sv_FI.UTF-8` | `LC_TIME` override |

Set any `locale_lc_*` variable to `""` (empty string) to omit it from
`update-locale`, letting it fall back to `LANG`.

## Tags

| Tag | Tasks |
|---|---|
| `locale` | All tasks in this role |

## Dependencies

None.

## Example playbook

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: locale_settings
      vars:
        locale_lang: en_US.UTF-8
        locale_lc_time: fi_FI.UTF-8
        locale_lc_monetary: fi_FI.UTF-8
```

```bash
ansible-playbook playbooks/general-settings.yml --tags locale
ansible-playbook playbooks/general-settings.yml --tags locale --check --diff
```

## License

MIT-0
