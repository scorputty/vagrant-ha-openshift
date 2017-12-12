# vagrant-ha-openshift
Openshift Vagrant playground to test High Availability features with GlusterFS  

# Warning Proxy!!!
If you need to use a proxy in a corporate network and want to test how this effects an OpenShift setup and install you can do that with this test setup.

*** If not I highly recommend to comment-out the proxy section in the two following files.

### Vagrantfile proxy section:
```yaml
# Comment out if no Proxy is needed
if Vagrant.has_plugin?("vagrant-proxyconf")
  config.proxy.http     = "http://10.0.2.2:3128"
  config.proxy.https    = "http://10.0.2.2:3128"
  config.proxy.no_proxy = "localhost,127.0.0.1,.vagrant.test,kube-service-catalog.svc,192.168.42.0/24,172.30.0.0/16,10.128.0.0/14"
end
```

### OpenShift inventory proxy section (hosts.yml)
```yaml
# Comment out if no Proxy is needed
openshift_http_proxy=http://10.0.2.2:3128
openshift_https_proxy=http://10.0.2.2:3128
openshift_no_proxy="localhost,127.0.0.1,.vagrant.test,kube-service-catalog.svc,192.168.42.22,192.168.42.32,192.168.42.33,192.168.42.52,192.168.42.53,172.30.0.0/16,10.128.0.0/14"

# Overview
The separate inventory-* directories are to test different types of environments. Currently only inventory-simple contains files.

# testing with Ansible Vault to encrypt secrets in the hosts.yml (inventory)
To check or edit the encrypted vaul-vars.yml file: (The password is "test1234")
```sh
ansible-vault edit inventory-simple/vault-vars.yml
```
This will open your default shell editor (export EDITOR=vim)
```yaml
# These are the vault encrypted variables
# Set variables common for all OSEv3 hosts
[OSEv3:vars]

openshift_master_htpasswd_users={'admin': '$apr1$6CZ4noKr$IksMFMgsW5e5FL0ioBhkk/', 'developer': '$apr1$AvisAPTG$xrVnJ/J0a83hAYlZcxHVf1'}
```
You can add or remove variables to this file and be secure! Just remember to remove them from the plain-text inventory first ;-)

# full (re)deploy cycle
change IP and inventory when needed...
```sh
vagrant destroy -f
vagrant landrush rm --all
time vagrant up
vagrant landrush ls |grep 192.168.42 |sed 's/\,//' |cut -d" " -f1 |while read I; do  ssh-copy-id vagrant@$I; done
ansible nodes -i inventory-simple/ --ask-vault-pass -m shell -a "sed -i '/127\.0\.0\.1.*vagrant*/d' /etc/hosts"
ansible nodes -i inventory-simple/ --ask-vault-pass -m shell -a "cat /etc/hosts"
git clone https://github.com/openshift/openshift-ansible
cd openshift-ansible
git checkout release-3.7
git fetch
cd ../
time ansible-playbook -i inventory-simple/ openshift-ansible/playbooks/byo/config.yml --ask-vault-pass
```
