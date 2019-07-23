# vagrant-libvirt-maas-kvm

Setup virtual [MAAS](https://maas.io/) + [KVM](https://www.linux-kvm.org/page/Main_Page) pod in [Vagrant](https://www.vagrantup.com/) with [vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt) plugin.

## Get Started

Install packages:

    sudo apt install -y vagrant qemu-kvm libvirt-clients virt-manager

Install Vagrant plugins:

    vagrant plugin install vagrant-libvirt
    vagrant plugin install vagrant-mutate

Start Vagrant:

    vagrant up

In the meanwhile, you can start `virt-manager` to watch VM status in GUI.

Once finshed, you can access MAAS at:

    http://192.168.100.100:5240/MAAS

Enable MAAS DHCP:

    Subnets -> click VLAN -> Take action -> Provide DHCP


## Add localhost as KVM pod

To add localhost as KVM pod, user `maas` on maas controller must be able to ssh to localhost.
To do that, you can either input password, or setup ssh key.

### setup ssh key(optional)

This allows user `maas` on maas controller ssh to localhost without password.

First, ssh to maas controller:

    vagrant ssh

Second, setup key for user `maas` and check:

    sudo chsh -s /bin/bash maas  # default is /usr/sbin/nologin
    sudo su - maas
    ssh-keygen -t rsa -N ''
    ssh-copy-id $KVM_USER@KVM_HOST
    virsh -c qemu+ssh://$KVM_USER@$KVM_HOST/system list --all

`$KVM_HOST` should be gateway IP, `192.168.100.1` in this case.
`$KVM_USER` is your localhost user.

## Add pod

Go to Pods -> Add pod:

    Pod type: Virsh (virsh systems)
    Virsh address: qemu+ssh://$KVM_USER@$KVM_HOST/system

replace `$KVM_USER` and `$KVM_HOST` to real value.
`Virsh password` is required if you didn't setup ssh as above,
which is the password for your localhost user.

## Management Network

When the VM is up, you may notice it has 2 NICs and is in 2 virtual networks.

According to [vagrant-libvirt doc](https://github.com/vagrant-libvirt/vagrant-libvirt#management-network):

    > vagrant-libvirt uses a private network to perform some management
    > operations on VMs. All VMs will have an interface connected to this
    > network and an IP address dynamically assigned by Libvirt unless you
    > set :mgmt_attach to 'false'. This is in addition to any networks you
    > configure. The name and address used by this network are configurable
    > at the provider level.

The default name is `vagrant-libvirt`, you can view it in `virt-manage` or `virsh net-list --all`.

This feature may confused MAAS sometimes and cause trouble.
As the doc says, we can set `:mgmt_attach` to `false` to disable management network.
However, there will be another error when run `vagrant up`:

    > Management network can't be disabled when VM use box.
    > Please fix your configuration and run vagrant again.

No idea how can we not use `vm.box`, give up.
Back to the errors.

For example, you may see this error while commission:

    > "E: Unable to locate package lldpd."

To work around, you can disable proxy:

    Settings -> Network services -> Proxy -> Don't use a proxy -> save

You may also see this while try to deploy:

    > "Error: Node must be connected to a network."

In this case, you can mark the node as broken, and edit the interface,
to add it to a VLAN and auto assign it IP.
