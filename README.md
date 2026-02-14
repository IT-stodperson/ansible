# Ansible — Server Hardening & Configuration

This repository is the central source of truth for all Ansible playbooks, roles, templates, and variables used to configure and harden the servers in the **its.ax** infrastructure. It is hosted on **ansible.its.ax** and version-controlled with Git.

---

> **Important — Two ways to edit, one workflow.**
>
> Files in this repository can be edited in two places:
>
> 1. **Locally on the server** (`ansible.its.ax`) using a terminal editor.
> 2. **On GitHub** through the web interface or by cloning the repo to another machine.
>
> **No matter where you make your changes, you must always follow the Git workflow below.** Failing to pull before editing will cause merge conflicts and can overwrite other people's work. Always pull first.

---

## Repository structure

```
.
├── ansible.cfg              # Ansible configuration (user, paths, vault, SSH)
├── inventory/
│   ├── hosts.yml            # Host inventory (proxmox, debian_servers)
│   ├── group_vars/          # Group-scoped variables and vault secrets
│   └── host_vars/           # Per-host variable overrides
├── playbooks/
│   ├── security-hardening.yml
│   ├── sudo-hardening.yml
│   ├── nftables-hardening.yml
│   ├── motd-deploy.yml
│   └── templates/           # Jinja2 templates used by playbooks
└── roles/
    ├── accounts_hardening   # Password policies, shell hardening, timeouts
    ├── apparmor             # Mandatory access control profiles
    ├── apt_hardening        # Package manager lockdown
    ├── cron_hardening       # Cron access whitelist
    ├── grub_hardening       # Bootloader password & kernel parameters
    ├── kernel_hardening     # Sysctl, module blacklisting, limits
    ├── logging_auditing     # Audit daemon & syslog configuration
    ├── login_banners        # Console & SSH login banners
    ├── motd                 # Dynamic message-of-the-day scripts
    ├── nftables             # Firewall rules (nftables)
    ├── ntp                  # Time synchronisation hardening
    ├── pam_hardening        # PAM module configuration
    ├── ssh_hardening        # SSH daemon hardening
    └── sudo_hardening       # Sudoers policies & auditing
```

## Target hosts

| Group             | Hosts                                                  |
| ----------------- | ------------------------------------------------------ |
| `proxmox`         | pve1.its.ax, pve2.its.ax, pve3.its.ax                 |
| `debian_servers`  | wazuh.its.ax, webftp.its.ax, ansible.its.ax, mariadb.its.ax |

## Prerequisites

- Ansible >= 2.20
- Debian-based targets (Bookworm / Trixie, Ubuntu Jammy / Noble)
- The `ansible` SSH user with sudo privileges on all managed hosts
- The vault password file (`.vault_pass`) in the repository root

---

## Git workflow — the golden rule

Every change, whether made **on the server** or **on GitHub**, must follow this sequence:

```
pull  →  edit  →  add  →  commit  →  push
```

### Step by step

```bash
# 1. PULL — always start by getting the latest changes
sudo git pull

# 2. EDIT — make your changes to playbooks, roles, templates, etc.
sudo nano roles/ssh_hardening/defaults/main.yml   # example

# 3. ADD — stage the files you changed
sudo git add roles/ssh_hardening/defaults/main.yml

# 4. COMMIT — describe what you changed and why
sudo git commit -m "ssh_hardening: disable root login"

# 5. PUSH — send your commit to the remote repository
sudo git push -u origin main
```

### Why this order matters

| Step     | What happens if you skip it                                        |
| -------- | ------------------------------------------------------------------ |
| **pull** | Your local copy is outdated. You may overwrite someone else's changes or create merge conflicts. |
| **add**  | Your changes are not staged and will not be included in the commit. |
| **commit** | Nothing is recorded. A push without a commit sends nothing.      |
| **push** | Your changes stay local. Others (and GitHub) will not see them.    |

> **Remember:** If you edited a file on GitHub, you must run `git pull` on the server before doing anything else. If you edited on the server, you must `git push` before making further changes on GitHub. **Keep both sides in sync.**

---

## Running playbooks

Apply the full security hardening baseline:

```bash
ansible-playbook playbooks/security-hardening.yml
```

Dry-run with diff output (no changes applied):

```bash
ansible-playbook playbooks/security-hardening.yml --check --diff
```

Run a specific set of tasks using tags:

```bash
ansible-playbook playbooks/security-hardening.yml --tags accounts_shell
```

Target a single host:

```bash
ansible-playbook playbooks/security-hardening.yml --limit wazuh.its.ax
```

## License

MIT — see [LICENSE](LICENSE).
