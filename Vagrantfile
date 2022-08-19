# -*- mode: ruby -*-
# vi: set ft=ruby :

NUM_MASTER_NODE = 2
NUM_WORKER_NODE = 2

IP_NW = "10.240.0."
MASTER_IP_START = 10
NODE_IP_START = 20
LB_IP_START = 30

HOSTS_FILE = <<-EOF
10.240.0.21 node1
10.240.0.22 node2
10.240.0.11 cp1
10.240.0.12 cp2
EOF

DOCKER_CONFIG = <<-EOS
# mkdir -p /mnt
sudo apt-get update -y 
mkdir -p /etc/docker
sudo apt-get -y install docker.io
cat > /etc/docker/daemon.json << EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
mkdir -p /etc/systemd/system/docker.service.d
sudo systemctl daemon-reload
sudo systemctl enable docker --now
EOS
OS_CONFIG = <<-EOS
echo "#{HOSTS_FILE}" >> /etc/hosts
#echo "\#{AUTH_KEY}" >> /root/.ssh/authorized_keys" || true
apt-get update -y
apt-get install -y jq tmux curl wget vim
echo "alias k=kubectl" >> /etc/bash.bashrc
echo "complete -o default -F __start_kubectl k" >> /etc/bash_completion.d/kubectl

# wget -q --show-progress --https-only  \
#   https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
#   https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
# chmod +x cfssl cfssljson
# mv cfssl cfssljson /usr/local/bin/
# wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl
# chmod +x kubectl
# sudo mv kubectl /usr/local/bin/

EOS

KUBE_CONFIG = <<-EOS
cat > /etc/modules-load.d/k8s.conf << EOF
  br_netfilter
  ip_vs
  ip_vs_rr
  ip_vs_sh
  ip_vs_wrr
  nf_conntrack_ipv4
EOF
sudo modprobe br_netfilter ip_vs ip_vs_rr ip_vs_sh ip_vs_wrr nf_conntrack_ipv4
sudo apt-get update -y 
# cat << EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
#   [kubernetes]
#   name=Kubernetes
#   baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
#   enabled=1
#   gpgcheck=1
#   gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
#   EOF
# sudo apt-get install -y kubectl kubelet kubeadm
# sudo kubeadm config images pull
EOS


Vagrant.configure("2") do |config|
  config.vm.boot_timeout = 600
  config.vbguest.installer_options = { allow_kernel_upgrade: true }
  (1..NUM_MASTER_NODE).each do |j|
    config.vm.define "cp#{j}" do |cp|
    
      cp.vm.box = "ubuntu/focal64"
      cp.vm.box_download_insecure = true

      # VM basic config
      cp.vm.hostname = "k8s-master-#{j}"
      cp.vm.disk :disk, size: '80GB', primary: true
      cp.vm.network :private_network, ip: IP_NW + "#{MASTER_IP_START + j}"
      cp.vm.network "forwarded_port", guest: 22, host: "#{2710 + j}"

      cp.vm.provider "virtualbox" do |v|
        v.memory = "2048"
        v.cpus = "2"
        v.name = "k8s-cp-#{j}"
        v.customize ["modifyvm", :id, "--description", "Connect to this machine using SSH: \nHost: 127.0.0.1 | Port: 2710+node_number \nUsername: root | Pass: customer \n regular user with secure permissions"]
        
      end
      # cp.vm.provision "shell", inline: DOCKER_CONFIG
      # cp.vm.provision "shell", inline: KUBE_CONFIG
      cp.vm.provision "shell", inline: OS_CONFIG
    end
  end
  (1..NUM_WORKER_NODE).each do |i|
    config.vm.define "node#{i}" do |node|
    
      node.vm.box = "ubuntu/focal64"
      node.vm.box_download_insecure = true

      # VM basic config
      node.vm.hostname = "k8s-node-#{i}"
      node.vm.disk :disk, size: '60GB', primary: true
      node.vm.network :private_network, ip: IP_NW + "#{NODE_IP_START + i}"
      node.vm.network "forwarded_port", guest: 22, host: "#{2720 + i}"

      node.vm.provider "virtualbox" do |v|
        v.memory = "1024"
        v.cpus = "2"
        v.name = "k8s-node-#{i}"
        v.customize ["modifyvm", :id, "--description", "Connect to this machine using SSH: \nHost: 127.0.0.1 | Port: 2720+node_number \nUsername: root | Pass: customer \n regular user with secure permissions"]
      end
      node.vm.provision "shell", inline: DOCKER_CONFIG
      node.vm.provision "shell", inline: KUBE_CONFIG
      node.vm.provision "shell", inline: OS_CONFIG
    end
  end
  
  # Install vagrant vbguest plugin if not exists
  # na pipeline: vagrant plugin install --local
  config.vagrant.plugins = ["vagrant-vbguest"]
  #config.ssh.private_key_path
  #config.vagrant.sensitive = ["MySecretPassword", ENV["MY_TOKEN"]]
end
