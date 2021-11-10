# -*- mode: ruby -*-
# vi:set ft=ruby sw=2 ts=2 sts=2:

NUM_MASTER_NODE = 1
NUM_WORKER_NODE = 2

IP_NW = "192.168.10."
MASTER_IP_START = 10
NODE_IP_START = 20

Vagrant.configure("2") do |config|
  ssh_pub_key = File.readlines("C:\\Users\\kamal\\.ssh\\id_rsa.pub").first.strip
  config.vm.box = "debian/bullseye64"
  config.vm.box_check_update = false
  (1..NUM_MASTER_NODE).each do |i|
      config.vm.define "kubemaster" do |node|
        node.vm.provider "virtualbox" do |vb|
            vb.name = "kubemaster"
            vb.customize ["modifyvm", :id, "--groups", "/Kubernetes Cluster"]
            vb.memory = 2048
            vb.cpus = 2
        end
        node.vm.hostname = "kubemaster"
        node.vm.network :private_network, ip: IP_NW + "#{MASTER_IP_START + i}"
        node.vm.network "forwarded_port", guest: 22, host: "#{2710 + i}"
        node.vm.synced_folder "./share", "/share"
        node.vm.provision "setup-hosts", :type => "shell", :path => "scripts/setup-hosts.sh" do |s|
          s.args = ["enp0s8"]
        end
        node.vm.provision "setup-dns", type: "shell", :path => "scripts/update-dns.sh"
        node.vm.provision "shell", inline: <<-SHELL
            apt-get update && apt-get upgrade -y
            echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
            echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
            cp /share/.bashrc /home/vagrant/
            cp /share/.bashrc /root/
        SHELL
      end
  end

  (1..NUM_WORKER_NODE).each do |i|
    config.vm.define "kubenode0#{i}" do |node|
        node.vm.provider "virtualbox" do |vb|
            vb.name = "kubenode0#{i}"
            vb.customize ["modifyvm", :id, "--groups", "/Kubernetes"]
            vb.memory = 1024
            vb.cpus = 2
        end
        node.vm.hostname = "kubenode0#{i}"
        node.vm.synced_folder "./share", "/share"
        node.vm.network :private_network, ip: IP_NW + "#{NODE_IP_START + i}"
                node.vm.network "forwarded_port", guest: 22, host: "#{2720 + i}"
        node.vm.provision "setup-hosts", :type => "shell", :path => "scripts/setup-hosts.sh" do |s|
          s.args = ["enp0s8"]
        end
        node.vm.provision "setup-dns", type: "shell", :path => "scripts/update-dns.sh"
        node.vm.provision "shell", inline: <<-SHELL
            apt-get update && apt-get upgrade -y
            mkdir /root/.ssh -p
            mkdir /home/vagrant/.ssh -p
            echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
            echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
            cp /share/.bashrc /home/vagrant/
            cp /share/.bashrc /root/
        SHELL
    end
  end
end