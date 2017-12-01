# Basic RHEV Install Demo using Vagrant-Libvirt

This  install includes 2 RHEL 7.4 VMs, the first is the RHEV Manager and the second
one is the RHEV hypervisor. The vagrant install does the following:

* Install RHEL7.4 from a Vagrant box
* Add the necessary RHEL subscriptions to install RHEV
* Install RHEVM on one VM and using Ovirt Ansible modules create a RHEV Hypervisor on the 2nd VM.
* Download the [Openstack Cirros](https://docs.openstack.org/image-guide/obtain-images.html) VM and install it as a RHEV template
* Deploy 1 VM based on the cirros vm template.

## Requirements

* Fast internet connection. RHEV wants to download alot of packages from the internet.
* PC with at least 8GB of RAM, 50G of disk.
* RHEL 7.4 iso
* Red Hat Subscriptions  that can install RHEV Manager and RHEV Hypervisor code.
* Vagrant Hypervisor that has [KVM Nested Virtualization](https://fedoraproject.org/wiki/How_to_enable_nested_virtualization_in_KVM enabled. The RHEV Hypervisor will fails to be installed if it cannot detect KVM virtualization. 


## Build a RHEL Vagrant Box
Build a RHEL vagrant box.

* Install [Vagrant-libvirt](https://linuxsimba.com/vagrant-libvirt-install)
* Download [Packer](https://www.packer.io/downloads.html)
* Download [RHEL 7.4 Binary DVD](https://access.redhat.com/downloads), around 3-4GB.
* Download the [Chef Bento](https://github.com/chef/bento) Git project.

Create a _packer_ var file pointing to the location where the RHEL iso exists
on your server/laptop.

## Topology

```
+---------------+             +----------------+
|               |             |                |
|    RHEV       |             |  RHEV          |
|    Manager    |eth1       eth1 Hypervisor    |
|               +-----------+ |                |
|               |             |                |
|               |             |                |
+-----eth0------+             +------eth0------+
        |                              |
+-----------+------------------------------+------------+
|                                                       |
|               Vagrant hyperVisor                      |
+-------------------------------------------------------+

```


## Create a RHEL vagrant box

```
# cd git/bento/rhel
# cat rhel_local.json

{
  "mirror": "file://",
  "mirror_directory": "/home/linuxsimba/isos"
}
# packer build -only qemu -var-file rhel_local.json rhel-7.4-x86_64.json
qemu output will be in this color.

==> qemu: Downloading or copying ISO
    qemu: Downloading or copying: file:///home/linuxsimba/isos/rhel-server-7.4-x86_64-dvd.iso
==> qemu: Creating hard drive...
...
.....
[ after 10-15 minutes]
....
....
==> Builds finished. The artifacts of successful builds are:
--> qemu: 'libvirt' provider box: ../builds/rhel-7.4.libvirt.box

```

* Upload the new RHEL box into your default vagrant box store

```
 vagrant box add ../builds/rhel-7.4.libvirt.box --name rhel74
```


## Download RHEV Vagrant libvirt Git Repo

```
git clone https://github.com/linuxsimba/install-rhev-using-vagrant
```

## Install Ansible

Use your system package manager or PIP  to install Ansible 2.4 or higher.

## Create Secret Info file

Create the file ``group_vars/all/vault.yml`` in the rhev vagrant directory
Pool IDs are Red Hat subscription pools that contain the Red Hat Virtualization repos.
Review the [RHEV Install Docs](https://access.redhat.com/documentation/en-us/red_hat_virtualization/4.1/html/installation_guide/chap-installing_red_hat_enterprise_virtualization#Subscribing_to_the_Red_Hat_Enterprise_Virtualization_Manager_Channels_using_Subscription_Manager) for more details on how to get find the right Pool IDs
```
# cd $HOME/git/install-rhev-using-vagrant
# echo "

vault_subscription_pass: [access_redhat_password]
vault_subscription_user: [access_redhat_username]
vault_pool_ids:
  - 1234523423434
  - 23423234234
vault_ovirt_password: [rhevm admin password]

" > ./group_vars/all/vault.yml

# ansible-vault encrypt ./group_vars/all/vault.yml
Password: [create_a_vault_password]
Confirm Password: [confirm_vault_password]

# echo "[password_used_to_encrypt_vault_file]" > v_pass

```

## Run Vagrant Up

run vagrant in non parallel mode because the RHEVM must come up first before the single hypervisor

```
vagrant up rhevm rhevh1 --no-parallel
````

After several minutes a RHEV test environment is up and running. Ready for you to
some basic testing. I needed to do this because I wanted to kick the tires
on the ovirt Ansible modules

## Accessing the UI from the Vagrant hypervisor
From your laptop or server, you neeed to modify the ``/etc/hosts`` file with the IP of the rhevm
management port (eth0). So something like this

```
...
192.168.100.1 rhevm.test.local rhevm
....

```
The reason for this is that RHEVM does not allow you to connect by IP. Must be by name.
Weird, but that's how it works.

After that from your browser type [https://rhevm.test.local/ovirt-engine]([https://rhevm.test.local/ovirt-engine). Click on the _Administration Portal_.
The username is ``admin`` and the password is whatever you set in ``vault_ovirt_password`` in your vault file.

## VM Console

The default Native Client console does not work due to the networking in the VM. Instead of doing vagrant port masquerading, go to the Console Options for the VM (right click the VM to see this) and change the Console invocation to SPICE HTML5.

You will see a message saying the following:

```
Can't connect to websocket proxy server wss://rhevm.test.local:6100. Please check that:
websocket proxy service is running,
firewalls are properly set,
websocket proxy certificate is trusted by your browser. Default CA certificate.
```

Click on the [Default CA Certificate](https://rhevm.test.local/ovirt-engine/services/pki-resource?resource=ca-certificate&format=X509-PEM-CA) and install the CA Certificate into your browser.
Close the console browser window and bring up the console again in a new window. The console should now be visible on your browser.

## Troubleshooting

RHEV seems to have issues rendering the login page and adminstrative portal pages. Recommend using Incognito/Private Window modes in either Chrome or Firefox. Seems to resolve all those page loading issues.

Each time you rebuild this environment the CA certificate will change so you will need to update your browser CA certs as well.

This setup requires a reliable internet connection. To save on bandwidth you could consider building a RHEL _golden image_ vagrant box with the appropriate subscriptions and Yum update completed.

