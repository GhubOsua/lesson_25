# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouterlesson25 => {
        :box_name => "centos7",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                ]
  },
  :centralRouterlesson25 => {
        :box_name => "centos7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.254.1', adapter: 4, netmask: "255.255.255.252", virtualbox__intnet: "router-net1"},
                   {ip: '192.168.253.1', adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "router-net2"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.0.33', adapter: 5, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.0.65', adapter: 7, netmask: "255.255.255.192", virtualbox__intnet: "wifi-net"},
                ]
  },
  :office1Routerlesson25 => {
        :box_name => "centos7",
        :net => [
                   {ip: '192.168.254.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net1"},
                   {ip: '192.168.2.129', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "mgmt-net1"},
                   {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test-net1"},
                   {ip: '192.168.2.1', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "dev-net1"},
                   {ip: '192.168.2.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "hw-net1"},
                ]
  },
  :office2Routerlesson25 => {
        :box_name => "centos7",
        :net => [
                   {ip: '192.168.253.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net2"},
                   {ip: '192.168.1.193', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "hw-net2"},
                   {ip: '192.168.1.1', adapter: 4, netmask: "255.255.255.128", virtualbox__intnet: "dev-net2"},
                   {ip: '192.168.1.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "test-net2"},
                ]
  },  
  :centralServerlesson25 => {
        :box_name => "centos7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                  #{adapter: 3, auto_config: false, virtualbox__intnet: true},
                  #{adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  },
  :office1Serverlesson25 => {
        :box_name => "centos7",
        :net => [
                   {ip: '192.168.2.200', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "hw-net1"},
                ]
  },
  :office2Serverlesson25 => {
        :box_name => "centos7",
        :net => [
                   {ip: '192.168.1.200', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "hw-net2"},
                ]
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        
        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        
        case boxname.to_s
        when "inetRouterlesson25"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sysctl net.ipv4.conf.all.forwarding=1
            iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
            ip route add 192.168.0.0/28 dev eth1 via 192.168.255.2 dev eth1 #for centralServerlesson25
            ip route add 192.168.2.192/26 via 192.168.255.2 dev eth1 #for office1Serverlesson25
            ip route add 192.168.1.192/26 via 192.168.255.2 dev eth1 #for office2Serverlesson25
            ip route add 192.168.254.0/30 via 192.168.255.2 dev eth1 #for office1Routerlesson25
            ip route add 192.168.253.0/30 via 192.168.255.2 dev eth1 #for office2Routerlesson25
            
            SHELL
        when "centralRouterlesson25"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sysctl net.ipv4.conf.all.forwarding=1
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1            
            systemctl restart network
            ip route add 192.168.2.192/26 via 192.168.254.2 #for office1Serverlesson25
            SHELL
        when "centralServerlesson25"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            SHELL
        when "office1Routerlesson25"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sysctl net.ipv4.ip_forward=1
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.254.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            SHELL
        when "office2Routerlesson25"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sysctl net.ipv4.ip_forward=1
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.253.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            SHELL
        when "office1Serverlesson25"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.2.193" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            SHELL
        when "office2Serverlesson25"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.1.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            SHELL
        end

      end

  end
  
  
end
