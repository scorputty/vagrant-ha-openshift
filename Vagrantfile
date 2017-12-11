# -*- mode: ruby -*-
# # vi: set ft=ruby :

# This Vagrantfile installs a x node setup to test Openshift High Availabillity concepts
#
# Warning, this is only tested on Mac OSX, it should work on any Linux with minor adaptations...
#
# Download the following plugins:
#
# hostmanager (1.2.2)
# landrush (1.2.0) (also brew install dnsmasq)
# vagrant-hostmanager (1.8.7)
# vagrant-proxyconf (1.5.2)
# vagrant-share (1.1.9, system)
# vagrant-vbguest (0.14.2)

# Notes
# stage 1 installs the "infra" hosts
# Tip, ping your hosts to make sure Landrush DNS works, if not do this "sudo killall -HUP mDNSResponder"
# or try to restart dnsmasq:
# sudo launchctl stop homebrew.mxcl.dnsmasq
# sudo launchctl start homebrew.mxcl.dnsmasq
# stage 2 setup openshift via ansible-playbook
# git clone https://github.com/openshift/openshift-ansible
# cd openshift-ansible/
# git checkout release-3.6 # or any branch you want to test...
# git pull
# cd ../
# test ansible:
# ansible all -m ping -i inventory-ha.yml --ask-pass
# run Openshift install from cli like this:
# ansible-playbook --ask-pass ./openshift-ansible/playbooks/byo/config.yml -i inventory-simple.yml
# (pass = vagrant)
# or setup ssh keys like so:
# vagrant landrush ls |grep 192.168.42 |sed 's/\,//' |cut -d" " -f1 |while read I; do  ssh-copy-id vagrant@$I; done
# if you also want gluster it might be good to do it seperate:
# time ansible-playbook -i inventory-ha.yml openshift-ansible/playbooks/byo/openshift-glusterfs/config.yml
# if you are behind a corperate proxy install this:
# vagrant plugin install vagrant-proxyconf
# change the proxy settings or install cntlm to use <GUESTIP>:3128
# in Vagrant GUESTIP is useally "10.0.2.2"
# If you need authentication it's better to use your environment:
# export http_proxy="http://user:password@host:port"
# vagrant plugin install vagrant-proxyconf
# export VAGRANT_HTTP_PROXY="http://user:password@host:port"
# export VAGRANT_NO_PROXY="127.0.0.1"
# vagrant up

# HA proxy
# http://gateway1.vagrant.test:9000/stats

require 'fileutils'
require 'open-uri'
require 'tempfile'
require 'yaml'

Vagrant.require_version ">= 2.0.0"

# todo, variables in config.rb file
CONFIG = File.expand_path("config.rb")
if File.exist?(CONFIG)
  require CONFIG
end

# directory of ansible-playbooks
PROVTYPE = "provision"
# gateway (loadbalancer) server
GATEWAY = 0
# gluster servers with number of disks
GLUSTER = 3
DISKS = 2
DSIZE = 250
# openshift-cluster
OSMASTER = 1
OSINFRA = 0
OSNODE = 2

Vagrant.configure("2") do |config|
  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http     = "http://10.0.2.2:3128"
    config.proxy.https    = "http://10.0.2.2:3128"
    config.proxy.no_proxy = "localhost,127.0.0.1,.vagrant.test,192.168.42.0/24,172.30.0.0/16,10.128.0.0/14"
  end
  if Vagrant.has_plugin?("landrush")
    config.landrush.enabled = true
    config.landrush.host 'console-openshift-cluster.vagrant.test', 'osmaster1.vagrant.test'
    config.landrush.host 'apps.openshift-cluster.vagrant.test', 'osmaster1.vagrant.test'
  else
    abort("Please install the landrush plugin, vagrant plugin install landrush")
  end
  if Vagrant.has_plugin?("vagrant-hostmanager")
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = false
    config.hostmanager.manage_guest = true
  else
    abort("Please install the vagrant-hostmanager plugin, vagrant plugin install vagrant-hostmanager")
  end
  config.ssh.insert_key = false
  config.vm.synced_folder ".", "/home/vagrant/sync", disabled: true
  config.vm.provider "virtualbox" do |v|
    v.linked_clone = true
  end


  (1..GATEWAY).each do |i|
    config.vm.define "gateway#{i}" do |node|
      node.vm.box = "scorputty/centos"
      node.vm.hostname = "gateway#{i}.vagrant.test"
      node.vm.network :private_network, ip: "192.168.42.%d" % (1 + i )

      # Enable ssh forward agent
      config.ssh.forward_agent = false
      node.vm.provider :virtualbox do |vb|
        vb.memory = "512"
      end

      if i == GATEWAY
        node.vm.provision :ansible do |ansible|
          ansible.playbook = "#{PROVTYPE}/site.yml"
          ansible.become = true
          ansible.verbose = "v"
          ansible.host_key_checking = false
        end
      end
    end
  end


  (1..GLUSTER).each do |i|
    config.vm.define "gluster#{i}" do |node|
      node.vm.box = "scorputty/centos"
      node.vm.hostname = "gluster#{i}.vagrant.test"
      node.vm.network :private_network, ip: "192.168.42.%d" % (11 + i )
      # Enable ssh forward agent
      config.ssh.forward_agent = false
      node.vm.provider :virtualbox do |vb|
        vb.memory = "512"
        unless File.exist?("gluster#{i}-disk0.vdi")
          vb.customize ["storagectl", :id,"--name", "VboxSata", "--add", "sata"]
        end
      end
      (0..DISKS-1).each do |d|
        node.vm.provider :virtualbox do |vb|
          unless File.exist?("gluster#{i}-disk#{d}.vdi")
            vb.customize [ "createmedium", "--filename", "gluster#{i}-disk#{d}.vdi", "--size", 1024*DSIZE ]
          end
          vb.customize [ "storageattach", :id, "--storagectl", "VboxSata", "--port", 3+d, "--device", 0, "--type", "hdd", "--medium", "gluster#{i}-disk#{d}.vdi" ]
        end
      end

      # if i == GLUSTER
      #   node.vm.provision :ansible do |ansible|
      #     ansible.playbook = "#{PROVTYPE}/site.yml"
      #     ansible.become = true
      #     ansible.verbose = "v"
      #     ansible.host_key_checking = false
      #   end
      # end
    end
  end


  (1..OSMASTER).each do |i|
    config.vm.define "osmaster#{i}" do |node|
      node.vm.box = "scorputty/centos"
      # node.vm.box = "centos/atomic-host"
      node.vm.hostname = "osmaster#{i}.vagrant.test"
      node.vm.network :private_network, ip: "192.168.42.%d" % (21 + i )
      # Enable ssh forward agent
      config.ssh.forward_agent = false
      node.vm.provider :virtualbox do |vb|
        vb.memory = "4096"
      end

      if i == OSMASTER
        node.vm.provision :ansible do |ansible|
          ansible.playbook = "#{PROVTYPE}/site.yml"
          ansible.become = true
          ansible.verbose = "v"
          ansible.host_key_checking = false
        end
      end
    end
  end


  (1..OSINFRA).each do |i|
    config.vm.define "osinfra#{i}" do |node|
      node.vm.box = "scorputty/centos"
      # node.vm.box = "centos/atomic-host"
      node.vm.hostname = "osinfra#{i}.vagrant.test"
      node.vm.network :private_network, ip: "192.168.42.%d" % (31 + i )
      # Enable ssh forward agent
      config.ssh.forward_agent = false
      node.vm.provider :virtualbox do |vb|
        vb.memory = "1024"
      end

      if i == OSINFRA
        node.vm.provision :ansible do |ansible|
          ansible.playbook = "#{PROVTYPE}/site.yml"
          ansible.become = true
          ansible.verbose = "v"
          ansible.host_key_checking = false
        end
      end
    end
  end


  (1..OSNODE).each do |i|
    config.vm.define "osnode#{i}" do |node|
      node.vm.box = "scorputty/centos"
      # node.vm.box = "centos/atomic-host"
      node.vm.hostname = "osnode#{i}.vagrant.test"
      node.vm.network :private_network, ip: "192.168.42.%d" % (51+ i )
      # Enable ssh forward agent
      config.ssh.forward_agent = false
      node.vm.provider :virtualbox do |vb|
        vb.memory = "1024"
      end

      if i == OSNODE
        node.vm.provision :ansible do |ansible|
          ansible.playbook = "#{PROVTYPE}/site.yml"
          ansible.become = true
          ansible.verbose = "v"
          ansible.host_key_checking = false
        end
      end
    end
  end


end
