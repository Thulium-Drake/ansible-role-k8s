# Plain Kubernetes, powered by Ansible
This role provides you with a Kubernetes cluster that is as clean as possible. The only thing this role sets up for you is CRI-O as the Container Runtime. Plugins for CNI, Storage etc. still need to be setup after this role has run.

It provides a cluster for the following scenarios:

* Multiple control_planes with (or without) workers
* Single control_plane with workers

Missing features:

* Configuring private repositories beyond initial kubeadm setup
* Removing nodes

## Setup
Most of this role is controlled via the Ansible Inventory, for example a simple 1 control_plane 2 worker cluster looks like this:

```
[k8s_cluster]
k8s1.example.com
k8s2.example.com
k8s3.example.com
```

All of the other aspects of this cluster are controlled via Group and Host variables, e.g. the following files:

```yaml
---
# Hosts variables for k8s1.example.com
k8s_role: 'control-plane'
```

```yaml
---
# Group variables for k8s_cluster
k8s_role: 'worker'
k8s_cluster_group: 'k8s_cluster'
```

For all the other options available, please refer to the Defaults.

After configuring the inventory, you can deploy the cluster by running a playbook like this:

```yaml
---
- name: 'Deploy K8s'
  hosts: 'k8s_cluster'
  roles:
    - 'k8s'
```

## Multiple control-planes
When using multiple control-planes in the cluster, the role will automatically find the first node that has the control-plane node in the inventory. Some important notes:

* To be as idempotent as possible the role sorts all nodes in dynamic groups on alphanumerical order
* When adding additional control-planes, make sure to set their hostnames to be alphanumerically later then existing nodes
* A multi control-plane cluster requires a load balanced API endpoint

## Notes, tips and tricks
When installing a CNI plugin (e.g. Calico), delete all then-current running pods in the cluster. This is required to re-assign them new IP addresses that are actually processed properly by the CNI. You can do this as follows:
```
kubectl delete pods -A --all
```
Afterwards all the pods will respawn with their proper IP address and will have network connectivity.
