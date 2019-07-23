# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV["LC_ALL"] = "en_US.utf8"
ENV["VAGRANT_DEFAULT_PROVIDER"] = "libvirt"

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false

  config.vm.box = "generic/ubuntu1804"
  config.vm.hostname = "ubuntu"

  config.vm.provider "libvirt" do |v|
    v.memory = "4096"
    v.cpus = 4
    v.storage_pool_name = "default"
    v.storage_pool_path = "/var/lib/libvirt/images"
    v.default_prefix = "maas"
  end

  config.vm.define "controller" do |v|
    v.vm.network "private_network",
      libvirt__network_name: 'maas',
      libvirt__domain_name: 'maas',
      libvirt__forward_mode: 'nat',
      libvirt__dhcp_enabled: false,
      libvirt__netmask: '255.255.255.0',
      ip: '192.168.100.100'
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y language-pack-en
    update-locale LC_ALL=en_US.utf8

    apt-get install -y libvirt-clients maas

    maas init \
      --admin-username admin \
      --admin-password admin \
      --admin-email admin@maas \
      --admin-ssh-import lp:guoqiao

    echo "access url: http://192.168.100.100:5240/MAAS"
    echo "enable DHCP: Subnets -> click VLAN -> Take action -> Provide DHCP"

  SHELL

end
