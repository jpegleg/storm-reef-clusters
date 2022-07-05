# storm-reef-clusters 🌀🪸
Ansible template for (microk8s) kubernetes cluster configuration.
This is designed for Ubuntu(22) but should work on other debian-based distros.
Adjust the "apt" related refernces and confirm the rsyslog.conf settings to customize for another distro.

- permissive global network policy template
- calico eBPF dataplane with wireguard and DSR
- deploy rsyslog.conf
- add wazuh repo and register wazuh agent

#### nsure tha your wazuh manager/rsyslog  server IP is used in the rsyslog.conf and agent registration:

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

To only register the waazuh agent:

```
ansible-playbook -u root -i hosts.inventory build-reef.yml --tags wazuh
```

To only install misc packages:

```
ansible-playbook -u root -i hosts.inventory build-reef.yml --tags packages
```
