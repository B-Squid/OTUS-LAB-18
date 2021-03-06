# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  #routers
  :inetRouter => {
        :box_name => "centos/6",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net =>[ {ip: '192.168.255.1', libvirt__netmask: "255.255.255.252", libvirt__network_name: "router-net", auto_config: false},]
   },

  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', libvirt__netmask: "255.255.255.252", libvirt__network_name: "router-net"},
                   {ip: '192.168.0.1', libvirt__netmask: "255.255.255.240", libvirt__network_name: "dir-net"},
                   {ip: '192.168.0.33', libvirt__netmask: "255.255.255.240", libvirt__network_name: "hw-net"},
                   {ip: '192.168.0.65', libvirt__netmask: "255.255.255.192", libvirt__network_name: "wifi-net"},
		   {ip: '192.168.100.1', libvirt__netmask: "255.255.255.252", libvirt__network_name: "office1-net"},
		   {ip: '192.168.200.1', libvirt__netmask: "255.255.255.252", libvirt__network_name: "office2-net"},
                ]
  },
#ПОРАСКИДАЙ ПО 2 IP НА ОДИН ИНТЕРФЕЙС
  :office1Router => {
        :box_name => "centos/7",
        :net => [
		   {ip: '192.168.100.2',libvirt__netmask: "255.255.255.252", libvirt__network_name: "office1-net"},
                   {ip: '192.168.2.1',libvirt__netmask: "255.255.255.192", libvirt__network_name: "dev1-net"},
		   {ip: '192.168.2.65', libvirt__netmask: "255.255.255.192", libvirt__network_name: "office1_hard-net"},
		   {ip: '192.168.2.129', libvirt__netmask: "255.255.255.192", libvirt__network_name: "managers-net"},
		   {ip: '192.168.2.193', libvirt__netmask: "255.255.255.192", libvirt__network_name: "test1_serv-net"},
                ]
  },

  :office2Router => {
        :box_name => "centos/7",
        :net => [
		   {ip: '192.168.200.2',libvirt__netmask: "255.255.255.252", libvirt__network_name: "office2-net"},
		   {ip: '192.168.0.126', libvirt__netmask: "255.255.255.192", libvirt__network_name: "wifi-net"},
      	           {ip: '192.168.1.1', libvirt__netmask: "255.255.255.128", libvirt__network_name: "dev2-net"},
		   {ip: '192.168.1.129', libvirt__netmask: "255.255.255.192", libvirt__network_name: "office2_hard-net"},
		   {ip: '192.168.1.193', libvirt__netmask: "255.255.255.192", libvirt__network_name: "test2_serv-net"},
                ]
  },
 
  #servers
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', libvirt__netmask: "255.255.255.240", libvirt__network_name: "dir-net"},
                  # {libvirt__adapter: 3, libvirt__network_name: "true1", auto_config: true},
                  # {adapter: 4, auto_config: false, libvirt__network_name: true},
                ]
  },

  :office1Server => {
        :box_name => "centos/7",
        :net => [ {ip: '192.168.2.2', libvirt__netmask: "255.255.255.192", libvirt__network_name: "dev1-net"}, ]
  },

  :office2Server => {
        :box_name => "centos/7",
        :net => [ {ip: '192.168.1.2', libvirt__netmask: "255.255.255.128", libvirt__network_name: "dev2-net"}, ]
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s

	box.vm.provider :libvirt do |libvirt, override|
	  libvirt.memory = 448
	end

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
        when "inetRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "NM_CONTROLLED=no
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.168.255.1
NETMASK=255.255.255.252
DEVICE=eth1
PEERDNS=no
ARPCHECK=no" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "net.ipv4.ip_forward=1" >> /etc/sysctl.d/99.conf
	    sysctl -p /etc/sysctl.d/99.conf 
            iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
	    /etc/init.d/network restart
            SHELL

        when "centralRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
	    echo "net.ipv4.ip_forward=1" >> /etc/sysctl.d/99-sysctl.conf
	    sysctl -p 
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "DEFROUTE=yes" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "192.168.1.0/25 via 192.168.200.2 dev eth6" > /etc/sysconfig/network-scripts/route-eth6
	    echo "192.168.1.128/26 via 192.168.200.2 dev eth6" >> /etc/sysconfig/network-scripts/route-eth6
	    echo "192.168.1.192/26 via 192.168.200.2 dev eth6" >> /etc/sysconfig/network-scripts/route-eth6
	    echo "192.168.2.0/26 via 192.168.100.2 dev eth5" > /etc/sysconfig/network-scripts/route-eth5
	    echo "192.168.2.64/26 via 192.168.100.2 dev eth5" >> /etc/sysconfig/network-scripts/route-eth5
	    echo "192.168.2.128/26 via 192.168.100.2 dev eth5" >> /etc/sysconfig/network-scripts/route-eth5
	    echo "192.168.2.192/26 via 192.168.100.2 dev eth5" >> /etc/sysconfig/network-scripts/route-eth5
            systemctl restart network

            SHELL

	when "office1Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
	    echo "net.ipv4.ip_forward=1" >> /etc/sysctl.d/99-sysctl.conf
	    sysctl -p
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
	    echo "GATEWAY=192.168.100.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "DEFROUTE=yes" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "192.168.0.2/28 via 192.168.100.1 dev eth1" > /etc/sysconfig/network-scripts/route-eth1
	    echo "192.168.0.32/28 via 192.168.100.1 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
	    echo "192.168.0.64/26 via 192.168.100.1 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
            systemctl restart network
            SHELL

	when "office2Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
	    echo "net.ipv4.ip_forward=1" >> /etc/sysctl.d/99-sysctl.conf
	    sysctl -p
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
	    echo "GATEWAY=192.168.200.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "DEFROUTE=yes" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "192.168.0.2/28 via 192.168.200.1 dev eth1" > /etc/sysconfig/network-scripts/route-eth1
	    echo "192.168.0.32/28 via 192.168.200.1 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
	    echo "192.168.0.64/26 via 192.168.200.1 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
            systemctl restart network
            SHELL

        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "DEFROUTE=yes" >> /etc/sysconfig/network-scripts/ifcfg-eth1 
            systemctl restart network
            SHELL

        when "office1Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.2.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "DEFROUTE=yes" >> /etc/sysconfig/network-scripts/ifcfg-eth1 
            systemctl restart network
            SHELL

        when "office2Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.1.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "DEFROUTE=yes" >> /etc/sysconfig/network-scripts/ifcfg-eth1 
            systemctl restart network
            SHELL
        end
      end
  end
end
