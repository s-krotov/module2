# -*- mode: ruby -*-
# vi: set ft=ruby :

VM_COUNT = 5
LB_HOSTNAME = 'lb'

node_script = <<SCRIPT
  #!/bin/bash
  #install 
  yum install java-1.8.0-openjdk tomcat tomcat-webapps tomcat-admin-webapps -y

  #create sample data
  mkdir /usr/share/tomcat/webapps/test
  
  #starting node services
  systemctl enable tomcat
  systemctl start tomcat
  
  #TODO firewall setup vs disable
  systemctl stop firewalld
  systemctl disable firewalld
SCRIPT

lb_script = <<SCRIPT
  #!/bin/bash
  yum install httpd -y

  cp /vagrant/mod_jk.so /etc/httpd/modules/
  cp /vagrant/lb.conf /etc/httpd/conf.d/
  
  cp /vagrant/workers.template /etc/httpd/conf/workers.properties
  cat /vagrant/worker*.tmp >> /etc/httpd/conf/workers.properties
  rm /vagrant/worker*.tmp -f
  
  systemctl enable httpd
  systemctl start httpd
  systemctl restart httpd

  #TODO:create rules
  systemctl stop firewalld
  systemctl disable firewalld
SCRIPT



Vagrant.configure("2") do |config|
  config.vm.box = "bertvv/centos72"
  config.vm.provider "virtualbox" do |vb|
    #vb.gui = true
    vb.memory = 1024
  end

  (VM_COUNT-1).times do |i|
    node_id = 'vm' + i.to_s
    node_ip = '192.168.56.' + (11 + i).to_s
    config.vm.define node_id do |node|
      node.vm.hostname = "#{node_id}"
      node.vm.network :private_network, ip: "#{node_ip}", name: "VirtualBox Host-Only Ethernet Adapter"
      node.vm.provision :shell, :inline => node_script
      node.vm.provision :shell, inline: <<-SHELL
        #test data
        echo "<h1>i am vm#{i}</h1>" > /usr/share/tomcat/webapps/test/index.html
      
        #worker config for lb
        echo "worker.#{LB_HOSTNAME}.balance_workers=#{node_id}" >> /vagrant/worker#{i}.tmp
        echo "worker.#{node_id}.host=#{node_ip}" >> /vagrant/worker#{i}.tmp
        echo "worker.#{node_id}.port=8009" >> /vagrant/worker#{i}.tmp
        echo "worker.#{node_id}.type=ajp13" >> /vagrant/worker#{i}.tmp
      SHELL
    end
  end

  config.vm.define LB_HOSTNAME do |node|
    node.vm.hostname = LB_HOSTNAME
    node.vm.network :private_network, ip: "192.168.56.10", name: "VirtualBox Host-Only Ethernet Adapter" #name parameter is due https://github.com/hashicorp/vagrant/issues/6059#issuecomment-126834560
    node.vm.network "forwarded_port", guest: 80, host: 8080
    node.vm.provision :shell, :inline => lb_script
  end
end