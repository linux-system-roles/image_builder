# Role Name

Installs and configures Image Builder as a build node for creating OS images.

## Requirements

If running on RHEL/CentOS 7, please ensure the Extras repository is enabled as the role does not allow handling of that yet.

## Role Variables

By default, the core packages for the Image Builder CLI and build service will be installed (see 'vars/<distro>.yml`):
  - lorax-composer
  - composer-cli

```yaml
ib_with_gui: **false** | true
  - cockpit-composer	## Beautiful graphical interface via Cockpit Web Console
```

[To Do] Specify the repository to enable in case non-standard repository mirror names are used.
This has yet to be implemented, though the variable exists but not used.

```yaml
ib_enablerepo: "rhel-7-server-extras-rpms"  
```

## Dependencies

1. RHEL/CentOS 7.x depend on the Extras repository being enabled.
2. Other considerations include
    - using linux-system-roles.storage to prep a dedicated filesystem for build space
    - using linux-system-roles.firewall to make the Web Console available remotely

## Example Playbook

```yaml
- hosts: fedora, rhel7, rhel8
  become: yes
  roles:
    - linux-system-roles.image_builder
```

## License

GPLv3

