# -*- mode: ruby -*-
# vi: set ft=ruby :

LB_NODE_IPS = [
  "192.168.56.11"
]

DB_NODE_IPS = [
  "192.168.56.21",
  "192.168.56.22",
  "192.168.56.23"
]

Vagrant.require_version ">= 1.8.0"

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  DB_NODE_IPS.each_with_index do |ip, index|
    vm_hostname = "db-#{index}"

    config.vm.define vm_hostname do |node|
      node.vm.hostname = vm_hostname
      node.vm.network :private_network, ip: ip

      node.vm.provider "virtualbox" do |vb|
        vb.name = vm_hostname
      end

      extra_vars = {
        node_index: index,
        ip: ip,
        vm_hostname: vm_hostname,
      }

      if Range.new(0,2).include? index
        if index == 0
          extra_vars['gcomm_addresses'] = DB_NODE_IPS[..2].join(',')
          extra_vars['mariadb_command'] = 'galera_new_cluster'
        else
          extra_vars['gcomm_addresses'] = DB_NODE_IPS.join(',')
          extra_vars['mariadb_command'] = 'service mariadb start'
        end
      end

      node.vm.provision "ansible" do |ansible|
        ansible.verbose = 'vvv'
        ansible.playbook = 'mariadb-new-node.yaml'
        ansible.extra_vars = extra_vars
      end
    end    
  end

  LB_NODE_IPS.each_with_index do |ip, index|
    vm_hostname = "lb-#{index}"

    config.vm.define vm_hostname do |node|
      node.vm.hostname = vm_hostname
      node.vm.network :private_network, ip: ip
      node.vm.network "forwarded_port", guest: 80, host: 3000
      node.vm.network "forwarded_port", guest: 8404, host: 8404

      node.vm.provider :virtualbox do |vb|
        vb.name = vm_hostname
      end

      node.vm.provision :ansible do |ansible|
        ansible.verbose = 'vvv'
        ansible.playbook = 'haproxy-new-node.yaml'
      end
    end
  end
end