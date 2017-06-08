# -*- mode: ruby -*-
# vi: set ft=ruby :
# sudo apt-get install docker-ce=<VERSION>
# issues:
#       see https://superuser.com/questions/1148055/cant-access-web-from-local-to-vagrant-host

$script = <<SCRIPT
echo I am provisioning...
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-get remove docker docker-engine
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
sudo cp /vagrant/sources.list /etc/apt/sources.list
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt-get update -y

echo provisioning docker

sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual
apt-cache policy docker-engine
sudo apt-get install -y docker-engine=1.12.6-0~ubuntu-xenial
sudo usermod -aG docker $USER

docker pull gcr.mirrors.ustc.edu.cn/google_containers/kube-apiserver-amd64:v1.6.4
docker pull gcr.mirrors.ustc.edu.cn/google_containers/kube-controller-manager-amd64:v1.6.4
docker pull gcr.mirrors.ustc.edu.cn/google_containers/kube-scheduler-amd64:v1.6.4
docker pull gcr.mirrors.ustc.edu.cn/google_containers/kube-proxy-amd64:v1.6.4
docker pull quay.io/coreos/flannel:v0.7.1-amd64

docker pull gcr.mirrors.ustc.edu.cn/google_containers/etcd-amd64:3.0.17
docker pull gcr.mirrors.ustc.edu.cn/google_containers/pause-amd64:3.0
docker pull gcr.mirrors.ustc.edu.cn/google_containers/k8s-dns-sidecar-amd64:1.14.1
docker pull gcr.mirrors.ustc.edu.cn/google_containers/k8s-dns-kube-dns-amd64:1.14.1
docker pull gcr.mirrors.ustc.edu.cn/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.1

docker tag gcr.mirrors.ustc.edu.cn/google_containers/kube-apiserver-amd64:v1.6.4 gcr.io/google_containers/kube-apiserver-amd64:v1.6.0
docker tag gcr.mirrors.ustc.edu.cn/google_containers/kube-controller-manager-amd64:v1.6.4 gcr.io/google_containers/kube-controller-manager-amd64:v1.6.0
docker tag gcr.mirrors.ustc.edu.cn/google_containers/kube-scheduler-amd64:v1.6.4 gcr.io/google_containers/kube-scheduler-amd64:v1.6.0
docker tag gcr.mirrors.ustc.edu.cn/google_containers/kube-proxy-amd64:v1.6.4 gcr.io/google_containers/kube-proxy-amd64:v1.6.0
docker tag gcr.mirrors.ustc.edu.cn/google_containers/etcd-amd64:3.0.17 gcr.io/google_containers/etcd-amd64:3.0.17
docker tag gcr.mirrors.ustc.edu.cn/google_containers/pause-amd64:3.0 gcr.io/google_containers/pause-amd64:3.0
docker tag gcr.mirrors.ustc.edu.cn/google_containers/k8s-dns-sidecar-amd64:1.14.1 gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.1
docker tag gcr.mirrors.ustc.edu.cn/google_containers/k8s-dns-kube-dns-amd64:1.14.1 gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.1
docker tag gcr.mirrors.ustc.edu.cn/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.1 gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.1

curl -LO http://7pn5d3.com1.z0.glb.clouddn.com/k8s/releases/v1.6.4/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
sudo apt-get install -y --allow-unauthenticated kubelet kubeadm kubernetes-cni


sudo sysctl -w net.ipv4.ip_forward=1

date > /etc/vagrant_provisioned_at
SCRIPT

$kubemaster = <<SCRIPT
sudo sed 's/127\.0\.0\.1.*cluster.*/192\.168\.33\.10 cluster/' -i /etc/hosts

kubeadm init --apiserver-advertise-address 192.168.33.10 --pod-network-cidr 10.244.0.0/16 --token 8c2350.f55343444a6ffc46

sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf

curl -O https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel-rbac.yml
kubectl create -f kube-flannel-rbac.yml

curl -O https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml

curl -O https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml

kubectl proxy --address=192.168.33.10 --accept-hosts='^localhost$,^192\.168\.33\.10$,^\[::1\]$'
SCRIPT

$kubeagent1 = <<SCRIPT
sudo sed 's/127\.0\.0\.1.*node1.*/192\.168\.33\.20 node1/' -i /etc/hosts
kubeadm join --token=8c2350.f55343444a6ffc46 192.168.33.10:6443
SCRIPT

$kubeagent2 = <<SCRIPT
sudo sed 's/127\.0\.0\.1.*node2.*/192\.168\.33\.30 node2/' -i /etc/hosts
kubeadm join --token=8c2350.f55343444a6ffc46 192.168.33.10:6443
SCRIPT

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/xenial64"
  config.vm.provision :shell, inline: $script

  config.vm.define "cluster" do |cluster|
    cluster.vm.hostname = "cluster"
    cluster.vm.provision :shell, inline: $kubemaster
    cluster.vm.network "private_network", ip: "192.168.33.10"
    cluster.vm.provider "virtualbox" do |v|
       v.memory = 4096
       v.cpus = 4
     end
  end

  config.vm.define "node1" do |node|
    node.vm.hostname = "node1"
    node.vm.provision :shell, inline: $kubeagent1
    node.vm.network "private_network", ip: "192.168.33.20"

    node.vm.provider "virtualbox" do |v|
       v.memory = 2048
       v.cpus = 2
     end
  end

  config.vm.define "node2" do |node|
    node.vm.hostname = "node2"
    node.vm.provision :shell, inline: $kubeagent2
    node.vm.network "private_network", ip: "192.168.33.30"

    node.vm.provider "virtualbox" do |v|
       v.memory = 2048
       v.cpus = 2
     end
  end

  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
