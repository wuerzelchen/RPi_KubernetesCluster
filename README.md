# RPi_KubernetesCluster [draft - not finished repo]
This is a PoC for a fully automated setup of a Kubernetes Cluster on Raspberry Pi

first things first  
if you have a fresh ubuntu on your wsl and no ansible installed:  
https://docs.ansible.com/ansible/2.7/installation_guide/intro_installation.html#latest-releases-via-apt-ubuntu  
$ sudo apt-get update  
$ sudo apt-get install software-properties-common  
$ sudo apt-add-repository --yes --update ppa:ansible/ansible  
$ sudo apt-get install ansible  

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

the content has to be in the format:
varName: string

for the user password what we want to store in the vault, we need to crypt the value. Documented here: https://docs.ansible.com/ansible/latest/modules/user_module.html
How to create crypted passwords for user module? https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-generate-crypted-passwords-for-the-user-module

In the ansible hosts file  
[all:vars]  
ansible_connection=ssh  
#add for the first step:  
ansible_ssh_user=pi  
ansible_ssh_password=raspberry  




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
