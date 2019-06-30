# LXD Playbooks

LXD playbooks help Sabayon users to install and manage LXD in our Sabayon nodes.

> Every playbook MUST be executed under the directory `sabayon-ansible-roles` of the git project and for configure node you need use root access.

## Getting Started

### Configure options

Ansible permit to define a list of variables or for single host or for a group of hosts.

So, the first step todo is configure your options and your inventory with all your nodes.

Default inventory contains only localhost node.

Currently, in this roles all groups configuration are available under [group_vars/lxd](https://github.com/Sabayon/sabayon-ansible-roles/blob/master/inventory/group_vars/lxd)
and hereinafter the list of configuration options available:

| Parameter | Example | Description |
|-----------|---------|-------------|
| lxc_config | - | Define the configuration to set on /etc/lxc/lxc.conf file |
| lxd_cluster | true | If configure the node as a Cluster LXD (true) or Standalone node (false). |
| lxd_server_name | - | This option is present only as indication inside group_vars. Normally must be set as host variable for define the name of the LXD instance for a specific node. |
| lxd_bootstrap_node | - | Define the name (ansible_hostname) of the first node of the LXD Cluster. |
| lxd_cluster_address | - | Define the address of the Cluster Address used. |
| lxd_cluster_node_iface | eth0 | LXD interface to use for cluster node for retrieve the IP Address to set as server_address in every LXD node |
| lxd_cluster_node_port | 8443 | Define the port for binding of LXD service. |
| lxd_cluster_cert_file | - | Path where store locally LXD Cluster certificate used by new nodes of the Cluster |
| lxd_cluster_secret | mysecret | LXD Cluster password |
| lxd_http_proxy | - | Define if configure http_proxy for LXD instance |
| lxd_https_proxy | - | Define if configure https_proxy for LXD instance |
| lxd_no_proxy | - | Define if configure no_proxy variable for LXD instance. Normally all nodes of the Cluster if Proxy is used. |
| lxd_member_config | - | Define the `member_config` section for new node of LXD Cluster. |
| lxd_cluster_config | - | Define the `config` section for a nwe node of LXD Cluster. |
| lxd_config | - | Define the `config` section of LXD for the bootstrap node. |
| lxd_storage_pools | - | Define the `storage_pools` section. |
| lxd_networks | - | Define the `networks` section. |
| lxd_profiles | - | Define the `profiles` section with all LXD profiles to configure. |

For the details of LXD configuration see:

 * [Clustering](https://lxd.readthedocs.io/en/latest/clustering/)
 * [LXD documentation](https://github.com/lxc/lxd/tree/master/doc)

Currently, LXD doesn't support multiple DHCP instances, so for Cluster it's better works directly at Layer2 with
an external DHCP Server, with static ip addresses or with [FanNetworking](https://wiki.ubuntu.com/FanNetworking).

#### Configuration Examples

An example of profiles:

```yaml
lxd_profiles:
  - name: default
    devices:
      root:
        path: /
        pool: default
        type: disk
      eth0:
        name: eth0
        nictype: bridged
        parent: lxdbr0
        type: nic
   - name: privileged
     description: "Privileged profile"
     config:
       security.privileged: "true"
     devices:
       fuse:
         path: /dev/fuse
         type: unix-char
       tuntap:
         path: /dev/net/tun
         type: unix-char
       zfs:
         path: /dev/zfs
         type: unix-char

```

An example of configuration for bootstrap node configuration:

```yaml
lxd_config:
  config:
    core.trust_password: mysecret
    core.https_address: "[::]:8443"
    images.auto_update_interval: 15
```

> It seems that for cluster it's better define the server interface used instead of use `[::]`.


An example of configuration of the networks:

```yaml
lxd_networks:
  - name: lxdbr0
    type: bridge
    config:
      # ipv4.address: 192.168.200.14/24
      ipv4.address: none
      ipv4.firewall: false
      ipv4.nat: false
      ipv4.dhcp: false
      ipv6.address: none
      ipv6.dhcp: false
      dns.mode: none
```

### Configure Sabayon standalone LXD node or first Cluster node

After the configuration of group/host variables, execute this command under `sabayon-ansible-roles`
project directory:


```bash
 $# ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook --limit node1 playbooks/lxd-configure.yml
```

If all packages are already available on target host it's possible avoid check dependencies step:

```bash
ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook --limit node1 playbooks/lxd-configure.yml  --skip-tags install_deps
```

By default `lxd` group variables are configured for localhost node so for configure and bootstrap your LXD node
it's needed only this:

```bash
$# sudo su
$# equo i ansible
$# git clone https://github.com/Sabayon/sabayon-ansible-roles
$# cd sabayon-ansible-roles
$# ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook playbooks/lxd-configure.yml
$# # Now your node is configured with LXD!
```

** If you have already an installation of LXD it's better clean installation data: **

```bash
$# systemctl stop lxd
$# rm -rf /var/lib/lxd/*
$# ip link set down dev lxdbr0
$# brctl del-br lxdbr0
```

### Configure LXC stuff

For configure only LXC stuff could be used this command:

```bash
  $# ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook --limit node1 playbooks/lxd-configure.yml -t lxc_conf_override
```

### Add new node to Cluster


If not available fetch primary node cluster certificate on local host with:

```bash
  $# ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook   --limit node1 playbooks/lxd-configure.yml --skip-tags systemd_stuff,lxd_boostrap_node
```

Add nodes to Cluster

```bash
  $# ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook   --limit node2,node3,node4 playbooks/lxd-add-cluster-node.yml
```

### Create your first LXD Container

After that playbook is executed you can enable access to LXD socket at your user:

```bash
$# sudo su && gpasswd -a <user> lxd
```

and then create your first container:

```bash
$# lxc image list images:
$# lxc launch images:sabayon c1
$# # For execute a container inside container
$# lxc exec c1 /bin/bash
```

## Playbooks Availables

| Playbook | Description |
|----------|-------------|
|lxd-configure.yml | Create a Bootstrap Node of LXD Cluster or a Standalone node. |
|lxd-add-cluster-node.yml | Add a node to an existing LXD Cluster. |

