user_management
===============
Manage user accounts on Debian-based servers. This role creates a shared
group, provisions user accounts with home directories, and deploys SSH
authorized keys for key-based authentication.
Requirements
------------
- Ansible >= 2.20
- Target must be a Debian-based system (Debian bookworm/trixie, Ubuntu jammy/noble).
- The role requires root privileges (`become: true`).
Role Variables
--------------
All variables are defined in `defaults/main.yml` and can be overridden.
### Group
| Variable | Default | Description |
|---|---|---|
| `user_management_group` | `teachers` | Group that managed users will be added to. |
### Users
| Variable | Default | Description |
|---|---|---|
| `user_management_users` | *(see below)* | List of user dicts to manage. Each entry requires `name` and `ssh_key`. Optional: `shell` (default `/bin/bash`), `comment` (GECOS / full name). |
Default users:
```yaml
user_management_users:
  - name: tobias
    ssh_key: "ssh-ed25519 AAAAC3… tobias"
    comment: Tobias
    shell: /bin/bash
  - name: kjell
    ssh_key: "ssh-ed25519 AAAAC3… kjell"
    comment: Kjell
    shell: /bin/bash
```
### What the role configures
- Creates the group specified by `user_management_group`
- Creates each user, adds them to the group, sets shell and comment
- Ensures `~/.ssh` directory exists with mode `0700`
- Deploys `authorized_keys` for each user from their `ssh_key` value
Dependencies
------------
None.
Use Cases
---------
### 1. Apply user management with defaults
```yaml
- hosts: debian_servers
  become: true
  roles:
    - user_management
```
```bash
ansible-playbook -i inventory site.yml
```
### 2. Change the shared group
```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: user_management
      user_management_group: staff
```
### 3. Add or replace the user list
```yaml
- hosts: debian_servers
  become: true
  roles:
    - role: user_management
      user_management_group: developers
      user_management_users:
        - name: alice
          ssh_key: "ssh-ed25519 AAAAC3… alice"
          comment: Alice Andersson
        - name: bob
          ssh_key: "ssh-ed25519 AAAAC3… bob"
          comment: Bob Bengtsson
          shell: /bin/zsh
```
### 4. Run against a single host
```bash
ansible-playbook -i inventory site.yml --limit webserver01
```
### 5. Override variables from the command line
```bash
ansible-playbook -i inventory site.yml \
  -e user_management_group=staff
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
    - user_management
    - accounts_hardening
    - sudo_hardening
    - ssh_hardening
```
### 8. Set variables per environment in group_vars
```yaml
# group_vars/production.yml
user_management_group: teachers
user_management_users:
  - name: tobias
    ssh_key: "ssh-ed25519 AAAAC3… tobias"
    comment: Tobias
# group_vars/development.yml
user_management_group: developers
user_management_users:
  - name: testuser
    ssh_key: "ssh-ed25519 AAAAC3… testuser"
    comment: Test User
```
License
-------
MIT
Author Information
------------------
Tobias Svenblad / IT-stodperson
