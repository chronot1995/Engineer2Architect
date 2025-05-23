# LAB2

### Table-of-Contents

- [LAB2](#lab2)
  - [Table-of-Contents](#table-of-contents)
  - [Lab2 Goals](#lab2-goals)
  - [Questions](#questions)
  - [Network-Diagram](#network-diagram)
  - [Network-Outline](#network-outline)
  - [Lab-Setup-and-Configuration](#lab-setup-and-configuration)
    - [Devconatiner-Installation](#devconatiner-installation)
  - [Ansible-Debugging](#ansible-debugging)
  - [Validate-Configuration](#validate-configuration)
  - [ANSWERS TO THE QUESTIONS](#answers-to-the-questions)

### Lab2 Goals

1. Enable Dynamic Routing on the Lab1 topology
2. Answer questions regarding this topology
3. Understand the output in the Nagios monitoring server

### Questions

i. Network Analysis

SSH into the "campus" devices

```
Username: admin
Password: admin
```

ii. Display the routing table look like on the campus devices?

```
show route
```

iii. Are the campus devices dynamically routing? How are they connected together?

iv. Display the OSPF adjacencies using the following command:

```
show ip ospf neighbor
```

v. Are there now any spanned VLANs in this design? What would happen if there was a loop on the Library router? What would be the fallout? How would that compare to the previous design?

vi. What may be a limitation of this design? Are you able to reach all of the devices in the topology?

vii. Log into the Nagios monitoring system by clicking this [link](https://127.0.0.1:8080)

```
Username: nagiosadmin
Password: nagios
```

Click the "Hosts" on the left-hand side to see what is currently being monitored. Are the currently configured OSPF loopbacks pingable? Why not?

### Network-Diagram

[Lab2 Network Diagram](https://github.com/chronot1995/Engineer2Architect/lab2/images/lab2.png)

### Network-Outline

For this site, we are using the following subnet:

```
10.137.0.0/16
```

We are going to be taking the last /24 subnet and using that for our Point-to-Point routed links and hte loopbacks:

```
10.137.255.0/24
```

We are then going to split this in half in the following manner:

```
Point-to-Points: 10.137.255.0/25
Loopbacks: 10.137.255.128/25
```

| Router Name          | OOB Management IP | Type        |
| -------------------- | ----------------- | ----------- |
| isp-edge-01          | 192.168.121.10    | FRR         |
| campus-edge-01       | 192.168.121.11    | Arista cEOS |
| campus-dorms-01      | 192.168.121.12    | Arista cEOS |
| campus-library-01    | 192.168.121.13    | Arista cEOS |
| campus-law-01        | 192.168.121.14    | Arista cEOS |
| campus-datacenter-01 | 192.168.121.15    | Arista cEOS |
| nagios               | 192.168.121.16    | Linux       |

| Router Name          | Loopback IP    | Type        |
| -------------------- | -------------- | ----------- |
| campus-edge-01       | 10.137.255.129 | Arista cEOS |
| campus-dorms-01      | 10.137.255.130 | Arista cEOS |
| campus-library-01    | 10.137.255.131 | Arista cEOS |
| campus-law-01        | 10.137.255.132 | Arista cEOS |
| campus-datacenter-01 | 10.137.255.133 | Arista cEOS |

| Router #1       | PTP-Link #1 | PTP-Link #2 | Router #2            |
| --------------- | ----------- | ----------- | -------------------- |
| isp-edge-01     | eth1        | eth1        | campus-edge-01       |
| campus-edge-01  | eth2        | eth1        | campus-dorms-01      |
| campus-dorms-01 | eth2        | eth2        | campus-library-01    |
| campus-dorms-01 | eth3        | eth3        | campus-law-01        |
| campus-dorms-01 | eth4        | eth4        | campus-datacenter-01 |

| Router #1       | PTP-Link IP #1   | PTP-Link IP #2   | Router #2            |
| --------------- | ---------------- | ---------------- | -------------------- |
| isp-edge-01     | 192.0.2.1/30     | 192.0.2.2/30     | campus-edge-01       |
| campus-edge-01  | 10.137.255.1/30  | 10.137.255.2/30  | campus-dorms-01      |
| campus-dorms-01 | 10.137.255.5/30  | 10.137.255.6/30  | campus-library-01    |
| campus-dorms-01 | 10.137.255.9/30  | 10.137.255.10/30 | campus-law-01        |
| campus-dorms-01 | 10.137.255.13/30 | 10.137.255.14/30 | campus-datacenter-01 |

PTP = Point-to-Point interface

### Lab-Setup-and-Configuration

1. Download and install the correct version of [Docker](https://www.docker.com/products/docker-desktop/) for your machine.

Note: this may also work with colima, bit I haven't tried it, as of yet.

2. If you are doing the devcontainer approach, you will need to install a couple of packages by running the following commands:

```
./launch-lab.sh
```

3. Load the Containerlab topology with the following command:

```
clab deploy -t lab2-topology.yaml
```

4. Next, run the Ansible playbook to configure the lab:

```
ansible-playbook playbook-lab2.yaml
```

You will get an output similar to the following:

```
PLAY RECAP *************************************************************************************************************************
campus-datacenter-01       : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
campus-dorms-01            : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
campus-edge-01             : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
campus-law-01              : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
campus-library-01          : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

5. To destroy the lab, run the following command:

```
clab destroy -t lab2-topology.yaml
```

#### Devconatiner-Installation

Note 1: I still need to setup a brand new version of macOS and run through the steps below.

Note 2: these next sections are the same for each of the labs in this repo

1. Follow this [link](https://containerlab.dev/macos/#docker-outside-of-docker-dood) to the install instructions for your platform

and a Youtube video [here](https://youtu.be/Xue1pLiO0qQ?t=603)

2. Install the FRR Docker image on your host.

```
docker pull quay.io/frrouting/frr:10.1.3
```

3. [Installing the Arista cEOS image](https://containerlab.dev/manual/kinds/ceos/#arista-ceos)

```
docker import cEOSarm-lab-4.33.1-EFT3.tar.xz ceos:4.33.1-EFT3
```

This work is based on my past work at Nvidia and Cumulus Networks. I wasn't "vibe coding" this by any means, and I did ask Claude and CoPilot on the best way to refine the Ansible and Python in this project and it was very helpful.

### Ansible-Debugging

1. Test if Ansible is installed correctly by running the following command:

```
ansible arista_switches -m ansible.builtin.ping
```

### Validate-Configuration

1. SSH into one of the campus switches and run the following command:

```
ping vrf MGMT ip 192.168.121.14
```

You will be able to ping the OOB / Management IP addresses of all of the devices in the lab.

2. SSH into the in-band loopback port

```
ping 10.137.255.131
```

You will now be able to ping the In-band Management IP / Loopback addresses of all of the OSPF-enabled devices in the lab.

### ANSWERS TO THE QUESTIONS

To work around the Nagios monitoring issue, one will need to adjust the routing table on the Nagios server itself.

Log into the Nagios server using this command:

```
docker exec -it clab-lab2-nagios /bin/bash
```

On the box, run the following command:

```
ip route show
```

To change the routing table, run the following:

```
ip route del default via 192.168.121.1 dev eth0
ip route add default via 10.137.121.1 dev eth1
```

The Nagios server will now be able to ping the loopbacks of all the campus switches.
