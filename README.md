# Ansible Role proxmox-vm-setup

This role is meant so create a VM on a physical host running proxmox.
The VM is only created if it does not already exist and is started afterwards.

The role is meant to create a VM using a pre existing template VM which contains only a fresh Debian Cloud Image like image.
This template is then cloned and configured using `cloud-init`.

There are two variables for storagepools:

* `proxmox_storagepool`: The storagepool in which the additional disks are created
* `proxmox_system_storagepool`: The storagepool in which the system disks and the cloud-init disks are created (defaults to `proxmox_storagepool`)

Currently supported storage types are: `btrfs` and `LVM`

## Example usage:

```
# vm-playbook.yml
vars:
  physical_host: kvm-server-99
  vm_template: "debian11-cloudinit-template" # (default)
roles:
  - proxmox-vm-setup
```

```bash
ansible-playbook vm-playbook.yml
```

```
# host_vars/vm.yml
vm_cores: 4
vm_memory: 2048
vm_swap: 2G
vm_disksize: 80G
vm_additional_disks:
  - id: 1
    disksize: 120G
    filesystem: btrfs
    mountpoint: /data
```

will create a VM on `kvm-server-99` from the template `debian11-cloudinit-template`.
The MAC address is set based on the `inventory_hostname` and it is idempotent, which means that two VM with the same name will have the same MAC.

## Dependencies

By default the role calls the [geerlingguy/ansible-role-swap](https://gitlabintern.emlix.com/emlix/infrastructure/ansible-mirror/ansible-role-swap) role to configure swap inside the VM.
This can be disables by setting `swap_file_state: absent`

## Troubleshooting

###  The task includes an option with an undefined variable. The error was: 'dict object' has no attribute 'vmid'.

This usually happend when two VMs with the same name exist in the cluster (which is not supported by this role yet)
or sometimes if a VM was created and is not yet known by its name in a slow cluster.
For more information run Ansible with `-vvvv` and look at the output of _"Get VM current state after creating"_
