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


