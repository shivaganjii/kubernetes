# Dependencies in Kubernetes;

You can start with 3 Master nodes.

	RAM = 2GB or more per machin

	CPU = 2Core or more per machin

# Set Timezone and Host_fil;

	$ timedatectl set-timezone Asia/Tehran

	$ cat >> /etc/hosts << EOF
	172.16.10.1 master1.domain.com master1
	172.16.10.2 master2.domain.com master2
	172.16.10.3 master3.domain.com master3
	EOF

# Install container runtime;

I want install Docker.
Setup docker in all nodes;

	$ apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

	$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

	$ echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

	$ apt update

	$ apt install -y docker-ce docker-ce-cli containerd.io

	$ systemctl start docker && systemctl enable docker && systemctl status docker

# Install kubelet, kubectl and kubeadm;

Setup Kubelet, Kubectl and Kubeadm in all nodes;

	$ cat << EOF | sudo tee /etc/modules-load.d/k8s.conf
	br_netfilter
	EOF

	$ cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
	net.bridge.bridge-nf-call-ip6tables = 1
	net.bridge.bridge-nf-call-iptables = 1
	EOF

	$ sysctl --system
	$ sed -i '/swap/d' /etc/fstab
	$ swapoff -a

	$ apt update
	$ apt install -y apt-transport-https ca-certificates curl
	$ curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
	$ echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

	$ apt update
	$ apt install -y kubelet=1.22.2-00 kubeadm=1.22.2-00 kubectl=1.22.2-00  ### You can install another version ###
	$ apt-mark hold kubelet kubeadm kubectl  ### Holding version ###
	$ systemctl enable kubelet
	$ systemctl restart kubelet

	$ echo "source <(kubectl completion bash)" >> ~/.bashrc
	$ kubectl completion bash >/etc/bash_completion.d/kubectl
	$ echo 'alias k=kubectl' >>~/.bashrc
	$ echo 'complete -F __start_kubectl k' >>~/.bashrc
	$ source ~/.bashrc
	$ source /etc/bash_completion
	$ apt install -y fzf


# Create config_file for installation;

Create config_file in master1:

	$ cat > kubeadm_config.yml << EOF
	apiVersion: kubeadm.k8s.io/v1beta3
	bootstrapTokens:
	- groups:
	  - system:bootstrappers:kubeadm:default-node-token
	  token: gtf85d.sjsdxjfg54b8pfdf
	  ttl: 24h0m0s
	  usages:
	  - signing
	  - authentication
	kind: InitConfiguration
	localAPIEndpoint:
	  advertiseAddress: 172.16.10.1
	  bindPort: 6443
	nodeRegistration:
	  criSocket: /var/run/dockershim.sock
	  name: master1
	  taints:
	  - effect: NoSchedule
	    key: node-role.kubernetes.io/master
	---
	apiServer:
	  extraArgs:
	    authorization-mode: "Node,RBAC"
	  timeoutForControlPlane: 4m0s
	  certSANs:
	  - "172.16.10.1"
	  - "172.16.10.2"
	  - "172.16.10.3"
	  - "master1.domain.com"
	  - "master2.domain.com"
	  - "master3.domain.com"
	  - "master1"
	  - "master2"
	  - "master3"
	apiVersion: kubeadm.k8s.io/v1beta3
	certificatesDir: /etc/kubernetes/pki
	clusterName: kubernetes
	controllerManager: {}
	dns:
	etcd:
	  local:
	    imageRepository: "k8s.gcr.io"
	    dataDir: "/var/lib/etcd"
	    serverCertSANs:
	  - "172.16.10.1"
	  - "172.16.10.2"
	  - "172.16.10.3"
	  - "master1.domain.com"
	  - "master2.domain.com"
	  - "master3.domain.com"
	  - "master1"
	  - "master2"
	  - "master3"
	    peerCertSANs:
	  - "172.16.10.1"
	  - "172.16.10.2"
	  - "172.16.10.3"
	  - "master1.domain.com"
	  - "master2.domain.com"
	  - "master3.domain.com"
	  - "master1"
	  - "master2"
	  - "master3"
	kubernetesVersion: v1.22.2
	imageRepository: k8s.gcr.io
	kind: ClusterConfiguration
	controlPlaneEndpoint: "master1.domain.com:6443"
	networking:
	  dnsDomain: domain.com
	  serviceSubnet: 10.8.0.0/16
	  podSubnet: 10.9.0.0/16
	scheduler: {}
	EOF


# Create Cluster by config_file;

	$ kubeadm config images pull --config kubeadm_config.yml  ### Download images ###

	$ kubeadm init --config kubeadm_config.yml

	$ mkdir -p $HOME/.kube

	$ cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

	$ chown $(id -u):$(id -g) $HOME/.kube/config

Your cluster is ready.

# Install container network; 

I want install Calico:

	$ curl -O https://docs.projectcalico.org/v3.20/manifests/calico.yaml

	$ kubectl create -f calico.yaml

# Copy TLS to master2 and master3;

Copy TLS from master1:

	$ scp -r /etc/kubernetes/pki root@172.16.10.2:/etc/kubernetes/

	$ scp -r /etc/kubernetes/pki root@172.16.10.3:/etc/kubernetes/

# Join master2 and master3 in cluster;

	$ kubeadm join master1.domain.com:6443 --token [your_token] --discovery-token-ca-cert-hash sha256:[your_token_ca_cert_hash] --control-plane --apiserver-advertise-address 172.16.10.2

	$ kubeadm join master1.domain.com:6443 --token [your_token] --discovery-token-ca-cert-hash sha256:[your_token_ca_cert_hash] --control-plane --apiserver-advertise-address 172.16.10.3

Enter this commands in master2 and master3:

	$ mkdir -p $HOME/.kube
	$ cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	$ chown $(id -u):$(id -g) $HOME/.kube/config 

# check every thing is ready;

	$ kubeadm certs check-expiration

	$ kubectl get node -A -wo wide

	$ kubectl get pod -A -wo wide

	$ kubectl cluster-info

	$ kubectl describe node master1

	$ kubectl describe pod -n kube-system calico-node-####

	$ kubectl logs -n kube-system calico-node-####

	$ kubectl delete -n kube-system pod calico-node-####

	$ ...

