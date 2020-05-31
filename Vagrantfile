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

  :inetRouter2 => {
        :box_name => "centos/7",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '192.168.254.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net2"},
                   {ip: '192.168.11.151', adapter: 3, netmask: "255.255.255.0", name: "vboxnet0"} 
                ]
  },

  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.254.2', adapter: 4, netmask: "255.255.255.252", virtualbox__intnet: "router-net2"},
                ]
  },
  
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: true},
                   {adapter: 4, auto_config: false, virtualbox__intnet: true},
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
          box.vm.provision "file", source: "inetrouter.rules", destination: "/home/vagrant/iptables.rules"

          box.vm.provision "shell", run: "always", inline: <<-SHELL
            yum install -y iptables-services
            echo "192.168.0.0/16 via 192.168.255.2 dev eth0" > /etc/sysconfig/network-scripts/route-eth1
            systemctl restart network
            systemctl restart network
            sysctl net.ipv4.conf.all.forwarding=1
            systemctl start iptables 
            systemctl enable iptables
            iptables-restore < /home/vagrant/iptables.rules
            service iptables save
            sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
            systemctl restart sshd
            SHELL
        when "inetRouter2"
          box.vm.provision "file", source: "inetrouter2.rules", destination: "/home/vagrant/iptables.rules"

          box.vm.provision "shell", run: "always", inline: <<-SHELL
            yum install -y iptables-services
            echo "192.168.0.0/16 via 192.168.254.2 dev eth1" > /etc/sysconfig/network-scripts/route-eth1
            systemctl restart network
            systemctl restart network
            sysctl net.ipv4.conf.all.forwarding=1
            systemctl start iptables
            systemctl enable iptables
            iptables-restore < /home/vagrant/iptables.rules
            service iptables save
            SHELL
        when "centralRouter"
          box.vm.provision "file", source: "knock.sh", destination: "/home/vagrant/knock.sh"

          box.vm.provision "shell", run: "always", inline: <<-SHELL
            yum install -y nmap
            chmod +x /home/vagrant/knock.sh
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "192.168.11.0/24 via 192.168.254.1 dev eth3" > /etc/sysconfig/network-scripts/route-eth3
            systemctl restart network
            systemctl restart network
            sysctl net.ipv4.conf.all.forwarding=1
            SHELL
        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            yum install -y epel-release
            yum install -y nginx
            systemctl enable nginx
            systemctl restart nginx
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            systemctl restart network
            SHELL
        end

      end

  end
  
  
end

