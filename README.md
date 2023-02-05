# Ansible-K8S
## Description:
This project is going to deply a K8S cluster by using Ansible. It cloud deploy a simple cluster or a highly available cluster. It contains 3 roles (**role-lbg**, **role-containerd**, and **role-k8s**), a simple Ansible configuration file (**ansible.cfg**), a inventory file (**hosts**) and etc.
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
Currently, it only support on "Ubuntu 20.04.5 LTS".

## REQUIREMENT

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

PS: This is just my practice. You don't know to follow if you are familiar with ansible and linux.

### Step 1. Prepare an ansible node (Rocky Linux 8.x):

1.1 Choose minimal installation for OS install
1.1.1 Create a user during the installation, my user is myadmin
1.2 After system is installed, update system and install related packages.
```
# Run as root
dnf -y update && dnf -y install epel-release && dnf -y install ansible git
```
1.3 Set SELinux to permissive mode (Which mode actually is not impacted. But I will set it to permissive )
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

# Demo 1, deploy a simple k8s cluster

My demo environment:

- One Anisble node
- One Master node:
    - hostname: master01
    - host ip: 192.168.1.3
- Three worker nodes:
    - hostname: worker01, worker02, and worker03
    - ips: 192.168.1.6, 192.168.1.7, and 192.168.1.8 

## Deployment step 1

1.1 Clone the repo

```
# login to ansible node as myadmin, and make a Projects folder and clone the repo

mkdir Projects && cd Projects
git clone https://github.com/williamlui00/Ansible-K8S.git
cd Ansible-K8S

```

1.2 Modify ansible inventory, 

```
vi hosts
### Don't change the gourp name or delete the group
### Add a host entry under related group
### Host entry format: HOSTNAME ansible_ssh_host=HOST-IP

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

1.3 Make a ansible hot command to test, and you will see the output is "...uid=0(root)..."

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
###          If you are not going to deploying multiple master nodes, this k8s_vip should be MASTER-NODE-IP
### k8s_version: This playbook is going to install "1.25.0"
### pod_cidr: This ip pool will be will by pods. If change it, make sure is /16
### k8s_packages: The verison should be the same as k8s_version.
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

