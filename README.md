# Ansible Role: virt_install_vm

An Ansible role to install a libvirt virtual machine with ```virt-install```
and ```cloud-init```. It is "designed" to be flexible.

An example template is provided to set up a Debian system.

## Requirements

The role is a wrapper around the following roles:

  * **stafwag.libvirt**:
    [https://github.com/stafwag/ansible-role-libvirt](https://github.com/stafwag/ansible-role-libvirt)
  * **stafwag.qemu_img**:
    [https://github.com/stafwag/ansible-role-qemu_img](https://github.com/stafwag/ansible-role-qemu_img)
  * **stafwag.cloud_localds**:
    [https://github.com/stafwag/ansible-role-cloud_localds](https://github.com/stafwag/ansible-role-cloud_localds)
  * **stafwag.virt_install_import**:
    [https://github.com/stafwag/ansible-role-virt_install_import](https://github.com/stafwag/ansible-role-virt_install_import)

Install the required roles with

```bash
$ ansible-galaxy install -r requirements.yml
```

this will install the latest default branch releases.

Or follow the installation instruction for each role on Ansible Galaxy.

[https://galaxy.ansible.com/stafwag](https://galaxy.ansible.com/stafwag)

# Installation

## Ansible galaxy

The role is available on [Ansible Galaxy](https://galaxy.ansible.com/stafwag/virt_install_import).

To install the role from Ansible Galaxy execute the command below. 
This will install the role with the dependencies.

```bash
$ ansible-galaxy install stafwag.virt_install_import
```

## Source Code

If you want to use the source code directly.

Clone the role source code.

```bash
$ git clone https://github.com/stafwag/ansible-role-virt_install_vm
```

and put into the [role search path](https://docs.ansible.com/ansible/2.4/playbooks_reuse_roles.html#role-search-path)

### Supported GNU/Linux Distributions

It should work on most GNU/Linux distributions.
```cloud-localds``` is required. ```cloud-localds``` was available on
Centos/RedHat 7 but not on Redhat 8. You'll need to install it manually
to use the role on Centos/RedHat 8.

* Archlinux
* Debian
* Centos 7
* RedHat 7
* Ubuntu

## Role Variables and templates

### Role Variables

See the documentation of the roles in the **Requirements** section.

* **virt_install_vm**: "namespace"

  * **skip_if_deployed**: boolean default: false. \

    * **When true**: \
      Skip role if the VM is already deployed. The role will exit successfully. \
    * **When false**: \
      The role will exit with an error if the VM is already deployed.

Return values:

  * **virt_install_vm_installed**:
    Default: ```false```.
    Set to ```true``` when the vm is installed by the role.

### Templates

#### Templates files

* ```templates/simple_debian```: Example template to create a Debian virtual machine. \
  This template use ```cloud_localds.cloudinfo``` to configure the cloud-init ```user-data```. \
  See the **Usage** section for an example.

#### Template variables

The template uses the following variables.

* **cloud_localds.cloud_config**: 
  * **system_info**:
    * **default_user**: The default user
      * **name**: Optional. Default: ```debian```
      * **passwd**: Optional. Default: ```false```. When set to ```false``` the user will be locked.
      * **ssh_authorized_keys**: An array with the authorized_key array.
  * **disable_cloud_init**: Optional. Default ```undefined```. When set to true. cloud-init will be disabled on the VM.
  * **network**:
      * **dns_nameservers**: List with the nameservers.
      * **dns_search**: The dns search string.
      * **interface**: The default network interface to configure.
          * **name**: The interface name.
          * **address**: The ip address.
          * **gateway**: The gateway.
  * **commands**: Array with commands to execute on the VM.
  * **poweroff**: Poeroff the VM after the VM is installed.
  * **reboot**: Reboot the VM after the VM is installed.

# Usage

## Create a virtual machine template

This is a file with the role variables to set set up a virtual machine with all the common settings for the virtual machines.
In this example ```vm.hostname``` and ```vm.ip_address``` can be configured
for each virtual machine.

* **debian_vm_template.yml:**

```yaml
qemu_img:
  dest: "/var/lib/libvirt/images/{{ vm.hostname }}.qcow2"
  format: qcow2
  src: /Downloads/isos/debian/cloud/debian-10-generic-amd64.qcow2
  size: "50G"
  owner: root
  group: kvm
  mode: 660
cloud_localds:
  dest: "/var/lib/libvirt/images/{{ vm.hostname }}_cloudinit.iso"
  config_template: "templates/simple_debian/debian.j2"
  network_config_template: "templates/simple_debian/debian_netconfig.j2"
  cloud_config:
    system_info:
      default_user:
        name: ansible
        passwd: "{{ ansible_become_hash }}"
        ssh_authorized_keys:
          - "{{lookup('file', '~/.ssh/ansible_ssh_key.pub') }}"
    network:
      dns_nameservers:
        9.9.9.9
      dns_search:
        intern.local
      interface:
        name:
          enp1s0
        address:
          "{{ vm.ip_address }}"
        gateway:
          192.168.123.1
    disable_cloud_init: true
    reboot:
      true
virt_install_import:
  wait: 0
  name: "{{ vm.hostname }}"
  os_type: Linux
  os_variant: debian10
  network: network:default
  graphics: spice
  disks:
    - "/var/lib/libvirt/images/{{ vm.hostname }}.qcow2,device=disk"
    - "/var/lib/libvirt/images/{{ vm.hostname }}_cloudinit.iso,device=cdrom"
```

## Playbook

Playbook to setup a virtual machine: 

```yaml
- name: Install tstdebian2
  hosts: kvmhost
  become: true
  vars:
    vm:
      hostname:
        tstdebian2
      ip_address:
        192.168.123.2/24
  pre_tasks:
    - name: Load the vm template
      include_vars: debian_vm_template.yml
    - name: display qemu_img
      debug:
        msg: 
          - "qemu_img: {{ qemu_img }}"
  roles:
    - stafwag.virt_install_vm
```
