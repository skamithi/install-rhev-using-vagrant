# -*- mode: ruby -*-
# vi: set ft=ruby :

## When running `vagrant up` run it with the `--no-parallel` option.
## This ensures that the fuel_master comes up first

rhevm_ip = '10.20.0.2'
rhevh1_ip = '10.20.0.3'
rhev_box = 'rhel74'
ansible_debug = ''
yum_update = true


rhev_core_subscription_repos = %w{
 	rhel-7-server-rpms
 	rhel-7-server-supplementary-rpms
 	rhel-7-server-rhv-4.1-rpms
 	rhel-7-server-rhv-4-tools-rpms
 	jb-eap-7-for-rhel-7-server-rpms
}

rhevh_subscription_repos = rhev_core_subscription_repos
rhevh_subscription_repos.push('rhel-7-server-rhv-4-mgmt-agent-rpms')

# Add DEBUG_INSTALL=yes before calling vagrant command to enable ansible debugs
if ENV['DEBUG_INSTALL']
  ansible_debug = 'vvvv'
end

# Add NO_YUM_UPDATE before calling the vagrant command to disable yum updates.
# useful for post install ansible troubleshooting. yum updates may take a while to
# run if it never completed
if ENV['NO_YUM_UPDATE']
  yum_update = false
end

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

    node.vm.box = rhev_box
    node.vm.hostname = "rhevm"
    node.vm.network :private_network,
      :ip => rhevm_ip,
      :prefix => '24',
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__network_name => 'rhevm_net',
      :libvirt__dhcp_enabled => false

    node.vm.provision :ansible do  |ansible|
      ansible.vault_password_file = "v_pass"
      ansible.playbook = "rhevm.yml"
      ansible.verbose = ansible_debug
      ansible.extra_vars = {
        "rhev_server_type": "rhevm",
        "update_yum": yum_update,
        "subscription_repos": rhev_core_subscription_repos,
        "ovirt_engine_host": "rhevm",
   			"hosts_additional_hosts": [{
         	"address": rhevh1_ip,
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

    node.vm.box = rhev_box
    node.vm.hostname = "rhevh1"
    node.vm.network :private_network,
      :ip => rhevh1_ip,
      :prefix => '24',
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__network_name => 'rhevm_net',
      :libvirt__dhcp_enabled => false

    node.vm.provision :ansible do  |ansible|
      ansible.vault_password_file = "v_pass"
      ansible.playbook = "rhevh1.yml"
      ansible.verbose = ansible_debug

      ansible.extra_vars = {
        "update_yum": yum_update,
        "rhev_server_type": "rhevh",
        "subscription_repos": rhevh_core_subscription_repos,
        "ovirt_engine_host": "rhevm",
        "rhel_hypervisor_ip": rhevh1_ip,
        "rhel_hypervisor_name": "rhevh1",
   			"hosts_additional_hosts": [{
         	"address": rhevm_ip,
          "hostnames": [
            	"rhevm"
         	]}
				]
      }
    end
  end

end
