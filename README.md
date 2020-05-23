# Role Name
Installs and configures Image Builder as a build node for creating Fedora and Red Hat Enterprise Linux OS images. Note, these are not container images, but rather complete OS images that could be deployed in cloud, virtual, and bare metal environments.


## Requirements
1. RHEL/CentOS 7.x depend on the Extras repository being enabled.
2. If running this against the local node, RHEL/CentOS will need the Ansible repo enabled.
3. Relies on linux-system-roles.cockpit to setup dependencies for the graphical frontend interface.
4. Optional considerations include:
    - using linux-system-roles.storage to prep a dedicated filesystem for storing the image artifacts.
    - using linux-system-roles.firewall to make the Web Console available remotely.

## Role Variables
By default, the core packages for the Image Builder CLI and build service will be installed:
```yaml
  vars:
    ib_packages: [default, minimal, full]
    ib_daemon: [lorax-composer, osbuild-composer]
```
  - minimal (CLI plus one or the other backend engine, not both)
    - lorax-composer	# backend build engine and API, based on lorax from the classic Anaconda installer
    - osbuild-composer	# nextgen backend engine and API
    - composer-cli	# a command line interface
  - default (include GUI from Cockpit)
    - cockpit-composer	# a graphical front end within the Cockpit Web Console
  - full (same as default today, may change in the future)
    - cockpit-composer	# a graphical front end within the Cockpit Web Console

## Example Playbooks
The most simple example.
```yaml
---
- hosts: fedora, rhel7, rhel8
  become: yes
  roles:
    - linux-system-roles.image_builder
```

A more advanced example which can configure optional storage and firewall.
```yaml
- hosts: fedora, rhel7, rhel8
  become: yes

# Assumes you are using RHEL System Roles
#   yum install --enablerepo=rhel-7-server-ansible-2-rpms rhel-system-roles ansible
#
# Assumes roles from galaxy
#   ansible-galaxy install linux-system-roles.storage
#   ansible-galaxy install linux-system-roles.firewall
#   ansible-galaxy install linux-system-roles.cockpit

  vars:
    USE_FIREWALL: 1
    CONFIG_STORAGE: 0

  tasks:
    # Optional example to configure a second disk for your node to contain
    # the OS image artifacts created.  Typical minimal images are ~ 2 GB each.
    # Another option is to simply link to a NAS share.
    - name: Configure storage for Image Builder
      include_role:
        name: linux-system-roles.storage
      vars:
        use_partitions: false
        storage_pools:
          - name: image_builder
            disks: ['']  # something like vdb
            # type: lvm
            state: present
            volumes:
              - name: composer
                size: "20G"     # estimate depending on how many images are created
                # type: lvm
                # fs_type: xfs
                fs_label: "imgbldr"
                mount_point: '/var/lib/lorax/composer'
      when: CONFIG_STORAGE

  # Optionally use the Cockpit role to install the full suite,
  # else the Image Builder role will install the minimal dependencies for GUI.
  - name: Install and enable RHEL/Fedora Web Console (Cockpit) service
    include_role:
      name: linux-system-roles.cockpit
    vars:
      cockpit_packages: "full"   # or "minimal" or "default"

  - name: Install and enable RHEL/Fedora Image Builder service
    include_role:
      name: linux-system-roles.image_builder
    vars:
      ib_packages: "full"   # or "minimal" or "default"

  - name: Configure Firewall for Web Console
    include_role:
      name: linux-system-roles.firewall
    vars:
      firewall:
        service: cockpit
        state: enabled
    when: USE_FIREWALL
```
## License
GPLv3
