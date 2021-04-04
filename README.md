# RPi_KubernetesCluster [draft - not finished repo]
This is a PoC for a fully automated setup of a Kubernetes Cluster on Raspberry Pi

first things first  
if you have a fresh ubuntu on your wsl and no ansible installed:  
https://docs.ansible.com/ansible/2.7/installation_guide/intro_installation.html#latest-releases-via-apt-ubuntu  

```shell
sudo apt-get update  
sudo apt-get install software-properties-common  
sudo apt-add-repository --yes --update ppa:ansible/ansible  
sudo apt-get install ansible  
```

Get your raspberry image  
Add a plain file on the ssd which is called ssh. This lets you enable ssh on the first boot, so you can work with your headless nodes without getting an hdmi cable, a monitor and an extra keyboard.  
  
Add your hosts  
In ansible hosts file (`/etc/ansible/hosts`)  
```ini
[masterNodes]  
masterNode1 ansible_host=192.168.1.50
[workerNodes]  
workerNode1 ansible_host=192.168.1.60
```
Add var folders in `/etc/ansible/groups_vars/[masterNodes | workerNodes | all]` and put there some ansible encrypted vaults into it (those vars will be used in the playbooks)  
groups_vars  
- masterNodes  
-- vault  
- workerNodes  
-- vault  

> you can create a new vault.yaml with `ansible-vault create vault.yaml`

the content has to be in the format:
- varName: string

in our case we need `user_password` and `root_password` since we are going to change them. To get the hashed password to pass into the vault as a value, you can use ansible itself

```shell
 ansible all -i localhost, -m debug -a "msg={{ 'mypassword' | password_hash('sha512', 'mysecretsalt') }}"
localhost | SUCCESS => {
    "msg": "$6$mysecretsalt$qJbapG68nyRab3gxvKWPUcs2g3t0oMHSHMnSKecYNpSi3CuZm.GbBqXO8BE6EI6P1JUefhA0qvD7b5LSh./PU1"
}
```
then you add this hash into your vault

```yaml
user_password: $6$mysecretsalt$qJbapG68nyRab3gxvKWPUcs2g3t0oMHSHMnSKecYNpSi3CuZm.GbBqXO8BE6EI6P1JUefhA0qvD7b5LSh./PU1
root_password: $6$mysecretsalt$qJbapG68nyRab3gxvKWPUcs2g3t0oMHSHMnSKecYNpSi3CuZm.GbBqXO8BE6EI6P1JUefhA0qvD7b5LSh./PU1
```


for the user password what we want to store in the vault, we need to crypt the value. Documented here: https://docs.ansible.com/ansible/latest/modules/user_module.html
How to create crypted passwords for user module? https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-generate-encrypted-passwords-for-the-user-module

In the ansible hosts file  
[all:vars]  
ansible_connection=ssh  
#add for the first step 01_:  
ansible_ssh_user=pi  
ansible_ssh_password=raspberry  

> you can add `ansible_python_interpreter=/usr/bin/python3` if you encounter some issues with the apt update/upgrade task in 02_

Now you run:

```shell
ansible-playbook 01_fresh_rpi.yaml --ask-vault-pass
```

> If you run into an error that there might be something with the tmp folder add to your /etc/ansible/ansible.cfg the following: remote_tmp = /tmp/.ansible-${USER}/tmp


### fresh step two:

```shell
start ssh agent
eval $(ssh-agent -s)
add your key to the agent
ssh-add ~/.ssh/id_rsa
```

after that you'll be able to run the second steps with the playbook

For that you'll need to pass the sudo password (-K)

```shell
ansible-playbook 02_fresh_rpi.yaml --ask-vault-pass -K
```



!important for ARM based cluster (RPi)
Kubernetes Dashboard (ARM) $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard-arm.yaml (https://github.com/kubernetes/dashboard/wiki/Installation)

!important for iptables version 1.8
You need to allow forward rules in iptables https://docs.oracle.com/cd/E52668_01/E88884/html/kube_admin_config_iptables.html
https://github.com/kubernetes/kubernetes/issues/71305


thoughts about ivp6:
enable ipv6 fowarding
enable ipv6 RA //when ipv6 forwarding enabled is, this option has to be set explicitly
enable ipv6proxyndp //docker container should get their own ipv6 address via RA

docker daemon.json
ipv6 true
fixed ipv6 subnet //I set the prefix which I get from my router

dns/ping doesn't work currently
result: install ndppd for forwarding the RA from eth0 to docker0, that the container also can react to RA
/etc/ndppd.conf
proxy eth0 {
  rule 2a02:8070:18a:200::a0/125 { auto }
}


kubeadm_v6.cfg
apiVersion: kubeadm.k8s.io/v1alpha3
kind: MasterConfiguration
api:
  advertiseAddress: fd00::100
networking:
  serviceSubnet: fd00:1234::/110
nodeName: kube-master
