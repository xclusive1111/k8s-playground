# -*- mode: ruby -*-
# vi: set ft=ruby :

cluster = {
  "master" => [
    { :ip => "192.168.33.10", :cpus => 2, :mem => 2048, :ssh => 3210 },
    { :ip => "192.168.33.11", :cpus => 2, :mem => 2048, :ssh => 3211 },
    { :ip => "192.168.33.12", :cpus => 2, :mem => 2048, :ssh => 3212 }
  ],
  "worker" => [
    { :ip => "192.168.33.20", :cpus => 1, :mem => 2048, :ssh => 3220 },
    { :ip => "192.168.33.21", :cpus => 1, :mem => 2048, :ssh => 3221 },
    { :ip => "192.168.33.22", :cpus => 1, :mem => 2048, :ssh => 3222 }
  ],
  "lb" => [
    { :ip => "192.168.33.30", :cpus => 1, :mem => 1024, :ssh => 3230 }
  ]
}

hosts = "
192.168.33.10 master-0
192.168.33.11 master-1
192.168.33.12 master-2

192.168.33.20 worker-0
192.168.33.21 worker-1
192.168.33.22 worker-2

192.168.33.30 lb-0
"


# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/jammy64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 22, host: 2222

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  cluster.each do |(hostname, nodes)|

    nodes.each_with_index do |info, index|
      config.vm.define "#{hostname}-#{index}" do |node|
        node.vm.provider "virtualbox" do |vb|
           vb.name = "kube-#{hostname}-#{index}"
           vb.memory = info[:mem]
           vb.cpus = info[:cpus]
        end

        node.vm.hostname = "#{hostname}-#{index}"
        node.vm.network :private_network, ip: info[:ip]
        node.vm.network "forwarded_port", guest: 22, host: info[:ssh]

        node.vm.provision "base configuration", type: 'shell', inline: <<-SHELL
           echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
           echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
           modprobe br_netfilter
           apt-get update
           apt-get install -y wget curl net-tools
        SHELL

        # TODO
        $setup_hosts = <<-SCRIPT
        echo -e \"$1 $2 $3 $4\" >> /etc/hosts
        sed -e '/^.*ubuntu-jammy.*/d' -i /etc/hosts # Remove this entry
        sed -e '/^.*127.0.2.1.*/d' -i /etc/hosts    # Remove unnecessary entry, e.g: '127.0.2.1 master-0 master-0'
        sed -e '/^.*127.0.2.1.*/d' -i /etc/hosts    # Remove duplicated entry
        SCRIPT

        node.vm.provision "setup hosts", type: "shell" do |s|
          s.inline = $setup_hosts
          s.args = ["#{info[:ip]}", "#{hostname}-#{index}", "#{hostname}-#{index}.local", "#{hosts}"]
        end

      end
    end
  end

  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #    sysctl net.bridge.bridge-nf-call-iptables=1
  # SHELL
end
