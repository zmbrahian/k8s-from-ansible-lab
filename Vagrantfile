# -*- mode: ruby -*-
# vi: set ft=ruby :
BOX_IMAGE="generic/rhel8"
NODEWORKERSCOUNT= 3
Vagrant.configure("2") do |config|
    (2..NODEWORKERSCOUNT) .each do |i|
        config.vm.define "rhel8-k8s0#{i}" do |subconfig|
            subconfig.vm.box = BOX_IMAGE
            subconfig.vm.hostname = "rhel8-k8s0#{i}"
            subconfig.vm.network "private_network", ip: "192.168.1.2#{i}"
            subconfig.vm.provider "virtualbox" do |v|
                v.name = "rhel8-k8s0#{i}"
                v.memory = 2048
                v.cpus = 2
                v.customize ["storageattach", "rhel8-k8s0#{i}", "--storagectl", "IDE Controller", "--port", "1", "--device", "1", "--type", "dvddrive", "--medium", "D:\\ISOs\\rhel-8.0-x86_64-dvd.iso"]
                end
            subconfig.vm.provision "file", source: "repo/media.repo", destination: "/tmp/"
            subconfig.vm.provision "file", source: "keys/id_rsa.pub", destination: "/tmp/"
            subconfig.vm.provision "shell", inline: <<-SHELL
              sudo cp /tmp/media.repo /etc/yum.repos.d/
              sudo mount /dev/sr0 /mnt
              sudo yum repolist
              sudo dnf install cockpit git python3 python3-pip -y
              sudo systemctl enable --now cockpit.socket
              sudo firewall-cmd --add-service=cockpit
              sudo firewall-cmd --add-service=cockpit --permanent
              sudo dnf module install container-tools -y
              sudo useradd ansible ;  echo "t00lk1t" | passwd --stdin ansible
              sudo echo "ansible ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
              sudo mkdir ~/.ssh
              sudo cat /tmp/id_rsa.pub >> ~/.ssh/authorized_keys
            SHELL
        end
    end
    config.vm.define "rhel8-k8s01" do |subconfig|
        subconfig.vm.box = BOX_IMAGE
        subconfig.vm.hostname = "rhel8-k8s01"
        subconfig.vm.network "private_network", ip: "192.168.1.21"
        subconfig.vm.provider "virtualbox" do |v|
            v.name = "rhel8-k8s01"
            v.memory = 2048
            v.cpus = 2
            v.customize ["storageattach", "rhel8-k8s01", "--storagectl", "IDE Controller", "--port", "1", "--device", "1", "--type", "dvddrive", "--medium", "D:\\ISOs\\rhel-8.0-x86_64-dvd.iso"]
            end
        subconfig.vm.provision "file", source: "repo/media.repo", destination: "/tmp/"
        subconfig.vm.provision "file", source: "playbooks/initial.yml", destination: "/tmp/"
        subconfig.vm.provision "file", source: "playbooks/master.yml", destination: "/tmp/"
        subconfig.vm.provision "file", source: "playbooks/workers.yml", destination: "/tmp/"
        subconfig.vm.provision "file", source: "playbooks/kube-dependencies.yml", destination: "/tmp/"
        subconfig.vm.provision "file", source: "keys/id_rsa.pub", destination: "/tmp/"
        subconfig.vm.provision "file", source: "keys/id_rsa", destination: "/tmp/"
        subconfig.vm.provision "file", source: "inventory/hosts", destination: "/tmp/"
        subconfig.vm.provision "file", source: "daemon.json", destination: "/tmp/"
        subconfig.vm.provision "shell", inline: <<-SHELL
            sudo cp /tmp/media.repo /etc/yum.repos.d/
            sudo mount /dev/sr0 /mnt
            sudo yum repolist
            sudo dnf install cockpit git python3 python3-pip -y
            sudo systemctl enable --now cockpit.socket
            sudo firewall-cmd --add-service=cockpit
            sudo firewall-cmd --add-service=cockpit --permanent
            sudo dnf module install container-tools -y
            sudo useradd ansible ;  echo "t00lk1t" | passwd --stdin ansible
            sudo echo "ansible ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
            sudo pip3 install ansible --user
            sudo mv /root/.local/bin/* /usr/bin/ ; mkdir ~/playbooks ; mkdir ~/inventory ; mv /tmp/hosts ~/inventory/ ; mv /tmp/*.yml ~/playbooks/
            sudo mkdir ~/.ssh
            sudo ssh-keyscan -H 192.168.1.21 >> ~/.ssh/known_hosts
            sudo ssh-keyscan -H 192.168.1.22 >> ~/.ssh/known_hosts
            sudo ssh-keyscan -H 192.168.1.23 >> ~/.ssh/known_hosts
            sudo cat /tmp/id_rsa.pub >> ~/.ssh/authorized_keys ; mv /tmp/id_rsa* ~/.ssh/ ; chown root:root ~/.ssh/id_rsa* ; chmod 600 ~/.ssh/id_rsa
            sudo ansible -m ping -i ~/inventory/hosts all
            sudo ansible-playbook ~/playbooks/kube-dependencies.yml -i ~/inventory/hosts
            sudo ansible-playbook ~/playbooks/initial.yml -i ~/inventory/hosts
            sudo ansible-playbook ~/playbooks/master.yml -i ~/inventory/hosts
            sudo ansible-playbook ~/playbooks/workers.yml -i ~/inventory/hosts
            SHELL
    end
end