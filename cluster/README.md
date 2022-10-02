Execute in all nodes
====================
sudo apt-get update

Install docker in all nodes
============================
sudo apt-get install docker.io -y
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker

sudo apt-get install -y apt-transport-https ca-certificates curl -y
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

sudo kubeadm init --pod-network-cidr 172.18.0.0/16 --apiserver-advertise-address 10.0.0.2

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

#kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/tigera-operator.yaml

# Modify the pod-network-cidr before executing the command
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/custom-resources.yaml

#Wait until each pod has the STATUS of Running.
watch kubectl get pods -n calico-system

kubectl taint nodes --all node-role.kubernetes.io/control-plane- node-role.kubernetes.io/master-

kubeadm join 10.0.0.2:6443 --token cbnf8a.c6v3ulnbr0p2ck7o --discovery-token-ca-cert-hash sha256:936e77f8f4223aee391d77ed7a76acda178d3f793490ab73e61653f2271109c8

On every worker node
sudo touch /etc/default/kubelet
sudo vi /etc/default/kubelet
add the following line
KUBELET_EXTRA_ARGS="--node-ip=10.0.0.3"

sudo systemctl daemon-reload
sudo systemctl restart kubelet.service