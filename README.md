Ansible Role: Proxmox VE NVIDIA Passthrough Config
=========

![Ansible Lint](https://github.com/simoncaron/ansible-role-pve_nvidia_passthrough/actions/workflows/lint.yml/badge.svg)
[![Ansible Role](https://img.shields.io/ansible/role/60890.svg)](https://galaxy.ansible.com/simoncaron/pve_nvidia_passthrough)
[![Ansible Role Downloads](https://img.shields.io/ansible/role/d/60890.svg)](https://galaxy.ansible.com/simoncaron/pve_nvidia_passthrough)

An Ansible Role that configures NVIDIA drivers on Proxmox VE 7.x for passthrough to VMs and LXCs.

This role was tested on Proxmox VE 7.3.

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

Dependencies
------------

None.

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
