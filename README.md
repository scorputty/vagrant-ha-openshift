# vagrant-ha-openshift
Openshift Vagrant playground to test High Availability features with GlusterFS  

# Overview

````


````

# full (re)deploy cycle
change IP and inventory when needed...
```sh
vagrant destroy -f
vagrant landrush rm --all
time vagrant up
vagrant landrush ls |grep 192.168.100 |sed 's/\,//' |cut -d" " -f1 |while read I; do  ssh-copy-id vagrant@$I; done
ansible nodes -i inventory-simple  -m shell -a "uptime" -b --ask-vault-pass
git clone https://github.com/openshift/openshift-ansible
cd openshift-ansible
git checkout release-3.6
git fetch
cd ../
time ansible-playbook -i inventory-simple openshift-ansible/playbooks/byo/config.yml --ask-vault-pass
```
