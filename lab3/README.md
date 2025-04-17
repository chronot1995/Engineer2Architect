# LAB3

### Table-of-Contents

- [LAB3](#lab3)
  - [Table-of-Contents](#table-of-contents)
  - [Lab3 Goals](#lab3-goals)
  - [Questions](#questions)
  - [Network-Diagram](#network-diagram)
  - [Network-Outline](#network-outline)
  - [Lab-Setup-and-Configuration](#lab-setup-and-configuration)
    - [Devconatiner-Installation](#devconatiner-installation)
  - [Ansible-Debugging](#ansible-debugging)
  - [Validate-Configuration](#validate-configuration)
  - [ANSWERS TO THE QUESTIONS](#answers-to-the-questions)

### Lab3 Goals

1. Enable BGP on campus-edge-01 with Arista cEOS
2. Enable BGP on the isp-edge-01 with Free Range Routing (FRR)
3. Answer questions regarding this topology
4. Understand the new output in the Nagios monitoring server, compared to Lab 2

==

### Questions

1. Ping from the FRR router

To access this router, you will run the following command

```
docker exec -it isp-edge-01 /bin/bash
```

Load the FRR shell with the following command:

```
vtysh
```

Run the following commands:

```
ping 10.137.255.130
```

This will attempt to ping the campus-dorms-01 router.

Question:

Did this work?
What may have caused this to not work?

Exit it out of this device and ssh into the campus-edge-01 router:

```
ssh admin@192.168.121.11
```

Let's determine if we are advertising the routes to the isp-edge-01:

Show Advertising BGP routes:

```
show bgp neighbors 192.0.2.1 ipv4 unicast advertised-routes
```

You should seen an output similar to the following:

```
10.137.0.0/16
```

We have been deploying /32s and /30s in our OSPF network. How are we able to advertise this large network on the Arista EOS side? Can you find that in the router and BGP configuration?

Now, let's take a look at what we are receiving from the isp-edge-01 peer:
Showing Received BGP routes:

```
show bgp neighbors 192.0.2.1 ipv4 unicast received-routes
```

This looks correct on the Arista side. Let's go back to the isp-edge-01 device:

```
docker exec -it isp-edge-01 /bin/bash
```

Load the FRR shell with the following command:

```
vtysh
```

Let's run the same commands that we ran on the Arista side, but now using the FRR syntax.

Hmm... something here does not look correct

Here, you'll be able to do Arista / Cisco level commands to show the status of the network connectivity

### Network-Diagram

[Lab3 Network Diagram](https://github.com/chronot1995/Engineer2Architect/lab3/images/lab3.png)

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

| BGP Router     | PTP-Link IP #1 | ASN Number |
| -------------- | -------------- | ---------- |
| isp-edge-01    | 192.0.2.1/30   | 65000      |
| campus-edge-01 | 192.0.2.2/30   | 65464      |

### Lab-Setup-and-Configuration

1. Download and install the correct version of [Docker](https://www.docker.com/products/docker-desktop/) for your machine.

Note: this may also work with colima, bit I haven't tried it, as of yet.

2. If you are doing the devcontainer approach, you will need to install a couple of packages by running the following commands:

```
./launch-lab.sh
```

3. Load the Containerlab topology with the following command:

```
clab deploy -t lab3-topology.yaml
```

4. Next, run the Ansible playbook to configure the lab:

```
ansible-playbook playbook-lab3.yaml
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
clab destroy -t lab3-topology.yaml
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

1. Why are we not able to receive a BGP route on the FRR device?

Inside of the FRR configuration you will see the following configuration:

```
 address-family ipv4 unicast
  neighbor 192.0.2.2 default-originate
  neighbor 192.0.2.2 route-map isp-accept-routes in
```

When we look at the route-map, we will see the following:

```
route-map isp-accepted-routes permit 1
```

There's the problem! There was a mispelling on the route map.

To fix this, run the following with the FRR VTYSH shell:

```
conf t
router bgp 65000
address-family ipv4 unicast
no neighbor 192.0.2.2 route-map isp-accept-routes in
neighbor 192.0.2.2 route-map isp-accepted-routes in
exit
write mem
```

Re-run the received routes command to see the difference.

You now should be able to successfully ping the dorms router:

```
ping 192.168.255.130
```
