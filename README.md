# Ansible Role: ISC Kea DHCP

Ansible role to install and manage the ISC Kea DHCPv4 server (`kea-dhcp4-server`).

[![CI](https://github.com/life-f0rm/ansible-role-isc-kea-dhcp/actions/workflows/ci.yml/badge.svg)](https://github.com/life-f0rm/ansible-role-isc-kea-dhcp/actions/workflows/ci.yml)
[![Actionlint](https://github.com/life-f0rm/ansible-role-isc-kea-dhcp/actions/workflows/actionlint.yml/badge.svg)](https://github.com/life-f0rm/ansible-role-isc-kea-dhcp/actions/workflows/actionlint.yml)
[![Security Scan](https://github.com/life-f0rm/ansible-role-isc-kea-dhcp/actions/workflows/security-scan.yml/badge.svg)](https://github.com/life-f0rm/ansible-role-isc-kea-dhcp/actions/workflows/security-scan.yml)
[![Release Please](https://github.com/life-f0rm/ansible-role-isc-kea-dhcp/actions/workflows/release-please.yml/badge.svg)](https://github.com/life-f0rm/ansible-role-isc-kea-dhcp/actions/workflows/release-please.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

The role:

- Installs Kea DHCP packages
- Creates configuration and log directories
- Renders a complete Kea DHCP4 configuration from an Ansible variable (YAML → JSON)
- Validates the configuration using `kea-dhcp4 -t`
- Ensures the Kea DHCP4 service is enabled and running

## Requirements

- Supported OS: Ubuntu (see `meta/main.yaml`)
- Minimum Ansible version: 2.10
- The role assumes the Kea packages create the Kea user and group (defaults to `_kea`).

## Role Variables

Defaults are defined in `defaults/main.yaml`:

You **must** override `kea_dhcp_config` with a full Kea DHCP4 configuration structure.
The role will convert this YAML data structure to pretty-printed JSON using `to_nice_json` and write it to `kea_dhcp_config_file`.

## Example `kea_dhcp_config`

Below is an example `kea_dhcp_config` you can place in `group_vars`/`host_vars`:

```yaml
# Complete Kea configuration in YAML (converted to JSON automatically)
kea_dhcp_config:
  Dhcp4:
    interfaces-config:
      interfaces:
        - "guests"
        - "default"
        - "priv_users"
        - "nw_mgt"
      dhcp-socket-type: "raw"
    lease-database:
      type: "memfile"
      persist: true
      lfc-interval: 0
    valid-lifetime: 3600
    renew-timer: 1800
    rebind-timer: 2700
    option-data:
      - name: "domain-name-servers"
        data: "9.9.9.9, 149.112.112.112"
      - name: "domain-name"
        data: "local"
    subnet4:
      # priv_users VLAN (192.168.3.0/24)
      - id: 3
        subnet: "192.168.3.0/24"
        pools:
          - pool: "192.168.3.101 - 192.168.3.199"
        option-data:
          - name: "routers"
            data: "192.168.3.254"
      # guests VLAN (192.168.10.0/26)
      - id: 11
        subnet: "192.168.10.0/26"
        pools:
          - pool: "192.168.10.2 - 192.168.10.60"
        option-data:
          - name: "routers"
            data: "192.168.10.62"
      # default VLAN (192.168.10.64/26)
      - id: 12
        subnet: "192.168.10.64/26"
        pools:
          - pool: "192.168.10.66 - 192.168.10.122"
        option-data:
          - name: "routers"
            data: "192.168.10.126"
      # nw-mgt VLAN (192.168.101.0/24)
      - id: 91
        subnet: "192.168.101.0/24"
        pools:
          - pool: "192.168.101.101 - 192.168.101.199"
        option-data:
          - name: "routers"
            data: "192.168.101.254"
    loggers:
      - name: "kea-dhcp4"
        output_options:
          - output: "syslog"
          - output: "stdout"
        severity: "INFO"
        debuglevel: 0
```

## Example Playbook

```yaml
- name: Configure Kea DHCPv4
  hosts: kea_servers
  become: true

  roles:
    - role: ansible-role-isc-kea-dhcp
```

Make sure `kea_dhcp_config` is defined for your hosts (e.g. in `group_vars/kea_servers.yaml`).

## License

MIT

## Development And QA

This repository uses security-first automation for validation and releases:

- CI runs `ansible-lint`, `yamllint`, and `molecule test`
- GitHub Actions workflows are pinned to immutable commit SHAs
- Dependency review runs on pull requests
- Renovate manages workflow and tooling updates
- Release Please manages changelog and release PR automation

### Local Validation

Recommended local flow:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements-dev.txt

ansible-lint .
yamllint .
molecule test
```

## Release Process

- Follow Conventional Commits (`feat:`, `fix:`, `chore:`, etc.)
- Merges to `main` update the Release Please release PR
- Merging the release PR creates the GitHub release and updates `CHANGELOG.md`

## Security Posture

- Minimal GitHub Actions token permissions are used by default
- No long-lived cloud secrets are required for CI
- Security-sensitive changes should be reviewed with extra care:
  - `.github/workflows/**`
  - `renovate.json`
  - release automation files

## Dependabot Alerts With Renovate (No Duplicate PRs)

This repository is configured for Renovate to open dependency update PRs.
To keep security visibility without duplicate PR noise, use Dependabot for alerting only:

- Enable dependency graph: GitHub repository Settings -> Security -> Dependency graph -> Enable
- Enable Dependabot alerts: GitHub repository Settings -> Security -> Dependabot alerts -> Enable
- Do not enable Dependabot version updates for this repository (do not add `.github/dependabot.yml`)
- Keep Renovate enabled so dependency update PRs continue to come from Renovate

Result: GitHub still raises advisory alerts while Renovate remains the only dependency PR bot.

## Branch And Workflow Protection Checklist

These controls are configured in GitHub repository settings (not fully in role files):

- Branch protection for `main`:
  - Require a pull request before merging
  - Require approvals (at least 1, preferably 2 for security-sensitive repos)
  - Require status checks before merging
  - Require branches to be up to date before merging
  - Restrict direct pushes to `main`
- Required status checks (after first successful runs, select exact check names):
  - `Lint`
  - `Molecule`
  - `Actionlint`
  - `Trivy FS Scan`
  - `dependency-review`
- Enable "Require review from Code Owners"
- Keep CODEOWNERS ownership for workflow files (`.github/workflows/**`) so pipeline edits require owner review

## Renovate Troubleshooting

If Renovate does not create or update PRs as expected:

- Confirm the Renovate app is installed on this repository
- Check Renovate dashboard/issues and bot logs for permission or config errors
- Validate `renovate.json` with the Renovate config validator
- Confirm no competing update bot is creating lock/conflicting PRs
- Check branch protection rules are not blocking Renovate from updating its branches
- Verify workflow action updates are not blocked by policy settings

## Notes

- [https://cloudsmith.io/~isc/repos/kea-3-0/setup/#formats-deb](https://cloudsmith.io/~isc/repos/kea-3-0/setup/#formats-deb)
- [https://kb.isc.org/docs/kea-ha-quickstart-guide](https://kb.isc.org/docs/kea-ha-quickstart-guide)
