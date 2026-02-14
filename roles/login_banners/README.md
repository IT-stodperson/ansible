login_banners
=============

Deploy login banners and clean up MOTD on Debian-based servers.
This role copies a static banner file to `/etc/issue` (local console) and
`/etc/issue.net` (SSH/network), and optionally removes static MOTD files
so that login output stays clean and policy-compliant.

Requirements
------------

- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).

Role Variables
--------------

All variables are defined in `defaults/main.yml` and can be overridden.

### Banner deployment

| Variable | Default | Description |
|---|---|---|
| `login_banners_src` | `banner.txt` | Source file shipped in the role's `files/` directory. Override to use a custom banner. |

### MOTD cleanup

| Variable | Default | Description |
|---|---|---|
| `login_banners_remove_motd` | `true` | Whether to remove static MOTD files |
| `login_banners_motd_files` | `[/etc/motd, /etc/update-motd.d/10-uname]` | Files to remove when `login_banners_remove_motd` is true |

### Design note

`banner.txt` is deliberately kept as a **static file** (not a Jinja2
template) because its content is displayed verbatim to users on
SSH/console login. An Ansible-managed header would be visible to every
connecting user.

Dependencies
------------

None.

Use Cases
---------

### 1. Apply all login banner settings with defaults

```yaml
- hosts: debian_servers
  become: true
  roles:
    - login_banners
```

```bash
ansible-playbook -i inventory site.yml
```

### 2. Use a custom banner file

Place your own banner in the role's `files/` directory (or a path
resolvable by `ansible.builtin.copy`), then override the variable:

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: login_banners
      login_banners_src: custom_banner.txt
```

### 3. Keep the default MOTD

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: login_banners
      login_banners_remove_motd: false
```

### 4. Remove additional MOTD scripts

```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: login_banners
      login_banners_motd_files:
        - /etc/motd
        - /etc/update-motd.d/10-uname
        - /etc/update-motd.d/50-landscape-sysinfo
```

### 5. Run only a specific subsystem using tags

Deploy only the login banners (skip MOTD cleanup):

```bash
ansible-playbook -i inventory site.yml --tags login_banners_deploy
```

Remove only the MOTD files (skip banner deployment):

```bash
ansible-playbook -i inventory site.yml --tags login_banners_motd
```

Available tags: `login_banners`, `login_banners_deploy`,
`login_banners_motd`.

### 6. Run against a single host

```bash
ansible-playbook -i inventory site.yml --limit webserver01
```

### 7. Dry-run (check mode)

Preview changes without modifying the system:

```bash
ansible-playbook -i inventory site.yml --check --diff
```

### 8. Use in a larger playbook with other hardening roles

```yaml
- hosts: debian_servers
  become: true
  roles:
    - common
    - login_banners
    - accounts_hardening
    - pam_hardening
    - ssh_hardening
```

### 9. Set variables per environment in group_vars

```yaml
# group_vars/production.yml
login_banners_remove_motd: true

# group_vars/development.yml
login_banners_remove_motd: false
```

License
-------

MIT

Author Information
------------------

Tobias Svenblad / IT-stodperson
