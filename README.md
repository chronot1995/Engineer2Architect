# Engineer2Architect

### Summary

This project came out of a discussion I had with a friend who is starting his networking career. One of his questions was how does one grow in this field from a Network Engineer into a Network Architect. This project, the presentation, and the overall design are based on my real-world experience during my first Network Architect job.

This is a project in two pieces. First, I will be uploading the Powerpoint and notes that I present as a keynote to the USNUA ((MO)NUG - Kansas City) inaugural KC event on April 10th, 2025.

I will use the majority of that event to talk about the soft skills necessary to go from a network engineer to a network architect.

The second piece of the project is to focus on the technical pieces of that journey. This github repo will contain 5 x labs that highlight the technical skills necessary. The first four labs should be complete by the time of the presentation, with the MPLS-based one a week or so after. Ha, I've had to dust off some knowledge to pull that one off.

Each lab will contain Arista EOS, FRR, and Linux server images in a Containerlab topology using devcontainers. The individual elements will be configured using Ansible within the devcontainer framework. The goal of the labs is to step through the presentation topics on a technical level to reinforce the steps of the journey.

### Table-of-Contents

- [Individual Labs](#Table-of-Contents)
  - [Lab1](https://github.com/chronot1995/Engineer2Architect/blob/main/lab1/README.md)
  - [Lab2](https://github.com/chronot1995/Engineer2Architect/blob/main/lab2/README.md)
  - [Lab3](https://github.com/chronot1995/Engineer2Architect/blob/main/lab3/README.md)
  - [Lab4](https://github.com/chronot1995/Engineer2Architect/blob/main/lab4/README.md)
  - [Lab5](https://github.com/chronot1995/Engineer2Architect/blob/main/lab5/README.md)

The Powerpoint presentation will eventually be found in the "presentation" folder within the repo.

### Overall Design

### Miscellaneous

Here's a list of tools and links that were used to build this project:

1. [Containerlab](https://containerlab.dev)

The [Containerlab Discord server](https://containerlab.dev/community/#discord) is a great community where you can talk with people who are very passionate about this project

2. [VSCode](https://code.visualstudio.com)

3. [Arista Validated Design](https://arista-netdevops-community.github.io/avd-cEOS-Lab/quickStart/)

AI Tools:

4. [Claude AI - Sonnet 3.7](http://www.claude.ai)

5. [Github Copilot](https://github.com/features/copilot)

Helpful blogs:

6. [Running Containerlab in macOS](https://www.packetswitch.co.uk/running-containerlab-in-macos-cisco-iol-ceos-2/)

### Potential Lab Improvements

I am accepting PRs on this repo. Here's a couple that I will aim for in the future:

1. Idempotency improvements. Run a diff analysis on the Arista EOS configuration when running the playbooks a second time.

Optional:

1. Ansible Vault for passwords, rather than the defaults
