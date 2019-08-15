# image_builder

Installs and configures Image Builder as a build node for creating OS images.

By default, the core packages for the Image Builder CLI and build service will
be installed (see `vars/main.yml`):

* `lorax-composer`
* `composer-cli`
* `cockpit-composer` (if `ib_with_gui` is `true`, see [Role Variables](#role-variables))

## Requirements

If running on RHEL/CentOS 7, please ensure the Extras repository is enabled as
the role does not allow handling of that yet.

## Role Variables

Available variables per distribution are listed below, along with default
values (see `defaults/main.yml`):

### `ib_packages`

Specify the list of packages that should be installed additionally to the core
Image Builder packages. By default, no additional packages are installed.

### `ib_gui_packages`

Specify the list of packages that should be installed additionally to the core
Image Builder GUI packages. This setting has an effect only if `ib_with_gui` is
set to `true`. By default, no additional GUI packages are installed.

### `ib_with_gui`

Specify whether or not to install GUI provided via Cockpit Web Console.
Disabled by default.

### `ib_enablerepo`

Specify the repository to enable in case non-standard repository mirror names
are used. The default value of this role variable depends on the used Linux
distribution. The table below summarize possible default values per Linux
distribution:

| Linux Distribution | `ib_enablerepo`'s Default Value |
| ------------------ | ------------------------------- |
| Fedora | `fedora <Fedora version number> - <architecture>` |
| Red Hat Enterprise Linux 7 | `rhel-7-server-extras-rpms` |
| Red Hat Enterprise Linux 8 | `rhel-8-for-<architecture>-appstream-rpms` |
| other | empty string |

To disable this feature, set the `ib_enablerepo` to `""` (empty string).

The value of `ib_enablerepo` has effect only on Linux distributions with yum
based packaging.

### `ib_service`

Specify the name of service that provides Image Builder. The default name is
`lorax-composer`.

### `ib_cockpit_service`

Specify the name of the Cockpit service. The default name is `cockpit`.

### `ib_restart_delay`

Specify the time interval in seconds between stopping and starting the services
when it is desired. Defaultly, this is set to 3 seconds.

## Dependencies

1. RHEL/CentOS 7.x depend on the Extras repository being enabled.
1. Other considerations include
   * using `linux-system-roles.storage` to prep a dedicated filesystem for
     build space
   * using `linux-system-roles.firewall` to make the Web Console available
     remotely

## Example Playbook

Install and enable Image Builder with default features:

```yaml
- hosts: all
  become: yes

  roles:
    - linux-system-roles.image_builder
```

Install and enable Image Builder together with beautiful graphical interface
via Cockpit Web Console:

```yaml
- hosts: all
  become: yes

  vars:
    ib_with_gui: true

  roles:
    - linux-system-roles.image_builder
```

## License

GPLv3
