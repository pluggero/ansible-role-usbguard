# ansible-role-usbguard

[![CI](https://github.com/pluggero/ansible-role-usbguard/actions/workflows/ci.yml/badge.svg)](https://github.com/pluggero/ansible-role-usbguard/actions/workflows/ci.yml) [![Ansible Galaxy downloads](https://img.shields.io/ansible/role/d/pluggero/usbguard?label=Galaxy%20downloads&logo=ansible&color=%23096598)](https://galaxy.ansible.com/ui/standalone/roles/pluggero/usbguard)

Installs and configures USBGuard on ArchLinux and Debian.

## Requirements

**Collections:**

- `ansible.builtin` (included with Ansible)
- `community.general` (for ArchLinux support)

**Role dependencies:**

None.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
usbguard_install_method: "package"
```

Installation method. Only `"package"` is supported.

```yaml
usbguard_service_enabled: true
usbguard_service_state: "started"
```

Controls the systemd service state. Set `usbguard_service_enabled: false` and `usbguard_service_state: "stopped"` to disable the service.

```yaml
usbguard_generate_policy: false
```

When `true`, runs `usbguard generate-policy > /etc/usbguard/rules.conf` to capture currently attached devices as the base policy. Only executes if `rules.conf` does not already exist (idempotent). Requires all external USB devices to be disconnected before running so only internal devices are captured.

```yaml
usbguard_stable_rules: true
```

When `true` (default), strips `parent-hash` and `with-connect-type` from the generated policy via `sed`. These attributes encode USB topology and connection type, making rules fragile across hardware changes. Stripping them produces portable rules that survive port changes and machine migrations. The role also validates that manually defined `usbguard_rules` do not contain these attributes at run time.

```yaml
usbguard_daemon_config:
  RuleFile: "/etc/usbguard/rules.conf"
  RuleFolder: "/etc/usbguard/rules.d/"
  ImplicitPolicyTarget: "block"
  PresentDevicePolicy: "apply-policy"
  PresentControllerPolicy: "apply-policy"
  InsertedDevicePolicy: "apply-policy"
  RestoreControllerDeviceState: "false"
  AuthorizedDefault: "none"
  IPCAllowedUsers: "root"
  IPCAllowedGroups: ""
  IPCAccessControlFiles: "/etc/usbguard/IPCAccessControl.d/"
  DeviceManagerBackend: "uevent"
  DeviceRulesWithPort: "false"
  AuditBackend: "FileAudit"
  AuditFilePath: "/var/log/usbguard/usbguard-audit.log"
  HidePII: "false"
```

Key-value pairs rendered into `/etc/usbguard/usbguard-daemon.conf`. Override individual keys to adjust daemon behaviour.

```yaml
usbguard_rules_dir: "/etc/usbguard/rules.d"
usbguard_rules_dir_mode: "0700"
```

Path and permissions for the rules drop-in directory.

```yaml
usbguard_rules_filename: "10-rules.conf"
```

Filename used when deploying the rendered rules file to `rules.d/`. The numeric prefix controls load order within the directory.

```yaml
usbguard_rules: []
```

Inline USB rules rendered into a single file in `rules.d/`. No file is deployed when this list is empty (the default). Rules are organised into blocks, each with an optional `comment` heading (rendered as `# heading`). Individual entries can be plain rule strings or dicts with an optional per-entry `comment` (rendered as `## note`):

```yaml
usbguard_rules:
  - comment: "Location or group name"
    entries:
      - comment: "Device description"
        rule: 'allow id xxxx:xxxx serial "..." name "..." hash "..." with-interface xx:xx:xx'
      - 'allow id xxxx:xxxx serial "..." name "..." hash "..." with-interface xx:xx:xx'
  - entries:
      - 'allow id xxxx:xxxx serial "..." name "..." hash "..." with-interface xx:xx:xx'
```

```yaml
usbguard_create_manual_rules_file: true
```

When `true`, seeds a `99-manual-rules.conf` file in `rules.d/` with a header comment on first run. Ansible never overwrites this file (`force: false`), so manual edits persist across runs. Set to `false` to skip creation entirely.

```yaml
usbguard_group: "usbguard"
```

System group created by the role. Members gain USBGuard IPC access via a group ACL file in `IPCAccessControl.d/`.

```yaml
usbguard_group_ipc:
  devices: ["list"]
  policy: []
  exceptions: []
```

IPC privileges granted to `usbguard_group`. `devices` accepts `list`, `modify`, `listen`; `policy` accepts `list`, `modify`; `exceptions` accepts `listen`. Defaults to `Devices=list` so members can list blocked devices without privilege escalation.

```yaml
usbguard_users:
  - "{{ ansible_user }}"
```

Users to add to `usbguard_group`. Defaults to the Ansible connection user. Set to `[]` to manage group membership outside the role.

## Dependencies

None.

## Example Playbook

```yaml
- hosts: all
  roles:
    - role: pluggero.usbguard
      vars:
        usbguard_generate_policy: true
        usbguard_daemon_config:
          RuleFile: "/etc/usbguard/rules.conf"
          RuleFolder: "/etc/usbguard/rules.d/"
          ImplicitPolicyTarget: "block"
          PresentDevicePolicy: "apply-policy"
          PresentControllerPolicy: "apply-policy"
          InsertedDevicePolicy: "apply-policy"
          RestoreControllerDeviceState: "false"
          AuthorizedDefault: "none"
          IPCAllowedUsers: "root"
          IPCAllowedGroups: ""
          DeviceManagerBackend: "uevent"
          DeviceRulesWithPort: "false"
        usbguard_users:
          - "alice"
        usbguard_rules:
          - comment: "Office devices"
            entries:
              - 'allow id xxxx:xxxx serial "..." name "..." hash "..." with-interface xx:xx:xx'
```

## License

MIT

## Author Information

This role was created in 2026 by pluggero.
