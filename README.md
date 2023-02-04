# Ansible-K8S
## Description:
This project is going to deply a K8S cluster by using Ansible. It cloud deploy a simple cluster or a highly available cluster. It contains 3 roles (**role-lbg**, **role-containerd**, and **role-k8s**), a simple Ansible configuration file (**ansible.cfg**), a inventory file (**hosts*) and etc.
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


