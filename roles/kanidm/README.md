# kanidm

Install and configure Kanidm Unix daemon (`kanidm-unixd`) for centralised
identity resolution and authentication via a Kanidm server.

## What this role does

1. Adds the Kanidm PPA and installs `kanidm-unixd`
2. Deploys `/etc/kanidm/config`, `/etc/kanidm/unixd`, and the service account token
3. Creates a systemd override for `kanidm-unixd` to load the token via `LoadCredential`
4. Configures `/etc/nsswitch.conf` with Kanidm as the primary identity source
5. Enables and starts `kanidm-unixd` and `kanidm-unixd-tasks`
6. Verifies the daemon is online and connected

## Requirements

- `vault_kanidm_unixd_token` must be defined in `inventory/group_vars/all/vault.yml`

## Role Variables

See `defaults/main.yml` for all configurable variables.
