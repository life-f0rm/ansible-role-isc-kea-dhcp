# Ansible Role: ISC Kea DHCP

Ansible role to install and manage the ISC Kea DHCPv4 server (`kea-dhcp4-server`).

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
        - "t3_users"
        - "nwmgmnt"
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
      # t3_users VLAN (192.168.3.0/24)
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
      # nwmgmnt VLAN (192.168.101.0/24)
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
          - output: "/var/log/kea/kea-dhcp4.log"
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
