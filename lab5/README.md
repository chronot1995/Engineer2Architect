# LAB5

### Table-of-Contents

- [LAB5](#lab5)
    - [Table-of-Contents](#table-of-contents)
    - [Lab5 Goals](#lab5-goals)
    - [Description](#description)
    - [Questions](#questions)
    - [Network-Diagram](#network-diagram)
    - [Network-Outline](#network-outline)
    - [Lab-Setup-and-Configuration](#lab-setup-and-configuration)
      - [Devconatiner-Installation](#devconatiner-installation)
    - [Ansible-Debugging](#ansible-debugging)
    - [Validate-Configuration](#validate-configuration)
    - [ANSWERS TO THE QUESTIONS](#answers-to-the-questions)

### Lab5 Goals

1. Verify OSPF connection between campus-new-datacenter-01 and campus-new-datacenter-02
2. Disable the link between campus-new-datacenter-01 and campus-new-datacenter-02

### Description

We have migrated the network from a legacy setup and design to a redundant, Layer 3 design. We have removed the spanned VLANs and greatly reduced the "blast radius" of a network issue affecting all of the nodes. First, congratulations for making it this far and walking through a network migration.

This lab will demonstrate what happens in an Layer 3 design when there is a cut between two routers. We will break the connection between the two data centers and watch how the network converges and what that means for traffic between the two servers.

### Questions

Here is the routing table on campus-new-datacenter-01, with all connections intact:

```
table

```

1. What does the routing table on campus-new-datacenter-01 look like after we disable the link with campus-new-datacenter-02?

2. Is server01 able to reach server02 in this configuration?

3. Are server01 and server02 using the same gateway in an active / active design?

4. What does an active / active design in a network look like? What type of technologies are used in that design?

### Network-Diagram

[Lab5 Network Diagram](https://github.com/chronot1995/Engineer2Architect/lab5/images/lab5.png)

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

| Router Name              | OOB Management IP | Type         |
| ------------------------ | ----------------- | ------------ |
| isp-edge-01              | 192.168.121.10    | FRR          |
| campus-edge-01           | 192.168.121.11    | Arista cEOS  |
| campus-new-dorms-01      | 192.168.121.12    | Arista cEOS  |
| campus-new-law-01        | 192.168.121.13    | Arista cEOS  |
| campus-new-datacenter-01 | 192.168.121.14    | Arista cEOS  |
| campus-new-datacenter-02 | 192.168.121.15    | Arista cEOS  |
| campus-new-library-01    | 192.168.121.16    | Arista cEOS  |
| nagios                   | 192.168.121.17    | Alpine Linux |
| server01                 | 192.168.121.18    | Alpine Linux |

| Router Name              | Loopback IP    | Type        |
| ------------------------ | -------------- | ----------- |
| campus-edge-01           | 10.137.255.129 | Arista cEOS |
| campus-new-dorms-01      | 10.137.255.130 | Arista cEOS |
| campus-new-law-01    | 10.137.255.131 | Arista cEOS |
| campus-new-datacenter-02 | 10.137.255.132 | Arista cEOS |
| campus-new-datacenter-01 | 10.137.255.133 | Arista cEOS |
| campus-new-library-01    | 10.137.255.134 | Arista cEOS |

| Router #1                | PTP-Link #1 | PTP-Link #2 | Router #2                |
| ------------------------ | ----------- | ----------- | ------------------------ |
| isp-edge-01              | eth1        | eth1        | campus-edge-01           |
| campus-edge-01           | eth2        | eth2        | campus-new-dorms-01      |
| campus-new-dorms-01      | eth3        | eth3        | campus-new-law-01        |
| campus-new-law-01        | eth4        | eth4        | campus-new-datacenter-02 |
| campus-new-datacenter-02 | eth5        | eth5        | campus-new-datacenter-01 |
| campus-new-datacenter-01 | eth6        | eth6        | campus-new-library-01 |
| campus-new-library-01    | eth7        | eth7        | campus-new-dorms-01 |

| Router #1           | PTP-Link IP #1   | PTP-Link IP #2   | Router #2            |
| ------------------- | ---------------- | ---------------- | -------------------- |
| isp-edge-01         | 192.0.2.1/30     | 192.0.2.2/30     | campus-edge-01       |
| campus-new-edge-01      | 10.137.255.1/30  | 10.137.255.2/30  | campus-new-dorms-01  |
| campus-new-dorms-01 | 10.137.255.5/30  | 10.137.255.6/30  | campus-new-law-01      |
| campus-new-law-01 | 10.137.255.9/30  | 10.137.255.10/30 | campus-new-datacenter-02    |
| campus-new-datacenter-02     | 10.137.255.13/30 | 10.137.255.14/30 | campus-new-datacenter-01 |
| campus-new-datacenter-01     | 10.137.255.17/30 | 10.137.255.18/30 | campus-new-library-01 |
| campus-new-library-01     | 10.137.255.21/30 | 10.137.255.22/30 | campus-new-dorms-01 |

PTP = Point-to-Point interface

| BGP Router     | PTP-Link IP #1 | ASN Number |
| -------------- | -------------- | ---------- |
| isp-edge-01    | 192.0.2.1/30   | 65000      |
| campus-edge-01 | 192.0.2.2/30   | 65464      |

| Server Name | Server Interface | Router Name              | Router Interface | Server IP     | Type         |
| ----------- | ---------------- | ------------------------ | ---------------- | ------------- | ------------ |
| nagios      | eth1             | campus-new-datacenter-01 | eth11            | 10.137.121.17 | Alpine Linux |
| server01    | eth1             | campus-new-datacenter-01 | eth14            | 10.137.11.18  | Alpine Linux |
| server02    | eth1             | campus-new-datacenter-02 | eth14            | 10.137.22.19  | Alpine Linux |

### Lab-Setup-and-Configuration

1. Download and install the correct version of [Docker](https://www.docker.com/products/docker-desktop/) for your machine.

Note: this may also work with colima, bit I haven't tried it, as of yet.

2. If you are doing the devcontainer approach, you will need to install a couple of packages by running the following commands:

```
./launch-lab.sh
```

3. Load the Containerlab topology with the following command:

```
clab deploy -t lab5-topology.yaml
```

4. Next, run the Ansible playbook to configure the lab:

```
ansible-playbook playbook-lab5.yaml
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
clab destroy -t lab5-topology.yaml
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

4. If "no" to question #6, what may be a way to enable this feature with in the network?
