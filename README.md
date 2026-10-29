# SDN lab for classrooms: Open vSwitch, OpenFlow and a learning switch on one VM

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md) | [Deutsch](README.de.md)

[![check](https://github.com/mitsurutech/lab-sdn-lab/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-sdn-lab/actions/workflows/check.yml)

A ready-made **software-defined networking (SDN) lab** for teaching how a switch forwards traffic, from the [MitsuruTech Learning Platform](https://mitsurutech.co.uk/learning), where schools and colleges give every student their own machines and terminal in the browser. Each student gets an Ubuntu **VM** with **Open vSwitch**, **Mininet** and the **os-ken** OpenFlow controller library, and a switch `br0` with three hosts, `h1`, `h2` and `h3`. The switch starts with an empty flow table, so it forwards nothing. The student writes **OpenFlow flows by hand** with `ovs-ofctl`, then finishes a **learning-switch controller** that learns which port each MAC address is on and installs a flow for it. Nothing is installed on the student's computer, and no network equipment is needed.

## What the student does

The lab writes a `README.md` in the home folder that says what to do and gives none of the answers, and two files to edit, `/etc/sdnlab/flows.txt` and `/etc/sdnlab/switch.py`, both loaded every time the lab starts. The hosts are network namespaces (`h1` is 10.0.0.1, `h2` 10.0.0.2, `h3` 10.0.0.3, with MAC addresses ending 01, 02 and 03) and the switch is in fail-mode secure, so with no flow it drops every packet. In three parts:

1. **Mininet** (not marked): `sudo mn --mac --test pingall`, then a Mininet prompt to look at nodes, links and the flows its controller installs.
2. **Write the flow table by hand** in `flows.txt`: h1 and h2 must reach each other both ways (ARP as well as the echo), a flow that matches h3 must drop it, and no flow may use the action `NORMAL`.
3. **Finish a controller** in `switch.py`: three TODO steps in `packet_in` to learn the source address, choose the port and install a flow, so the next packets of a conversation never reach the controller.

The lab adds an `sdnlab` command (`status`, `flows`, `controller`, `reset`, `shell h1`) to rebuild the lab, start the controller, put both files back as given and open a shell inside a host.

## How it is marked

The lab has a **check**, a script that marks a handed-in VM without the teacher reading it. It runs on a copy of the saved machine after a fresh start and loads the student's two files itself, so a flow typed by hand and never saved does not count.

- **Part 2, no controller:** h1 and h2 must reach each other, h3 must reach nobody and nobody must reach h3, and a flow that matches h3 must drop it and must have been hit by h3's packets. A flow with the action `NORMAL` does not pass.
- **Part 3, the student's controller:** the check starts it, gives h2 a MAC address nobody has seen, so a table written in advance is no use, and pings between every pair. The switch must send unknown packets to the controller, and the controller must have installed a flow for each address pair written as `priority=1,in_port=PORT,dl_src=SOURCE_MAC,dl_dst=DESTINATION_MAC` with the action output to the port it learned, and none of the flows it installed may flood.

The first line of what it prints says what is not right.

What it does not do: it tests with ping and reads the flow table, not other kinds of traffic, and a student with root on their own VM could still tamper with the tools the check relies on. It is a check for a classroom, not a proof.

## What is in the file

`sdn-lab.lab.yaml` is a whole **VM lab definition**, exactly what the platform runs:

| Key | What it is |
| --- | --- |
| `cpus`, `memory`, `disk` | the size of each student's VM: 1 CPU, 1024 MB and 10 GB |
| `open` | the ports that open in the browser: none, the lab is the terminal |
| `setup` | a script run once, as root, the first time each student's VM starts: it installs Open vSwitch, Mininet, os-ken, `iproute2`, `ping` and `tcpdump` from Ubuntu's archive, and writes the `sdnlab` command, a small launcher for the controller, the two files to edit, a systemd unit and the README |
| `check` | the marking script, run as root on a copy of the saved disk after a fresh boot; its exit status is the result and `0` passes |

## Using it on the platform

**To set it as work for a class:**

1. In your module, add a task and choose the **Ubuntu VM lab** card.
2. Press the Software-defined networking sample, or paste the contents of `sdn-lab.lab.yaml`. Using a sample replaces the text, and Undo brings yours back.
3. Publish and try your own copy first. Every student gets a VM of their own, kept between sessions. The first start takes a few minutes while the setup installs the packages.
4. Assign it like any task. When it is handed in, run the check on a student's row in the class list: it starts a copy of their saved machine that is not saved back, and shows whether it passed and what it printed.

A Basics lesson on the platform, SDN Basics, works in this lab. The help manual's [page for this lab](https://mitsurutech.co.uk/help/lab-sdn-lab), its [guide to Ubuntu VM labs](https://mitsurutech.co.uk/help/labs) and its [sample labs page](https://mitsurutech.co.uk/help/sample-labs) cover the rest.

No college account yet? [Your first year is free](https://mitsurutech.co.uk/pricing) for up to 50 students, and [signing up](https://mitsurutech.co.uk/signup) takes an email address.

## Checking the file

On every push, `.github/workflows/check.yml` checks that the file is valid YAML with the keys above and that `setup` and `check` are each inside the platform's 16 KB limit. It does not boot a VM: that is done on the platform's own machines.

## What does not work in a lab

The VM reaches Ubuntu's package archive, PyPI, npm and GitHub, and nothing else. The network between the hosts is virtual and inside the VM, so nothing leaves it. The controller library os-ken depends on eventlet, which is deprecated and prints a notice in the controller's log; that is why the lab teaches `ovs-ofctl` and Mininet first. Mininet's `sudo mn -c` removes every Open vSwitch bridge, `br0` included: run `sudo sdnlab flows` afterwards to build the lab again. The setup and the check were run in a container standing in for the VM, and a booted student VM is worth one try before a class.

## About

Made by [Mitsuru Technologies](https://mitsurutech.co.uk) for the [MitsuruTech Learning Platform](https://mitsurutech.co.uk/learning): Python, Java, Go, Node, SQL and Linux in the browser for schools and colleges, with container labs, VM labs, web apps deployed from GitHub, and quizzes that mark themselves. Try code with no account in the [playground](https://mitsurutech.co.uk/playground).

More labs from the same place: [lab-netlab](https://github.com/mitsurutech/lab-netlab), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea), [lab-sample](https://github.com/mitsurutech/lab-sample), [lab-reverse-lab](https://github.com/mitsurutech/lab-reverse-lab), [lab-formal-lab](https://github.com/mitsurutech/lab-formal-lab), [lab-hdl-lab](https://github.com/mitsurutech/lab-hdl-lab), [lab-k3s-lab](https://github.com/mitsurutech/lab-k3s-lab), [lab-tofu-lab](https://github.com/mitsurutech/lab-tofu-lab), [lab-ansible-roles](https://github.com/mitsurutech/lab-ansible-roles) and [lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics).

## Licence

This work is licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). You may share it and adapt it for learning and teaching, if you credit Mitsuru Technologies Ltd, do not use it commercially, and share anything you make from it under the same licence. The full text is in the `LICENSE` file.

[Open vSwitch](https://www.openvswitch.org), Mininet and os-ken are installed from Ubuntu's archive when a VM starts; nothing of them is distributed here.
