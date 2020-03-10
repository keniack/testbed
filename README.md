# Testbed Setup

Hardware Description:

  - RaspberryPi 4 B+ 4GB
  - RaspberryPi 3 B+
  - OrangePi PC
  - BeagleBone Black  
 
### OS 

  - Hypriot OS: https://blog.hypriot.com/downloads/  1.12.0 installed
 

If iptables is 1.8+, switch to legacy version. Network flannel/weave still have incompabilities with the newest iptables version.

      sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
      sudo reboot
Make sure ip forwarding is enabled: 

      sudo sysctl net.bridge.bridge-nf-call-iptables=1
      
      
### Kubernetes

##### In all the nodes and master : 
 Installation:

    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
    apt-get update && apt-get install -y kubeadm
  
    cat > /etc/docker/daemon.json <<EOF
    {
  	"exec-opts": ["native.cgroupdriver=systemd"],
	"log-driver": "json-file",
	"log-opts": {
	    "max-size": "100m"
	},
	"storage-driver": "overlay2"
    }
    EOF

    mkdir -p /etc/systemd/system/docker.service.d

    systemctl daemon-reload
    systemctl restart docker
    

##### Master setup

for connections over eth0: 

     sudo kubeadm init --pod-network-cidr 10.244.0.0/16 

for connections over wlan0: 

     sudo kubeadm init --pod-network-cidr 10.244.0.0/16 --apiserver-advertise-address=<your IP>
     
After kubernetes initializes: 

     mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config

##### Node setup

After the master starts it will print a command which should be executed on the nodes. It looks like : 

    sudo kubeadm join 192.168.1.10:6443 --token 408cgt.q4wytvbb13qzgv8e --discovery-token-ca-cert-hash sha256:dff264f213a3cfea7ce2407d64385159d663cc8d07fbdac36bb7ea6e36de4d06 
    
##### Post-init setup 

Install Flannel (arm/arm64) for netowrk management (for hybrid clusters apply both files):

     kubectl apply -f flannel/kube-flannel_arm.yaml
     
     kubectl apply -f flannel/kube-flannel_arm64.yaml

Remove taints from master node to allows pods scheduling on the master as well:

     kubectl taint nodes --all node-role.kubernetes.io/master-    

Label nodes:

     kubectl label node ananas guava node-role.kubernetes.io/node=node
     
### Helm

     curl -O  https://get.helm.sh/helm-v3.1.1-linux-arm.tar.gz
     tar -zxvf helm-v3.1.1-linux-arm.tar.gz
     mv linux-arm/helm /usr/local/bin/helm
     rm -rf linux-arm

     helm repo add stable https://kubernetes-charts.storage.googleapis.com


PS: Helm 3 does not need to start tiller

### OpenFaas 

Installation via kubectl (only on the master node):

    git clone https://github.com/openfaas/faas-netes
    cd faas-netes
    
    kubectl apply -f ./namespaces.yml
    
    PASSWORD=$(head -c 12 /dev/urandom | shasum| cut -d' ' -f1)	
    kubectl -n openfaas create secret generic basic-auth --from-literal=basic-auth-user=admin --from-literal=basic-auth-password="$PASSWORD"
    
    kubectl apply -f ./yaml_armhf
     
### MinIO

    kubectl create -f minio/minio-volume.yaml
    kubectl create -f minio/minio-pvc.yaml
    kubectl create -f minio/minio-deployment.yaml
    kubectl create -f minio/minio-service.yaml

### Redis 



