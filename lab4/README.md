# LAB4

### Table-of-Contents

- [LAB4](#lab4)
  - [Table-of-Contents](#table-of-contents)
  - [Lab4 Goals](#lab4-goals)
  - [Description](#description)
  - [Questions](#questions)
  - [Network-Diagram](#network-diagram)
  - [Network-Outline](#network-outline)
  - [Lab-Setup-and-Configuration](#lab-setup-and-configuration)
    - [Devconatiner-Installation](#devconatiner-installation)
  - [Ansible-Debugging](#ansible-debugging)
  - [Validate-Configuration](#validate-configuration)
  - [ANSWERS TO THE QUESTIONS](#answers-to-the-questions)

### Lab4 Goals

1. Create an OSPF connection between the "legacy network" and the "new network"
2. Move the OSPF connection from campus-dorms-01 to campus-new-dorms-01
3. Enable BGP on campus-edge-01 with Arista cEOS
4. Enable BGP on the isp-edge-01 with Free Range Routing (FRR)
5. Answer questions regarding this topology
6. Understand the new output in the Nagios monitoring server, compared to Lab 2

### Description

Let's take a step back on what we have accomplished so far.

In lab1 the major design feature was the spanned VLAN that created a large instability. This design also prevented other vendors from cleaning fitting into the design.

We rectified this in lab2. We moved the underlying topology to a routed network using OSPF. OSPF is a well known protocols supported on every major platform.

In lab3 we enabled dynamic routing / BGP at the edge of the network.

And now in lab4 the steps we have taken, especially in lab 2, allow us to add a whole new network to the topology and interconnect them with ease.

The goal of this lab is to interconnect the new and old networks. This is important because once a network is using standard protocols, it allows you as the architect to deploy a new vendor

### Questions

Here is the routing table from Lab3:

```
Gateway of last resort:
 O E2     0.0.0.0/0 [110/1]
           via 10.137.255.1, Ethernet1

 O        10.137.121.0/24 [110/20]
           via 10.137.255.14, Ethernet4
 C        10.137.255.0/30
           directly connected, Ethernet1
 C        10.137.255.4/30
           directly connected, Ethernet2
 C        10.137.255.8/30
           directly connected, Ethernet3
 C        10.137.255.12/30
           directly connected, Ethernet4
 O        10.137.255.129/32 [110/20]
           via 10.137.255.1, Ethernet1
 C        10.137.255.130/32
           directly connected, Loopback0
 O        10.137.255.131/32 [110/20]
           via 10.137.255.6, Ethernet2
 O        10.137.255.132/32 [110/20]
           via 10.137.255.10, Ethernet3
 O        10.137.255.133/32 [110/20]
           via 10.137.255.14, Ethernet4
```

1. Logging into campus-new-dorms-01, what is the new path to the reach the legacy campus-datacenter-01?

2. Does the egress traffic off the campus still go through the campus-dorms-01?

3. How does the legacy campus-datacenter-01 receive the default route?

4. The Nagios server output is different than in lab3. In this lab, the configuration is correctly configured with reachability to all hosts, why?

### Network-Diagram

[Lab4 Network Diagram](https://github.com/chronot1995/Engineer2Architect/lab4/images/lab4.png)

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

| Router Name          | OOB Management IP | Type         |
| -------------------- | ----------------- | ------------ |
| isp-edge-01          | 192.168.121.10    | FRR          |
| campus-edge-01       | 192.168.121.11    | Arista cEOS  |
| campus-dorms-01      | 192.168.121.12    | Arista cEOS  |
| campus-new-dorms-01  | 192.168.121.13    | Arista cEOS  |
| campus-new-law-01    | 192.168.121.14    | Arista cEOS  |
| campus-datacenter-01 | 192.168.121.15    | Arista cEOS  |
| nagios               | 192.168.121.16    | Alpine Linux |

| Router Name          | Loopback IP    | Type        |
| -------------------- | -------------- | ----------- |
| campus-edge-01       | 10.137.255.129 | Arista cEOS |
| campus-dorms-01      | 10.137.255.130 | Arista cEOS |
| campus-law-01        | 10.137.255.132 | Arista cEOS |
| campus-datacenter-01 | 10.137.255.133 | Arista cEOS |
| campus-new-dorms-01  | 10.137.255.134 | Arista cEOS |
| campus-new-law-01    | 10.137.255.135 | Arista cEOS |

| Router #1           | PTP-Link #1 | PTP-Link #2 | Router #2            |
| ------------------- | ----------- | ----------- | -------------------- |
| isp-edge-01         | eth1        | eth1        | campus-edge-01       |
| campus-edge-01      | eth2        | eth2        | campus-new-dorms-01  |
| campus-new-dorms-01 | eth3        | eth3        | campus-dorms-01      |
| campus-dorms-01     | eth4        | eth4        | campus-datacenter-01 |
| campus-new-dorms-01 | eth5        | eth5        | campus-new-law-01    |

| Router #1           | PTP-Link IP #1   | PTP-Link IP #2   | Router #2            |
| ------------------- | ---------------- | ---------------- | -------------------- |
| isp-edge-01         | 192.0.2.1/30     | 192.0.2.2/30     | campus-edge-01       |
| campus-edge-01      | 10.137.255.1/30  | 10.137.255.2/30  | campus-new-dorms-01  |
| campus-new-dorms-01 | 10.137.255.5/30  | 10.137.255.6/30  | campus-dorms-01      |
| campus-new-dorms-01 | 10.137.255.9/30  | 10.137.255.10/30 | campus-new-law-01    |
| campus-dorms-01     | 10.137.255.13/30 | 10.137.255.14/30 | campus-datacenter-01 |

PTP = Point-to-Point interface

| BGP Router     | PTP-Link IP #1 | ASN Number |
| -------------- | -------------- | ---------- |
| isp-edge-01    | 192.0.2.1/30   | 65000      |
| campus-edge-01 | 192.0.2.2/30   | 65464      |

| Server Name | Server Interface | Router Name          | Router Interface | Server IP     | Type         |
| ----------- | ---------------- | -------------------- | ---------------- | ------------- | ------------ |
| nagios      | eth1             | campus-datacenter-01 | eth11            | 10.137.121.17 | Alpine Linux |

### Lab-Setup-and-Configuration

1. Download and install the correct version of [Docker](https://www.docker.com/products/docker-desktop/) for your machine.

Note: this may also work with colima, bit I haven't tried it, as of yet.

2. If you are doing the devcontainer approach, you will need to install a couple of packages by running the following commands:

```
./launch-lab.sh
```

3. Load the Containerlab topology with the following command:

```
clab deploy -t lab4-topology.yaml
```

4. Next, run the Ansible playbook to configure the lab:

```
ansible-playbook playbook-lab4.yaml
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
clab destroy -t lab4-topology.yaml
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

1. In the previous labs, the campus-dorms-01 was the center of the network topology. Now, it is a spoke off of the new network.

2. No, in this lab we changed the network egress to go through the campus-new-dorms-01 router, rather than the legacy campus-dorms-01 router.

3. The default route is still learned and propagated through OSPF. From the campus-datacenter-01 perspective, the next hop for the default traffic is still coming from the campus-dorms-01 router. We did not make any changes to the underlying topology on the legacy side. As in question 2, when traffic reaches the campus-dorms-01 router, it now will take the connection to the new network and egress from there.

4. Linux has an underlying route table, similar to a switch or a router. In this lab, the management network is only for the 192.168.121.0/24 network. The default route for all other traffic is now the leg connected to the campus-datacenter-01 network. This configuration is one way to have a Linux server with legs in multiple networks. In lab3, the default route was in the management network and had no way to reach the loopbacks of the routers.
