Ansible Role: Proxmox VE NVIDIA Passthrough Config
=========

![Ansible Lint](https://github.com/simoncaron/ansible-role-pve_nvidia_passthrough/actions/workflows/lint.yml/badge.svg)
![Ansible Release](https://github.com/simoncaron/ansible-role-pve_nvidia_passthrough/actions/workflows/release.yml/badge.svg)
[![Ansible Galaxy Downloads](https://img.shields.io/badge/dynamic/json?color=blueviolet&label=Galaxy%20Downloads&query=%24.download_count&url=https%3A%2F%2Fgalaxy.ansible.com%2Fapi%2Fv1%2Froles%2F26240%2F%3Fformat%3Djson)](https://galaxy.ansible.com/ui/standalone/roles/simoncaron/pve_nvidia_passthrough/)

An Ansible Role that configures NVIDIA drivers on Proxmox VE 9.x for passthrough to VMs and LXCs.

This role was tested on Proxmox VE 9.X, on LXC containers based on Debian 12 template.

Requirements
------------

None.

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

    pve_nvidia_passthrough_driver_version: ""
    pve_nvidia_passthrough_driver_install_opts: --dkms -s
    pve_nvidia_passthrough_initramfs_update_options: -k all -u
    pve_nvidia_passthrough_install_container_runtime: true

The value `pve_nvidia_passthrough_driver_version` is required and should contain the desired driver version. You can use the official wizard to identify the latest version available for your GPU: https://www.nvidia.com/Download/index.aspx.

The key `pve_nvidia_passthrough_driver_install_opts` allows to configure the options passed to the NVIDIA driver installer.

The key `pve_nvidia_passthrough_initramfs_update_options` allows to configure the options of the initramfs command when modules are added.

The key `pve_nvidia_passthrough_install_container_runtime` controls whether to install and configure the NVIDIA Container Toolkit for Docker/container support (default: true).

Dependencies
------------

None.

Using NVIDIA GPUs in LXC Containers
------------------------------------

To use NVIDIA GPUs in LXC containers, you have two options for configuring device access:

### Option 1: Automatic Injection (Recommended)

Add the following lines to your LXC container .conf file (in `/etc/pve/lxc/<id>.conf`):

```
lxc.environment: NVIDIA_VISIBLE_DEVICES=all
lxc.environment: NVIDIA_DRIVER_CAPABILITIES=compute,utility,video
lxc.hook.mount: /usr/share/lxc/hooks/nvidia
```

This method automatically injects the NVIDIA devices and drivers into the container at mount time using the NVIDIA Container Toolkit hook. The environment variables configure which GPUs are visible (`all` or specific GPU IDs) and what capabilities are enabled (compute, utility, video, graphics, etc.).

### Option 2: Manual Device Passthrough (no longer recommended)

Alternatively, you can manually bind mount the NVIDIA devices. First, check the device entries on your host:

```
root@pve01:/etc/pve/lxc# ls -l /dev/nvidia*
crw-rw-rw- 1 root root 195,   0 Feb 13 21:15 /dev/nvidia0
crw-rw-rw- 1 root root 195, 255 Feb 13 21:15 /dev/nvidiactl
crw-rw-rw- 1 root root 195, 254 Feb 13 21:15 /dev/nvidia-modeset
crw-rw-rw- 1 root root 511,   0 Feb 13 21:15 /dev/nvidia-uvm
crw-rw-rw- 1 root root 511,   1 Feb 13 21:15 /dev/nvidia-uvm-tools
```

Then add the following lines to the LXC guest .conf file (in `/etc/pve/lxc/<id>.conf`):

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

After adding the configuration lines, you must reboot the LXC container.

See these resources for additional information:
- https://jocke.no/2022/02/23/plex-gpu-transcoding-in-docker-on-lxc-on-proxmox/
- https://theorangeone.net/posts/lxc-nvidia-gpu-passthrough/
- https://gist.github.com/kuanghan/1cf09e25c4a67d92d5e7b1804403f052?permalink_comment_id=5212453#gistcomment-5212453

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
