# vagrant-ha-openshift
Openshift Vagrant playground to test High Availability features with GlusterFS  

# Overview

````


````

# full (re)deploy cycle
```sh
vagrant destroy -f
vagrant landrush rm --all
time vagrant up
vagrant landrush ls |grep 192.168.100 |sed 's/\,//' |cut -d" " -f1 |while read I; do  ssh-copy-id vagrant@$I; done
ansible nodes -i inventory-ha.yml  -m shell -a "uptime" -b
time ansible-playbook -i inventory-ha.yml openshift-ansible/playbooks/byo/config.yml
time ansible-playbook -i inventory-ha-glusterfs.yml openshift-ansible/playbooks/byo/openshift-glusterfs/config.yml
```
