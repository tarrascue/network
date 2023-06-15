MACHINES = {
:inetRouter => {
        :box_name => "centos/7",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '1.1.1.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                ]
  },
  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '1.1.1.2', adapter: 2, netmask: "255.255.255.252", gateway: "1.1.1.1", virtualbox__intnet: "router-net"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "wifi"},
                ]
  },

  :office1Router => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.35', adapter: 2, netmask: "255.255.255.240", gateway: "192.168.0.33", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.2.1', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "of1-dev-net"},
                   {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "of1-tst-srv"},
                   {ip: '192.168.2.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "of1-managers"},
		   {ip: '192.168.2.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "of1-hw"},
                ]
  },

  :office2Router => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.36', adapter: 2, netmask: "255.255.255.240", gateway: "192.168.0.33", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.1.1', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "of2-dev-net"},
                   {ip: '192.168.1.129', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "of2-tst-srv"},
                   {ip: '192.168.1.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "of2-hw"},
                ]
  },

###Servers  
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.40', adapter: 2, netmask: "255.255.255.240", gateway: "192.168.0.33", virtualbox__intnet: "hw-net"},
                  # {adapter: 3, auto_config: false, virtualbox__intnet: true},
                  # {adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  },

  :of1Server => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.2.70', adapter: 2, netmask: "255.255.255.192", gateway: "192.168.2.65", virtualbox__intnet: "of1-tst-srv"},
                  # {adapter: 3, auto_config: false, virtualbox__intnet: true},
                  # {adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  },
  
  :of2Server => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.1.130', adapter: 2, netmask: "255.255.255.192", gateway: "192.168.1.129", virtualbox__intnet: "of2-tst-srv"},
                  # {adapter: 3, auto_config: false, virtualbox__intnet: true},
                  # {adapter: 4, auto_config: false, virtualbox__intnet: true},
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
            
            iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
	    ip route add 192.168.0.0/22 via 1.1.1.2
	    sysctl net.ipv4.conf.all.forwarding=1
            SHELL
        when "centralRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
#           echo "GATEWAY=1.1.1.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network            
#	    ip route add default via 1.1.1.1
	    ip route add 192.168.0.50 via 192.168.0.33
	    ip route add 192.168.1.0/24 via 192.168.0.36
	    ip route add 192.168.2.0/24 via 192.168.0.35
	    sysctl net.ipv4.conf.all.forwarding=1
	    SHELL
        when "office1Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
#           echo "GATEWAY=192.168.0.33" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
#	    sudo ip route add default via 192.168.0.33
	    sysctl net.ipv4.conf.all.forwarding=1
    	    SHELL
	when "office2Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
#           echo "GATEWAY=192.168.0.33" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
	    sudo ip route add default via 192.168.0.33
	    sysctl net.ipv4.conf.all.forwarding=1
    	    SHELL
        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
#           echo "GATEWAY=192.168.0.33" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
#	    sudo ip route add default via 192.168.0.33
	    sysctl net.ipv4.conf.all.forwarding=1
    	    SHELL
        when "of1Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
#           echo "GATEWAY=192.168.2.65" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
#	    sudo ip route add default via 192.168.2.65
	    sysctl net.ipv4.conf.all.forwarding=1
            SHELL
        when "of2Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
#          echo "GATEWAY=192.168.1.129" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
#	    sudo ip route add default via 192.168.1.129
	    sysctl net.ipv4.conf.all.forwarding=1
            SHELL
       end

      end

  end
  
  
end
