# Ansible Sabayon Roles

Mission of the project is supply useful roles for configure your Sabayon nodes
through Red Hat [Ansible](https://www.ansible.com) tool.

## LXD Playbooks

### Configure Sabayon standalone LXD node or first Cluster node

```bash
 $# ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook --limit node1 playbooks/lxd-configure.yml
```

### Add new node to Cluster

If not available fetch primary node cluster certificate:

```bash
  $# ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook   --limit node1 playbooks/lxd-configure.yml --skip-tags systemd_stuff,lxd_boostrap_node
```

Add nodes to Cluster

```bash
  $# ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook   --limit node2,node3,node4 playbooks/lxd-add-cluster-node.yml
```

