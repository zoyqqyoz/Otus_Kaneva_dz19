# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:inetRouter => {
        :box_name => "centos/7",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                ]
  },
  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},
                   {ip: '192.168.255.9', adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
                   {ip: '192.168.255.5', adapter: 7, netmask: "255.255.255.252", virtualbox__intnet: "office2-central"},

                ]
  },

  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"}
                ]
  },

  :office1Router => {
    :box_name => "centos/7",
    :net => [
               {ip: '192.168.255.10', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
               {ip: '192.168.2.1', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "dev-office1-net"},
               {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "testservers-office1-net"},
               {ip: '192.168.2.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "managers-net"},
               {ip: '192.168.2.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "hardware-office1-net"}
            ]
  },

  :office1Server => {
    :box_name => "centos/7",
    :net => [
               {ip: '192.168.2.130', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "dev-office1-net"}
            ]
  },

  :office2Router => {
    :box_name => "centos/7",
    :net => [
              {ip: '192.168.255.6', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
              {ip: '192.168.1.1', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "dev-office2-net"},
              {ip: '192.168.1.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "testservers-office2-net"},
              {ip: '192.168.1.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "hardware-office2-net"}
            ]
  },

  :office2Server => {
    :box_name => "centos/7",
    :net => [
               {ip: '192.168.1.2', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "dev-office2-net"}
            ]
  }
}

Vagrant.configure("2") do |config|

 config.vm.synced_folder ".", "/vagrant", disabled: true
  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ip: ipconf[:ip]
        end

        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL

        case boxname.to_s
        when "inetRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sudo bash -c 'echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf'; sudo sysctl -p
            sudo yum install -y iptables-services; sudo systemctl enable iptables && sudo systemctl start iptables;
            sudo iptables -F; sudo iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE; sudo service iptables save
            sudo bash -c 'echo "192.168.0.0/16 via 192.168.255.2 dev eth1" > /etc/sysconfig/network-scripts/route-eth1'; sudo systemctl restart network
            sudo reboot
            SHELL
        when "centralRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sudo bash -c 'echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf'; sudo sysctl -p
            sudo bash -c 'echo "192.168.2.0/24 via 192.168.255.10" > /etc/sysconfig/network-scripts/route-eth5'
            sudo bash -c 'echo "192.168.1.0/24 via 192.168.255.6" >> /etc/sysconfig/network-scripts/route-eth6'
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo reboot
            SHELL
        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sudo sed -i '$a DEFROUTE="no"' /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo reboot
            SHELL
        when "office1Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sudo bash -c 'echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf'; sudo sysctl -p
            sudo sed -i '$a DEFROUTE="no"' /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.255.9" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo reboot
            SHELL
        when "office1Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sudo sed -i '$a DEFROUTE="no"' /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.2.129" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo reboot
            SHELL
        when "office2Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sudo bash -c 'echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf'; sudo sysctl -p
            sudo sed -i '$a DEFROUTE="no"' /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.255.5" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo reboot
            SHELL
         when "office2Server"
           box.vm.provision "shell", run: "always", inline: <<-SHELL
            sudo sed -i '$a DEFROUTE="no"' /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.1.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo reboot
            SHELL
        end
      end
  end
end
