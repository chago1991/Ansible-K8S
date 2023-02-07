# Ansible-K8S

### Description:

This project is going to deploy a K8S cluster by using Ansible. It could deploy a simple cluster or a highly available cluster. 

### How it works:

- Set up K8S nodes to be managed by Ansible Node
- Assign K8S node to Ansible Inventory Group (Ansible Inventory File - hosts). 
- Ansible Inventory Group associates with an Ansible role(s). 
- Ansible's roles (role-containerd, role-k8s, and role-lbg) define what needs to install and set up.
- The Ansible Playbook (deploy-cluster.yaml) will apply the related role(s) to the K8S node based on the assigned group. 
- The K8S node will act as a K8S role (Load Balancer Node, Master Node, or Worker Node) after running the playbook. 

```
.
├── ansible.cfg
├── deploy-cluster.yaml
├── hosts
├── README.md
├── roles
│   ├── role-containerd
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── files
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── meta
│   │   │   └── main.yml
│   │   ├── README.md
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── templates
│   │   │   ├── temp-containerd.conf.j2
│   │   │   └── temp-containerd.service.j2
│   │   ├── tests
│   │   │   ├── inventory
│   │   │   └── test.yml
│   │   └── vars
│   │       └── main.yml
│   ├── role-k8s
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── files
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── meta
│   │   │   └── main.yml
│   │   ├── README.md
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── templates
│   │   ├── tests
│   │   │   ├── inventory
│   │   │   └── test.yml
│   │   └── vars
│   │       └── main.yml
│   └── role-lbg
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       ├── handlers
│       │   └── main.yml
│       ├── meta
│       │   └── main.yml
│       ├── README.md
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       │   ├── temp-haproxy.j2
│       │   └── temp-keepalived.j2
│       ├── tests
│       │   ├── inventory
│       │   └── test.yml
│       └── vars
│           └── main.yml
└── templates
    └── temp-hosts.j2

29 directories, 33 files
```
**limitation**
Currently, it only support on "Ubuntu 20.04 LTS".

## Part 1. Requirement

Description: 
To setup an ansible node and other K8S nodes. Make sure the ansible node can manage other K8S nodes. If you aleady do that you can skip this part.

For deploy a simple cluster (One master node):
1. You need to prepare an ansible node
2. You need to prepare one master node
3. You need to prepare one or more worker nodes

For deploy a high availability cluster:
1. You need to prepare an ansible node
2. You need to prepare three or more master nodes (Odd number)
3. You need to prepare one or more worker nodes
4. You need to prepare two or more load balancer nodes to  install haproxy or keepalived

## Here are the steps to prepare the nodes both for deploy a simple cluster and a high availability cluster:

### Step 1. Prepare an ansible node:

1.1 Minimal installation Rocky Linux 8.x

1.1.1 Create a user during the installation, my user is myadmin, also setup ip and hostname

1.2 After system is installed, update system and install related packages.

```
# Run as root
dnf -y update && dnf -y install epel-release && dnf -y install ansible git
```

1.3 Set SELinux to permissive mode (Which mode actually is not impacted, but I will set it to permissive )

```
# Run as root
setenforce 0 && sed -i 's/SELINUX=.*/SELINUX=permissive/' /etc/selinux/config
```

1.4  Generate a SSH Key for myadmin

```
# Run as myadmin
ssh-keygen
```

### Setp 2. Prepare other nodes (master nodes, worker nodes and load balancer nodes):

2.1 Install Ubuntu 20.04.x LTS (also minimal installation) with openssh-server, also create a user myadmin

2.2 After installation, Generate a SSH Key for user myadmin

```
# Run as myadmin
ssh-keygen
```

2.3 Setup hostname, if you haven't set it

```
# Run as root
hostnamectl set-hostname XXX
```

2.4 Make user myadmin sudo without password

```
# Run as root, check "%sudo   ALL=(ALL:ALL) NOPASSWD: ALL"
vi /etc/sudoers
...(Skipped)
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
...(Skipped)

# Run as myadmin, should not ask for password
sudo su -

# (Optional) Run as root, if still ask for password
usermod -a -G sudo myadmin
```

2.4 Check /etc/hosts

```
# Run as root, check "127.0.1.1 THE-HOST-NAME" is correct
vi /etc/hosts
...(Skipped)
127.0.1.1 THE-HOST-NAME
...(Skipped)
```

### Step 3, copy myadmin's public key to other nodes

3.1 Login the ansible node as myadmin, copy myadmin's public key to other nodes one by one

```
# run as myadmin
ssh-copy-id OtherNodeIP
```

3.2 SSH remote login Testing

```
# run as myadmin on Ansible node, it should show "uid=0(root) gid=0(root) groups=0(root)“ without ask password
ssh OtherNodeIP "sudo id" 
```

# Part 2. Demo 1, deploy a simple k8s cluster

Description: 

This part is going to demo how to set up a simple K8S cluster. If you want to deploy a highly available cluster. Skip this part.

My demo environment:

- One Anisble node
- One Master node:
    - hostname: master01
    - host ip: 192.168.1.3
- Three worker nodes:
    - hostname: worker01, worker02, and worker03
    - ips: 192.168.1.6, 192.168.1.7, and 192.168.1.8 

1.1 Clone the repo

```
# login to ansible node as myadmin, and make a Demo1 folder and clone the repo

mkdir Demo1 && cd Demo1
git clone https://github.com/williamlui00/Ansible-K8S.git
cd Ansible-K8S

```

1.2 Modify ansible inventory, 

```
vi hosts
### Don't change the group name or delete the group
### Add a host entry under a related group
### Host entry format: HOSTNAME ansible_ssh_host=HOST-IP
### Empty host entry of a group that means the role will not be deployed

### The [lbg] is the load balancer nodes' group
[lbg]


### The [masterg] is the k8s control plane nodes' group
[masterg]
master01 ansible_ssh_host=192.168.1.3 ansible_ssh_user=myadmin

### The [workerg] is the k8s worker nodes' group
[workerg]
worker01 ansible_ssh_host=192.168.1.6 ansible_ssh_user=myadmin
worker02 ansible_ssh_host=192.168.1.7 ansible_ssh_user=myadmin
worker03 ansible_ssh_host=192.168.1.8 ansible_ssh_user=myadmin

```

1.3 Make an ansible hot command to test, and you will see the output is "...uid=0(root)..."

```
$ ansible all -m shell -a id -b
worker03 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)
worker02 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)
worker01 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)
master01 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)

```

1.4 Modify the deploy-cluster.yaml, only change the k8s_vip to "master01's ip"

PS: Ignore the frist k8s_vip, just change the second one.

```
vi deploy-cluster.yaml
...(Skipped)
- name: Deploy k8s nodes
  hosts: masterg,workerg
  become: yes
  vars:
### Change me(must)
### k8s_vip: If you are going to deploy Load Balancers, this k8s_vip should the same as above, e.g., 192.168.0.10
###          If you are not going to deploying multiple master nodes, this k8s_vip should be the MASTER-NODE-IP
### k8s_version: This playbook is going to install "1.25.0"
### pod_cidr: This ip pool will be used by the pods. If change it, make sure is /16
### k8s_packages: The verison should be the same as k8s_version. (End with "-00")
    k8s_vip: "192.168.1.3" 
    k8s_version: "1.25.0"
    pod_cidr: "10.10.0.0/16"
    k8s_packages:
    - "kubelet=1.25.0-00"
    - "kubeadm=1.25.0-00"
    - "kubectl=1.25.0-00"
  roles:
    - role-k8s
...(Skipped)
```
<img width="806" alt="image" src="https://user-images.githubusercontent.com/9592837/216815180-21c99e9d-117d-40c2-a668-b47ba01a637b.png">

1.5 Run the playbook - deploy-cluster.yaml

```
ansible-playbook deploy-cluster.yaml
```
You should see something like this, with failed=0
<img width="806" alt="image" src="https://user-images.githubusercontent.com/9592837/216815467-faf00312-2ebf-482a-9abb-1d3f222890eb.png">

1.6 Login to master01 as myadmin to check the nodes status, you may need to wait a few minutes.
```
kubectl get nodes

```
<img width="470" alt="image" src="https://user-images.githubusercontent.com/9592837/216815604-6eb49c4a-19ad-4192-bedb-747336f512e3.png">

1.7 Testing kubectl 
- Create a test deployment
```
kubectl create deployment --replicas 3 --image nginx my-nginx

```

<img width="471" alt="image" src="https://user-images.githubusercontent.com/9592837/216815762-93b1ee44-7aa3-4abc-a083-d4b09ce3fd31.png">

- Create get the pods
```
kubectl get pods
kubectl get pods -o wide
```
<img width="969" alt="image" src="https://user-images.githubusercontent.com/9592837/216815784-85951c5b-1958-4c99-8a0a-dbd235681d11.png">

<Demo1 Done>



# Demo 2, deploy a highly available cluster

Description:

This part is going to demo how to set up a highly available K8S cluster. The cluster is following "kubeadm HA topolagy - stacked etcd". 

<img width="632" alt="image" src="https://user-images.githubusercontent.com/9592837/217154972-48b4340b-5ec6-4fd8-9200-bb833d79138b.png">

Refer https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/

My demo environment:

- One Anisble node
- One Master node:
    - hostname: master01, master02, and master03
    - host ip: 192.168.1.3, 192.168.1.4, and 192.168.1.5
- Three worker nodes:
    - hostname: worker01, worker02, and worker03
    - ips: 192.168.1.6, 192.168.1.7, and 192.168.1.8
- Two load balancer nodes nodes:
    - hostname: lb01 and lb02
    - ips: 192.168.1.1, 192.168.1.2
- VIP for the load balancer: 192.168.1.0/22

2.1 Clone the repo

```
# login to ansible node as myadmin, and make a Demo2 folder and clone the repo

mkdir Demo2 && cd Demo2
git clone https://github.com/williamlui00/Ansible-K8S.git
cd Ansible-K8S

```

2.2 Modify ansible inventory, 

```
### Don't change the group name or delete the group
### Add a host entry under a related group
### Host entry format: HOSTNAME ansible_ssh_host=HOST-IP
### Empty host entry of a group that means the role will not be deployed
### If the ssh remote user is not myadmin, please change it. 

### The [lbg] is the load balancer nodes' group
[lbg]
lb01 ansible_ssh_host=192.168.1.1 ansible_ssh_user=myadmin
lb02 ansible_ssh_host=192.168.1.2 ansible_ssh_user=myadmin

### The [masterg] is the k8s control plane nodes' group
[masterg]
master01 ansible_ssh_host=192.168.1.3 ansible_ssh_user=myadmin
master02 ansible_ssh_host=192.168.1.4 ansible_ssh_user=myadmin
master03 ansible_ssh_host=192.168.1.5 ansible_ssh_user=myadmin

### The [workerg] is the k8s worker nodes' group
[workerg]
worker01 ansible_ssh_host=192.168.1.6 ansible_ssh_user=myadmin
worker02 ansible_ssh_host=192.168.1.7 ansible_ssh_user=myadmin
worker03 ansible_ssh_host=192.168.1.8 ansible_ssh_user=myadmin

```


2.3 Make an ansible hot command to test, and you will see the output is "...uid=0(root)..."

```
$ ansible all -m shell -a id -b
worker02 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)
worker03 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)
master01 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)
lb01 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)
worker01 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)
master03 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)
lb02 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)
master02 | CHANGED | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)

```
<img width="310" alt="image" src="https://user-images.githubusercontent.com/9592837/217158561-7b228474-72a1-4efa-85cf-ce5de5565dce.png">

2.4 Modify the deploy-cluster.yaml

```
vi deploy-cluster.yaml
...(Skipped)
- name: Deploy Load Balancers
  hosts: lbg
  become: yes
  vars:
### Change me if you are deploying multiple master nodes
### Change the k8s_vip that will set it to keepalived
### If you are going to deploy Load Balancers, you need to assign a VIP, e.g., 192.168.0.10/24.
    k8s_vip: '192.168.1.0/22'
  roles:
    - role-lbg
...(Skipped)

- name: Deploy k8s nodes
  hosts: masterg,workerg
  become: yes
  vars:
### Change me(must)
### k8s_vip: If you are going to deploy Load Balancers, this k8s_vip should the same as above, e.g., 192.168.0.10
###          If you are not going to deploying multiple master nodes, this k8s_vip should be MASTER-NODE-IP
### k8s_version: This playbook is going to install "1.25.0"
### pod_cidr: This ip pool will be will by pods. If change it, make sure is /16
### k8s_packages: The verison should be the same as k8s_version.
    k8s_vip: "192.168.1.0"
    k8s_version: "1.25.0"
    pod_cidr: "10.10.0.0/16"
...(Skipped)

```

<img width="885" alt="image" src="https://user-images.githubusercontent.com/9592837/217159377-14db3e9e-0ae8-42ca-8031-47510bf22dc7.png">


2.5 Run the playbook - deploy-cluster.yaml

```
ansible-playbook deploy-cluster.yaml
```

You should see the ansible play recap like this, with failed=0

<img width="832" alt="image" src="https://user-images.githubusercontent.com/9592837/217160817-396dcd73-737b-4f4d-b582-d41831b8ee5f.png">


2.6 Login to master01 as myadmin to check the nodes status. The nodes may take a few minutes to be ready.
```
kubectl get nodes
```

<img width="473" alt="image" src="https://user-images.githubusercontent.com/9592837/217161503-1801da0c-d187-486b-b93f-02840b564449.png">

2.7 Testing kubectl 

- Create a test deployment

```
kubectl create deployment --replicas 3 --image nginx my-nginx
```

- Create get the pods

```
kubectl get pods
kubectl get pods -o wide
```

<img width="433" alt="image" src="https://user-images.githubusercontent.com/9592837/217162785-eaa4674b-b7b6-4bb9-b772-17462d781742.png">

<img width="963" alt="image" src="https://user-images.githubusercontent.com/9592837/217162876-602646c9-6024-4f31-a911-7e5068966b37.png">

<Demo2 Done>


