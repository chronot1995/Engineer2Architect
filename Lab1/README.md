# LAB1

### Table-of-Contents

- [Table of Contents](#Table-of-Contents)
  - [Lab Setup and Configuration](#Lab-Setup-and-Configuration)
  - [Ansible Debugging](#Ansible-Debugging)
  - [Validate Configuration](#Validate-Configuration)

### Network-Diagram

I found the diagrams from 15 years ago and built the lab diagrams using the same objects and fonts.

[Lab1 Network Diagram](https://github.com/chronot1995/Engineer2Architect/lab1/images/lab1-03262005.png)

### Network-Outline

| Router Name          | Management IP  | Type        |
| -------------------- | -------------- | ----------- |
| isp-edge-01          | 192.168.121.10 | FRR         |
| campus-edge-01       | 192.168.121.11 | Arista cEOS |
| campus-dorms-01      | 192.168.121.12 | Arista cEOS |
| campus-library-01    | 192.168.121.13 | Arista cEOS |
| campus-law-01        | 192.168.121.14 | Arista cEOS |
| campus-datacenter-01 | 192.168.121.15 | Arista cEOS |
| nagios               | 192.168.121.16 | Linux       |

| Router #1       | PTP-Link #1 | PTP-Link #2 | Router #2            |
| --------------- | ----------- | ----------- | -------------------- |
| isp-edge-01     | eth1        | eth1        | campus-edge-01       |
| campus-edge-01  | eth2        | eth1        | campus-dorms-01      |
| campus-dorms-01 | eth2        | eth2        | campus-library-01    |
| campus-dorms-01 | eth3        | eth3        | campus-law-01        |
| campus-dorms-01 | eth4        | eth4        | campus-datacenter-01 |

PTP = Point-to-Point interface

### Lab-Setup-and-Configuration

Note: these next three sections are the same for each of the labs in this repo

This is the first pass of this project. I used to use Nvidia Air and Vagrant for this type of work before the pandemic. This is my first foray into a complex containerlab configuration.

1. Follow this [link](https://containerlab.dev/macos/#docker-outside-of-docker-dood) to the install instructions for your platform

https://containerlab.dev/macos/#docker-outside-of-docker-dood

and here:

https://youtu.be/Xue1pLiO0qQ?t=603

Note:

docker pull quay.io/frrouting/frr:10.1.3

Installing the image

https://containerlab.dev/manual/kinds/ceos/#arista-ceos

docker import cEOSarm-lab-4.33.1-EFT3.tar.xz ceos:4.33.1-EFT3

This overall work was based on my past work at Nvidia and Cumulus Networks. I wasn't "vibe coding" this by anymeans, and I did ask Claude and CoPilot on the best way to refine the Ansible and Python in this project and it was super helpful.

### Ansible-Debugging

`ansible arista_switches -m ansible.builtin.ping`

### Validate-Configuration

`ping vrf MGMT ip 192.168.121.14`
