# -*- mode: ruby -*-
# vi: set ft=ruby :

## When running `vagrant up` run it with the `--no-parallel` option.
## This ensures that the fuel_master comes up first

rhevm_ip = '10.2.0.2'
rhevh1_ip = '10.2.0.3'

Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Disable the synced_folder feature
  config.vm.synced_folder '.', '/vagrant', :disabled => true

  config.vm.define :rhevm do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 4096
      domain.cpus = 2
    end

    node.vm.box = "rhel73"
    node.vm.hostname = "rhevm"
    node.vm.network :private_network,
      :ip => '10.20.0.2',
      :prefix => '24',
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__network_name => 'rhevm_net',
      :libvirt__dhcp_enabled => false

    node.vm.provision :ansible do  |ansible|
      ansible.vault_password_file = "v_pass"
      ansible.playbook = "rhevm.yml"
#      ansible.tags = "rhev"
      ansible.extra_vars = {
   			"host_additional_hosts": [{
         	"address": rhevh1_ip
      		},
      		{
         		"hostnames": [
            	"rhevh1"
         	]
      		}
				]
			}
    end
  end

  config.vm.define :rhevh1 do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 4096
      domain.cpus = 2
    end

    node.vm.box = "rhel73"
    node.vm.hostname = "rhevh1"
    node.vm.network :private_network,
      :ip => '10.20.0.3',
      :prefix => '24',
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__network_name => 'rhevm_net',
      :libvirt__dhcp_enabled => false

    node.vm.provision :ansible do  |ansible|
      ansible.vault_password_file = "v_pass"
      ansible.playbook = "rhevh1.yml"
      ansible.extra_vars = {
   			"host_additional_hosts": [{
         	"address": rhevm_ip
      		},
      		{
         		"hostnames": [
            	"rhevm"
         	]
      		}
				]

      }
    end
  end

end
