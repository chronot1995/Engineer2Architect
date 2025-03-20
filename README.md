# Engineer2Architect

### Summary

This project came to fruition out of a discussion I was having with a buddy who is breaking into networking. One of his questions was how does one grow in this field from a Network Engineer into a Network Architect. This project, the presentation, and the overall design are based on my real-world experience during my first Network Architect job.

This is the initial design of this project toma

### Side Notes

This is the repo used for the KC NUG Keynote Project

This repository will be using the Docker outside of Docker method described here:

### Overall Design

This is the first pass of this project. I used to use Nvidia Air and Vagrant for this type of work before the pandemic. This is my first foray into a complex containerlab configuration.

https://containerlab.dev/macos/#docker-outside-of-docker-dood

and here:

https://youtu.be/Xue1pLiO0qQ?t=603

Note:

docker pull quay.io/frrouting/frr:10.1.3

Installing the image

https://containerlab.dev/manual/kinds/ceos/#arista-ceos

docker import cEOSarm-lab-4.33.1-EFT3.tar.xz ceos:4.33.1-EFT3

This overall work was based on my past work at Nvidia and Cumulus Networks. I wasn't "vibe coding" this by anymeans, and I did ask Claude and CoPilot on the best way to refine the Ansible and Python in this project and it was super helpful.

### Miscellaneous and Tools:

Containerlab - https://containerlab.dev

The containerlab Discord server is a great community where you can talk and discuss with people who are very passionate about this project

https://containerlab.dev/community/#discord

Claude AI - Sonnet 3.7 http://www.claude.ai
Github Copilot https://github.com/features/copilot
VSCode https://code.visualstudio.com

https://www.packetswitch.co.uk/running-containerlab-in-macos-cisco-iol-ceos-2/

https://github.com/srl-labs/containerlab/blob/main/docs/manual/kinds/ceos.md

https://arista-netdevops-community.github.io/avd-cEOS-Lab/quickStart/