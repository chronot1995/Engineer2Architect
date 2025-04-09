# LAB1

### Table-of-Contents

- [LAB1](#lab1)
  - [Table-of-Contents](#table-of-contents)
  - [Lab1 Goals](#lab1-goals)
  - [Network-Diagram](#network-diagram)
  - [Network-Outline](#network-outline)
  - [Lab-Setup-and-Configuration](#lab-setup-and-configuration)
    - [Devconatiner-Installation](#devconatiner-installation)
  - [Ansible-Debugging](#ansible-debugging)
  - [Validate-Configuration](#validate-configuration)

### Lab1 Goals

1. Monitoring and Triage

Log into the monitoring system by clicking this [link](https://127.0.0.1:8080)

```
Username: nagiosadmin
Password: nagios
```

Click the "Hosts" on the left-hand side to see what is currently being monitored.

Questions:

i. Does the amount of devices you're seeing in Nagios match the [Network Diagram](#Network-Diagram) and [Network Outline](#Network-Outline) below?

ii. Can you adjust the `nagios.cfg` to reflect the true state of the network?

iii. Rebuild the devcontainers and launch them again after you have made this change?

iv. Are all of the devices green?

2. Network Analysis

SSH into the "campus" devices

```
Username: admin
Password: admin
```

i. Display the routing table look like on the campus devices?

```
show route
```

ii. Are the campus devices doing dynamic routing? How are they connected together?

iii. What may be a limitation of this design?

iv. If VLAN 555 was looped on campus-law-01, what would that do this topology? And why?

### Network-Diagram

I found the diagrams from 15 years ago and built the lab diagrams using the same objects and fonts.

[Lab1 Network Diagram](https://github.com/chronot1995/Engineer2Architect/lab1/images/lab1.png)

### Network-Outline

| Router Name          | OOB Management IP | Type        |
| -------------------- | ----------------- | ----------- |
| isp-edge-01          | 192.168.121.10    | FRR         |
| campus-edge-01       | 192.168.121.11    | Arista cEOS |
| campus-dorms-01      | 192.168.121.12    | Arista cEOS |
| campus-library-01    | 192.168.121.13    | Arista cEOS |
| campus-law-01        | 192.168.121.14    | Arista cEOS |
| campus-datacenter-01 | 192.168.121.15    | Arista cEOS |
| nagios               | 192.168.121.16    | Linux       |

| Router #1       | PTP-Link #1 | PTP-Link #2 | Router #2            |
| --------------- | ----------- | ----------- | -------------------- |
| isp-edge-01     | eth1        | eth1        | campus-edge-01       |
| campus-edge-01  | eth2        | eth1        | campus-dorms-01      |
| campus-dorms-01 | eth2        | eth2        | campus-library-01    |
| campus-dorms-01 | eth3        | eth3        | campus-law-01        |
| campus-dorms-01 | eth4        | eth4        | campus-datacenter-01 |

PTP = Point-to-Point interface

### Lab-Setup-and-Configuration

1. Download and install the correct version of [Docker](https://www.docker.com/products/docker-desktop/) for your machine.

Note: this may also work with colima, bit I haven't tried it, as of yet.

2. If you are doing the devcontainer approach, you will need to install a couple of packages by running the following commands:

```
cd lab1
pip install -r requirements.txt
```

3. Load the Containerlab topology with the following command:

```
clab deploy -t lab1-topology.yaml
```

4. Next, run the Ansible playbook to configure the lab:

```
ansible-playbook playbook-lab1.yaml
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
clab destroy -t lab1-topology.yaml
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
