# Kubernetes Installation


///AT BOTH MASTER(KMASTER) AND SLAVE (KNODE)

-Master atleast 2 core CPUs & 4Gb RAM
-Node atleast 1 core CPUs & 4Gb RAM


///Master Setup


//Update the repository
$ sudo su 
# apt-get update


//Turn off swap space 
# swapoff -a
# nano /etc/fstab  [come down last line and put '#/swapfile' to disable that]


//Update hostname
# nano /etc/hostname  [Rename 'parvez-master']


//Note the ip adress
# ifconfig


//Set a static ip address
# nano /etc/network/interfaces [check the ip address static or DHCP]
[-add the below at the end of the file-]
[ auto enp2s0
  iface enp2s0 inet static
  address 192.168.1.6 ]

  
//update the host file  
# nano /etc/hosts [add ip address and name '192.168.1.6  parvez-master']


//Restart the machine [hostname is changed]
$ ifconfig   [ip is not changed]


//Install OpenSSH server
$ sudo su
# sudo apt-get install openssh-server


//Install Docker
# apt-get update
# apt-get install -y docker.io


//Run the following command before installing the kube environment
# apt-get update && apt-get install -y apt-transport-https curl
# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
# cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
# apt-get update


//Install Kubeadm,Kubelet & Kubectl
# apt-get install -y kubelet kubeadm kubectl
# exit


//Update the Kubernetes configuration
$ sudo su
# nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
  [add below after the environment last line]
  [Environment="cgroup-driver=systemd/cgroup-driver=cgroupfs"]
#exit




///Slave Setup


//Go to the VM terminal and give same setup

 
//Update the repository
$ sudo su 
# apt-get update


//Turn off swap space
# swapoff -a
# nano /etc/fstab   [come down last line and put '#/swapfile' to disable that]


//Update hostname
# nano /etc/hostname   [Rename 'parvez1-slave']


//Note the ip adress
# ifconfig


//Set a static ip address
# nano /etc/network/interfaces [check the ip address static or DHCP]
[-add the below at the end of the file-]
[ auto enps0
  iface enps0 inet static
  address 10.0.2.15 ]
  
  
  
//update the host file  
# nano /etc/hosts [add ip address and name '10.0.2.15  parvez1-slave']


//Restart the machine [hostname is changed]
$ ifconfig   [ip is not changed]


//Install OpenSSH server
$ sudo su
# sudo apt-get install openssh-server


//Install Docker
# apt-get update
# apt-get install -y docker.io


//Run the following command before installing the kube environment
# apt-get update && apt-get install -y apt-transport-https curl
# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
# cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
# apt-get update


//Install Kubeadm,Kubelet & Kubectl
# apt-get install -y kubelet kubeadm kubectl
# exit


//Update the Kubernetes configuration
$ sudo su
# nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
  [add below after the environment last line]
  [Environment="cgroup-driver=systemd/cgroup-driver=cgroupfs"]
#exit


$ ifconfig
$ cat /etc/hosts  [check the hostname]
$cat /etc/network/interfaces  [check the staic ip address]
$ sudo nano /etc/hosts
  [add ip address of master]
  [192.168.1.6   parvez-master]


//Go to main machine(parvez-master)
$ cat /etc/hosts  [check the hostname]
$ sudo nano /etc/hosts
  [add ip address of master]
  [10.0.2.15   parvez1-slave]


//KUBERNETES ENVIRONMENT ESTABLISHED
//RESTART THE BOTH MACHINE    
  
-------------------------------------------------------------------------


///ONLY MASTER SETUP


    
$ sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.168.1.6
  [For starting a Calico CNI: 192.168.0.0/16 or For starting a Flannel CNI: 10.244.0.0/16]
  

//Copy the join number of machine
[later use on slave terminal to join cluster: kubeadm join 192.168.1.6:6443 --token xrjzjy.rc4i5mm6jauprdg0 \
    --discovery-token-ca-cert-hash sha256:1bd9a270628d4577d8d39f5bd4bf6222d640d069b2ef389100997fb0366327c5]  
    
    
//Run the foillowing command as normal user  
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config


//For creating a POD network based on Calico
$ kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml


//Detailed status of PODS
$ kubectl get pods -o wide --all-namespaces


//Status of Nodes
$ kubectl get nodes


//Status of PODS
$ kubectl get pods --all-namespaces


//For creating the dashboard first - bring this up before starting nodes
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
$ kubectl get pods -o wide --all-namespaces


//To enable proxy and continues with new terminal window
$ kubectl proxy


//Dashboard & coreDNS panding
$ export kubever=$(kubectl version | base64 | tr -d '\n')
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
$ kubectl get pods -o wide --all-namespaces


//goning to the browser and search ber= localhost:8001


//For accessing the dashboard 
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login {or} http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/



//Getting error 
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "no endpoints available for service \"kubernetes-dashboard\"",
  "reason": "ServiceUnavailable",
  "code": 503
}


//endpoint not available
$ kubectl -n kubernetes-dashboard get endpoints -o wide


//To create a service account for your dashboard
  open new terminal
$ kubectl create serviceaccount dashboard -n default


//To add cluster binding rules for ur roles on dashboard
$ kubectl create clusterrolebinding dashboard-admin -n default \
  --clusterrole=cluster-admin \
  --serviceaccount=default:dashboard
  
  
//To get the secret key to be pasted into the dashboard token pwd. copy the outcoming secret key
$ kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode  



//For generating token again
$ sudo kubeadm create --print-join-command


///ONLY SLAVE SETUP


//join the cluster
copy the join number and paste
$ sudo kubeadm join 192.168.1.6:6443 --token xrjzjy.rc4i5mm6jauprdg0 \
    --discovery-token-ca-cert-hash sha256:1bd9a270628d4577d8d39f5bd4bf6222d640d069b2ef389100997fb0366327c5]
    

//Going to dashboard-click (nodes) 


//Going to master terminal
$ kubectl get nodes    
