# RPi_KubernetesCluster [draft - not finished repo]
This is a PoC for a fully automated setup of a Kubernetes Cluster on Raspberry Pi

first things first
Get your raspberry image
Add a plain file on the ssd which is called ssh. This lets you enable ssh on the first boot, so you can work with your headless nodes without getting an hdmi cable, a monitor and an extra keyboard.

Add your hosts
In ansible hosts file (/etc/ansible/hosts)
[hostGroupMaster]
[hostGroupNodes]

Add var folders in groups_vars and put there some ansible encrypted vaults into it (those vars will be used in the playbooks)
groups_vars
- hostGroupMaster
-- vault
- hostGroupNodes
-- vault

In the ansible hosts file
[all:vars]
ansible_connection=ssh
#add for the first step:
ansible_ssh_user=pi
ansible_ssh_password=raspberry
