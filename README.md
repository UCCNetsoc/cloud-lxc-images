# Netsoc Cloud LXC Images
This repo contains an ansible role for building LXC images for [Netsoc Cloud](https://github.com/UCCNetsoc/cloud)

Ansible executes [Packer](https://www.packer.io) for building the images. Packer uses the Ansible provisioner to provision the images.

Images are defined in [images.yml](./images.yml). Each image requires the keys `name`, `distro` and `rel` to be specified. \
`name` specifies the name of the image \
`distro` and `rel` correspond to base [LXC images](https://uk.images.linuxcontainers.org)

Provisioning playbooks are stored in [playbooks](./playbooks). The playbook to use is read from the path `playbooks/<dist>/<name>.yml`.

Container networking is handled one of two ways:
1. If the `dev_env` variable is set to true then `lxc-net` will be used which handles creating a bridge interface and uses dnsmasq to assign IP addresses to containers.
2. If the `dev_env` variable isn't defined or is set to false then a macvlan will be used linked to vmbr0.255. The vmbr0.255 interface must exist and DHCP must be available to assign IP addresses to containers.

### How To Use
In [NaC](https://github.com/UCCNetsoc/NaC) submodule this role. \
The build host should be a Proxmox node (running Python 3) as it is already set up to run LXC containers.

Packer must be installed. This role **does not** install Packer

Set the `dev_env` variable if running in the dev-env

Example play:
```yml
- name: Images
  hosts: build_host
  become: yes
  tasks:
    - block:
      - set_fact:
          image_output: "/tmp/images/"
      - include_role:
          name: <cloud-lxc-images-submodule>/
  ignore_errors: yes
  tags:
    - always
```
*note:* `image_output` must be specified as it is the directory the images will be output to.

Only images specified using ansible tags will be provisioned. Tags should be of the format `<name>-<dist>` e.g.
`--tags "test-alpine"`