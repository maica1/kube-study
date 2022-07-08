# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  
  (1..3).each do |i|
    config.vm.define "node#{i}" do |node|
    
      node.vm.box = "fedora/36-cloud-base"
      node.vm.box_version = "36-20220504.1"
      node.vm.box_download_insecure = true

      # VM basic config
      node.vm.hostname = "k8s-node-#{i}"
      node.disksize.size = '30GB'
      node.vm.network "forwarded_port", guest: 22, host: 2200 + i

      node.vm.provider "virtualbox" do |v|
        v.memory = "4096"
        v.cpus = "2"
        v.name = "k8s-node-#{i}"
        v.customize ["modifyvm", :id, "--description", "Connect to this machine using SSH: \nHost: 127.0.0.1 | Port: 220+node_number \nUsername: root | Pass: customer \n regular user with secure permissions"]
        
      end
      node.vm.provision "shell", inline: $docker_config
      node.vm.provision "shell", inline: $kube_config
    end
  end

  $docker_config = <<-EOS
    mkdir -p /mnt
    sudo dnf -y update
    sudo dnf -y install docker 
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

  $kube_config = <<-EOS
    cat > /etc/modules-load.d/k8s.conf << EOF
      br_netfilter
      ip_vs
      ip_vs_rr
      ip_vs_sh
      ip_vs_wrr
      nf_conntrack_ipv4
    EOF
    sudo modprobe br_netfilter ip_vs ip_vs_rr ip_vs_sh ip_vs_wrr nf_conntrack_ipv4
    sudo dnf -y update
    cat << EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
      [kubernetes]
      name=Kubernetes
      baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
      enabled=1
      gpgcheck=1
      gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
      EOF
    sudo yum install -y kubectl kubelet kubeadm
    sudo kubeadm config images pull
  EOS

  
  # Install vagrant vbguest plugin if not exists
  # na pipeline: vagrant plugin install --local
  config.vagrant.plugins = [{"vagrant-vbguest" => {"version" => "0.30.0"}},"vagrant-disksize"]
  #config.ssh.private_key_path
  #config.vagrant.sensitive = ["MySecretPassword", ENV["MY_TOKEN"]]
end


