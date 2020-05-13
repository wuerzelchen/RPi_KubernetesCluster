```bash
# add the pi user and password to ansible hosts file

# if already there but commented out
sudo sed -i 's/^#.*ansible_/ansible_/g' /etc/ansible/hosts


# this adds a ssh public key with the given username to the node

ansible-playbook 00_add_sshpubkey.yaml --ask-vault

# this will initialize the node with the given user and enables ssh on startup

ansible-playbook 01_fresh_rpi.yaml --ask-vault-pass

# now remove the pi user and default password from the ansible hosts file

# you can also comment it out
#TODO: need to fix this, it does not add # in front of the line
sudo sed '/^#*ansible_user/d' /etc/ansible/hosts
sudo sed '/^#*ansible_host/d' /etc/ansible/hosts

# execute the script to update to latest version and disable password/root login
# this takes quite some time
ansible-playbook 02_fresh_rpi.yaml --ask-vault-pass -K

# install docker
ansible-playbook 03_install_docker.yaml --ask-vault-pass -K


ansible-playbook 04_install_kubeadm.yaml --ask-vault-pass -K

```