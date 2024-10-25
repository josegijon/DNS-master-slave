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
      cp -v /vagrant/comun/named /etc/default/named
      cp -v /vagrant/comun/named.conf.options /etc/bind
      cp -v /vagrant/master/named.conf.localmaster /etc/bind/named.conf.local
      cp -v /vagrant/comun/db.sistema.test /etc/bind
      cp -v /vagrant/comun/db.57.168.192 /etc/bind

      systemctl restart bind9
      systemctl status bind9
    SHELL
  end

    # slave
    config.vm.define "venus" do |venus|
      venus.vm.hostname = "venus.sistema.test"
      venus.vm.network "private_network", ip: "192.168.57.102"
  
      venus.vm.provision "shell", name: "dns-slave", inline: <<-SHELL
        cp -v /vagrant/comun/named /etc/default/named
        cp -v /vagrant/comun/named.conf.options /etc/bind
        cp -v /vagrant/slave/named.conf.localslave /etc/bind/named.conf.local
  
        systemctl restart bind9
        systemctl status bind9
      SHELL
    end
end
