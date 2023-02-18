Ansible Role: Proxmox VE NVIDIA Passthrough Config
=========

![Ansible Lint](https://github.com/simoncaron/ansible-role-pve_nvidia_passthrough/actions/workflows/lint.yml/badge.svg)
[![Ansible Role](https://img.shields.io/ansible/role/61478.svg)](https://galaxy.ansible.com/simoncaron/pve_nvidia_passthrough)
[![Ansible Role Downloads](https://img.shields.io/ansible/role/d/61478.svg)](https://galaxy.ansible.com/simoncaron/pve_nvidia_passthrough)

An Ansible Role that configures NVIDIA drivers on Proxmox VE 7.x for passthrough to VMs and LXCs.

This role was tested on Proxmox VE 7.3, on LXC containers based on Ubuntu 22.04 template.

Requirements
------------

None.

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

    pve_nvidia_passthrough_driver_version: ""
    pve_nvidia_passthrough_initramfs_update_options: -k all -u

The value `pve_nvidia_passthrough_driver_version` is required and should contain the desired driver version. You can use the official wizard to identify the latest version available for your GPU: https://www.nvidia.com/Download/index.aspx.

The key `pve_nvidia_passthrough_initramfs_update_options` allows to configure the options of the initramfs command when modules are added.

By default, the role installs/configures the driver for the Proxmox Host. To configure the driver for an LXC guest, the key `pve_nvidia_passthrough_install_mode` should be set to `guest`.

Dependencies
------------

Before running the role on LXC guests, you will need to set the appropriate lxc flags in your container .conf file.

Example, for the following entries in `/dev`:
```
root@pve01:/etc/pve/lxc# ls -l /dev/nvidia*
crw-rw-rw- 1 root root 195,   0 Feb 13 21:15 /dev/nvidia0
crw-rw-rw- 1 root root 195, 255 Feb 13 21:15 /dev/nvidiactl
crw-rw-rw- 1 root root 195, 254 Feb 13 21:15 /dev/nvidia-modeset
crw-rw-rw- 1 root root 511,   0 Feb 13 21:15 /dev/nvidia-uvm
crw-rw-rw- 1 root root 511,   1 Feb 13 21:15 /dev/nvidia-uvm-tools
```

You must add the following lines to the LXC guest .conf file (in /etc/pve/lxc/<id>.conf)
```
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 508:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
lxc.mount.entry: dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
```

After the lines are added, you must reboot the LXC. You can run this role with `pve_nvidia_passthrough_install_mode` set to `guest` to configure the guest.

See these resources for additional information:
- https://jocke.no/2022/02/23/plex-gpu-transcoding-in-docker-on-lxc-on-proxmox/
- https://theorangeone.net/posts/lxc-nvidia-gpu-passthrough/

Example Playbook
----------------

    - hosts: localhost

      vars:
        pve_nvidia_passthrough_driver_version: "525.89.02"

      roles:
        - simoncaron.pve_nvidia_passthrough

License
-------

MIT

Author Information
------------------

This role was created in 2023 by [Simon Caron](https://simoncaron.com/).
