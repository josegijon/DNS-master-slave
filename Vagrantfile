# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.vm.provision "shell", name: "update", inline: <<-SHELL
      apt-get update
      apt-get install -y bind9
  SHELL

  # master
  config.vm.define "tierra" do |tierra|
    tierra.vm.hostname = "tierra.sistema.test"
    tierra.vm.network "private_network", ip: "192.168.57.103"

    tierra.vm.provision "shell", name: "dns-master", inline: <<-SHELL
    #   cp -v /vagrant/named /etc/default
    #   cp -v /vagrant/named.conf.options /etc/bind
    #   cp -v /vagrant/named.conf.localmaster /etc/bind
    #   cp -v /vagrant/tierra.sistema.dns /var/lib/bind
    #   cp -v /vagrant/192.168.57.dns /var/lib/bind

    #   systemctl reload named
    #   systemctl status named
    SHELL
  end

    # slave
    config.vm.define "venus" do |venus|
      venus.vm.hostname = "venus.sistema.test"
      venus.vm.network "private_network", ip: "192.168.57.102"
  
      venus.vm.provision "shell", name: "dns-slave", inline: <<-SHELL
        cp -v /vagrant/named /etc/default
      #   cp -v /vagrant/named.conf.options /etc/bind
      #   cp -v /vagrant/named.conf.localslave /etc/bind
  
      #   systemctl reload named
      #   systemctl status named
      SHELL
    end
end
