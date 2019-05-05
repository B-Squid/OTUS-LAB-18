# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
   :inetRouter => {
        :box_name => "centos/6",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '192.168.255.1', libvirt__netmask: "255.255.255.252", libvirt__network_name: "router-net"},
                ]
   },
  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2',libvirt__netmask: "255.255.255.252", libvirt__network_name: "router-net"},
                   {ip: '192.168.0.1', libvirt__netmask: "255.255.255.240", libvirt__network_name: "dir-net"},
                   {ip: '192.168.0.33', libvirt__netmask: "255.255.255.240", libvirt__network_name: "hw-net"},
                   {ip: '192.168.0.65', libvirt__netmask: "255.255.255.192", libvirt__network_name: "mgt-net"},
                ]
  },
  
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', libvirt__netmask: "255.255.255.240", libvirt__network_name: "dir-net"},
#                   {ip: '', libvirt__netmask: "", libvirt__adapter: 3, libvirt__network_name: true},
#                   {adapter: 4, auto_config: false, libvirt__network_name: true},
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
        when "inetRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sysctl net.ipv4.conf.all.forwarding=1
            iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
            SHELL
        when "centralRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sysctl net.ipv4.conf.all.forwarding=1
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            SHELL
        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            SHELL
        end

      end

  end
  
  
end
