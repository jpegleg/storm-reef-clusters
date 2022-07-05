# storm-reef-clusters 🌀🪸🌀
Ansible template for (microk8s) kubernetes cluster configuration.
This is designed for Ubuntu(22) but should work on other debian-based distros.
Adjust the "apt" related refernces and confirm the rsyslog.conf settings to customize for another distro.

- calico eBPF dataplane with wireguard and DSR
- deploy rsyslog.conf
- add wazuh repo and register wazuh agent

#### Ensure tha your wazuh manager/rsyslog  server IP is used in the rsyslog.conf and agent registration:

```
sed -i 's/SETMETOTHELOGGINGHOST/blah.blah.blah.yourlogginghost/g' files/rsyslog.conf
sed -i 's/SETMETOTHELOGGINGHOST/blah.blah.blah.yourlogginghost/g' build-reef.yml
```


#### Example inventory:

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

## Example usage:

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


To only apply the rsyslog.conf:

```
ansible-playbook -u root -i hosts.inventory build-reef.yml --tags rsyslog
```

To only register the wazuh agent:

```
ansible-playbook -u root -i hosts.inventory build-reef.yml --tags wazuh
```

To only install misc packages:

```
ansible-playbook -u root -i hosts.inventory build-reef.yml --tags packages
```


#### Later adding a node to the control plane or worker pool:

To manually do it, just the usual easy microk8s add-node:

```
root@reef6:~# microk8s add-node
From the node you wish to join to this cluster, run the following:
microk8s join 192.168.1.140:25000/eea74306aa30df26807fe0d739ce240a/fef7f9a2d5be

Use the '--worker' flag to join a node as a worker not running the control plane, eg:
microk8s join 192.168.1.140:25000/eea74306aa30df26807fe0d739ce240a/fef7f9a2d5be --worker

If the node you are adding is not reachable through the default interface you can use one of the following:
microk8s join 192.168.1.140:25000/eea74306aa30df26807fe0d739ce240a/fef7f9a2d5be
microk8s join 10.1.103.194:25000/eea74306aa30df26807fe0d739ce240a/fef7f9a2d5be
```

Instead of manual/other adding to the control plane or worker pool, the anisble tag "form" in the build-reef.yml will also join/re-join nodes from inventory:

```
ansible-playbook -u root -i hosts.ini build-reef.yml --tags form
```

#### Notes about rejoining and rebuilding

Once a node is joined, attempts to join it again will error. There are a few ays to handle this.
We can create additional inventory files that only contain new nodes when doing a cluster expansion. amd the designated control plane node that provides the token.
We can set ignore errors on node join in the ansible.
Or we can can rebuild the cluster completely.

Rebuilding the cluster is fairly quick. We can use `microk8s leave` on each node to dissolve the cluster entirely, or `microk8s remove-node $node` to remove a specific node from the cluster but there may be files or containers that do not get destroyed, that later cause conflict. We can avoid that by instead of doing a leave, completely remove the microk8s snap with `snap remove microk8s`, which will be better for rebuilding the cluster cleanly.

#### Calicoctl version

The calicoctl version installed is ahead of the calico version used in microk8s, so the --allow-version-mismatch flag is required. You might align those if desired.
