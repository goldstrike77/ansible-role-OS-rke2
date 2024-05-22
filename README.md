![](https://img.shields.io/badge/Ansible-rke2-green.svg?logo=angular&style=for-the-badge)

>__Please note that the original design goal of this role was more concerned with the initial installation and bootstrapping environment, which currently does not involve performing continuous maintenance, and therefore are only suitable for testing and development purposes,  should not be used in production environments. The author does not guarantee the accuracy, completeness, reliability, and availability of the role content. Under no circumstances will the author be held responsible or liable in any way for any claims, damages, losses, expenses, costs or liabilities whatsoever, including, without limitation, any direct or indirect damages for loss of profits, business interruption or loss of information.__

>__请注意，此角色的最初设计目标更关注初始安装和引导环境，目前不涉及执行连续维护，因此仅适用于测试和开发目的，不应在生产环境中使用。作者不对角色内容之准确性、完整性、可靠性、可用性做保证。在任何情况下，作者均不对任何索赔，损害，损失，费用，成本或负债承担任何责任，包括但不限于因利润损失，业务中断或信息丢失而造成的任何直接或间接损害。__

__Table of Contents__

- [Overview](#overview)
- [Requirements](#requirements)
  * [Operating systems](#operating-systems)
  * [RKE2 versions](#rke2-versions)
- [ Role variables](#Role-variables)
  * [Main Configuration](#Main-parameters)
  * [Other Configuration](#Other-parameters)
- [Dependencies](#dependencies)
- [Example Playbook](#example-playbook)
  * [Hosts inventory file](#Hosts-inventory-file)
  * [Vars in role configuration](#vars-in-role-configuration)
  * [Combination of group vars and playbook](#combination-of-group-vars-and-playbook)
- [License](#license)
- [Author Information](#author-information)
- [Donations](#Donations)

## Overview
RKE2, also known as RKE Government, is Rancher's next-generation Kubernetes distribution, As a CNCF-certified Kubernetes distribution that runs entirely within Docker containers, RKE2 securely solves the frustration of installation complexity by removing most host dependencies and presenting a stable path for deployment, upgrades, and rollbacks.

## Requirements
### Operating systems
This role will work on the following operating systems:

  * CentOS 7, 8

### RKE2 versions

The following list of supported the RKE2 releases:

* 1.28

## Role variables
### Main parameters #
There are some variables in defaults/main.yml which can (Or needs to) be overridden:

##### General parameters
* `rke2_is_install`: A boolean to determine whether or not to install the Rancher Kubernetes Engine v2.
* `rke2_version`: Specify the Rancher Kubernetes Engine v2 version.
* `rke2_node_role`: Type of nodes in cluster, server or agent.
* `rke2_debug`: A boolean to determine whether or not to turn on debug logs.
* `rke2_cluster_domain`: Specify the Cluster Domain.
* `rke2_egress_selector_mode`: Specify the egress mode, One of 'agent', 'cluster', 'pod', 'disabled'.
* `rke2_disable_kube_proxy`: A boolean to determine whether or not to running kube-proxy.
* `rke2_disable_cloud_controller`: A boolean to determine whether or not to disable rke2 default cloud controller manager.
* `rke2_cloud_provider_external`: A boolean to determine whether or not to specify that the cloud-provider is external.
* `rke2_disable`: Disable deploy packaged components.
* `rke2_server`: The address or DNS name of Server to used to join a cluster.
* `rke2_node_extra_labels`: Defined node labels.

##### Role dependencies
* `rke2_haproxy_dept`: A boolean to determine whether or not use HAProxy as a load balancer.
* `rke2_haproxy_api`: Specify the port to be used for the apiserver load-balancer.
* `rke2_haproxy_server`: Specify the port to be used for the server load-balancer.
* `rke2_haproxy_proxy_args`: Specify Essential configuration sections.

##### CNI parameters
* `rke2_cni`: Specify the Container Network Interface parameters.

##### Kubelet parameters
* `rke2_kubelet_arg`: Specify the Kubelet's configuration parameters.

##### Auditing parameters
* `rke2_kube_audit`: Specify the auditing configuration parameters.

##### Backup parameters
* `rke2_backup`: Specify the Backup configuration parameters.

##### Service Mesh
* `environments`: Define the service environment.
* `datacenter`: Define the DataCenter.
* `domain`: Define the Domain.
* `customer`: Define the customer name.
* `tags`: Define the service custom label.
* `exporter_is_install`: Whether to install prometheus exporter.
* `consul_public_register`: Whether register a exporter service with public consul client.
* `consul_public_exporter_token`: Public Consul client ACL token.
* `consul_public_http_prot`: The consul Hypertext Transfer Protocol.
* `consul_public_clients`: List of public consul clients.
* `consul_public_http_port`: The consul HTTP API port.

### Other parameters
There are some variables in vars/main.yml:

## Dependencies
- Ansible versions >= 2.15
- Python >= 3.11.2

## Example

### Hosts inventory file
See tests/inventory for an example.

    node01 ansible_host='192.168.1.10' rke2_version="1.28.8"

### Vars in role configuration
Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: all
      roles:
         - role: ansible-role-OS-rke2
           rke2_version: "1.28.8"

### Combination of group vars and playbook
```yaml
rke2_version: "1.28.8"
rke2_node_role: "agent"
rke2_debug: false
rke2_cluster_domain: "cluster.local"
rke2_egress_selector_mode: "agent"
rke2_disable_kube_proxy: false
rke2_disable_cloud_controller: true
rke2_cloud_provider_external: true
rke2_disable:
  - "rke2-canal"
  - "rke2-coredns"
  - "rke2-ingress-nginx"
  - "rke2-metrics-server"
rke2_server: '192.168.0.150'
rke2_node_extra_labels:
  - 'longhorn/node=true'
rke2_haproxy_dept: false
rke2_haproxy_api: "6444"
rke2_haproxy_server: "9346"
rke2_haproxy_proxy_args:
  - name: "k8s-api"
    port: "{{ rke2_haproxy_api }}"
    syntax:
      - "listen k8s-api-{{ rke2_haproxy_api }}"
      - "  bind *:{{ rke2_haproxy_api }}"
      - "  mode tcp"
      - "  option tcp-check"
      - "  balance roundrobin"
      - "  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 256 maxqueue 256 weight 100"
      - "  server master1 192.168.0.151:6443 check"
      - "  server master2 192.168.0.152:6443 check"
      - "  server master3 192.168.0.152:6443 check"
  - name: "rke2-server"
    port: "{{ rke2_haproxy_server }}"
    syntax:
      - "listen rke2-server-{{ rke2_haproxy_server }}"
      - "  bind *:{{ rke2_haproxy_server }}"
      - "  mode tcp"
      - "  option tcp-check"
      - "  balance roundrobin"
      - "  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 256 maxqueue 256 weight 100"
      - "  server master1 192.168.0.151:9345 check"
      - "  server master2 192.168.0.152:9345 check"
      - "  server master3 192.168.0.152:9345 check"
rke2_cni:
  provider: "calico"
  pod_cidr: "10.42.0.0/16"
  srv_cidr: "10.43.0.0/16"
rke2_kubelet_arg:
  kubeapiburst: 100
  kubeapiqps: 50
  logmaxfiles: "10"
  logmaxsize: "20Mi"
  imagegchighthresholdpercent: "85"
  imagegclowthresholdpercent: "80"
  maxpods: "110"
  serializeimagepulls: true
  unsafesysctls:
    - "net.core.somaxconn"
    - "net.ipv4.ip_local_port_range"  
rke2_kube_audit:
  logmaxage: "20"
  logmaxbackup: "10"
  logmaxsize: "100"
  logtruncatemaxbatchsize: "32768"
  logtruncatemaxeventsize: "32768"
rke2_backup:
  s3: true
  endpoint: "obs.home.local:9000"
  retention: "5"
  access_key: "admin"
  secret_key: "password"
  bucket: "backup"
  region: "us-east-1"
  folder: "rke-it-prd-shared-toolchain-01"
  insecure: true
customer: "it"
environments: "prd"
project: "shared"
group: "toolchain"
tags:
  subscription: "default"
  owner: "nobody"
  department: "Infrastructure"
  organization: "The Company"
  region: "China"
exporter_is_install: false
consul_public_register: false
consul_public_exporter_token: "00000000-0000-0000-0000-000000000000"
consul_public_http_prot: "https"
consul_public_http_port: "8500"
consul_public_clients:
  - "127.0.0.1"
```

## License
![](https://img.shields.io/badge/MIT-purple.svg?style=for-the-badge)

## Author Information
Please send your suggestions to make this role better.

## Donations
Please donate to the following monero address.

    46CHVMbb6wQV2PJYEbahb353SYGqXhcdFQVEWdCnHb6JaR5125h3kNQ6bcqLye5G7UF7qz6xL9qHLDSAY3baagfmLZABz75
<p><img src="https://raw.githubusercontent.com/goldstrike77/goldstrike77.github.io/master/img/xmr_address.png" align="left" /></p>