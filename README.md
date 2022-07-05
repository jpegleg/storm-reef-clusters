# storm-reef-clusters 🌀🪸🌀
Ansible template for (microk8s) kubernetes cluster configuration.
This is designed for Ubuntu(22) but should work on other debian-based distros.
Adjust the "apt" related refernces and confirm the rsyslog.conf settings to customize for another distro.

- permissive global network policy template
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

#### Notes about policy and node joining order

The global network policy should be applied before a node is joined to ensure the node follows the policy immediately.

#### Calicoctl version

The calicoctl version installed is ahead of the calico version used in microk8s, so the --allow-version-mismatch flag is required. You might align those if desired.
