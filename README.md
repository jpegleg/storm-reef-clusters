# storm-reef-clusters 🌀🪸
Ansible template for (microk8s) kubernetes cluster configuration.

- permissive global network policy template
- calico eBPF dataplane with wireguard and DSR


Example inventory:

```
[reef]
reef1
reef2
reef3
reef4
reef5
reef6
reef7

[control-node]
reef6

[added-control-plane]
reef5
reef7

[workers]
reef4
reef3
reef3
reef1

```

Example usage:

```
ansible-playbook -u root -i hosts.inventory build-reef.yml
```

To only execute the cluster joins and apply network use the `form` tag (skip install calicoctl and snap):

```
ansible-playbook -u root -i hosts.inventory build-reef.yml --tags form
```

To only apply the network manifest only:

```
ansible-playbook -u root -i hosts.inventory build-reef.yml --tags net
```

