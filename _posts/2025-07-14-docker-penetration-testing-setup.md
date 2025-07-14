---
title: Setting up a Penetration Testing Lab with docker
description: This guide will show you how to set up a kali docker container that can be easily used from your host machine.
author: Hnin
date: 2025-07-14 11:33:00 +0800
categories: [Pentration Testing, Setup]
tags: [docker, ubuntu, kali, thinkpad]
---

# Background
Just a few days ago, when cleaning up the storage room in my house, I found a ThinkPad X230 lying around. As I procrastinate studying for CPTS, I thought it would be a great idea to turn this old laptop into my designated penetration testing laptop. 

![ThinkPad X230 Image](../assets/img/thinkpad-x230.png)
<p style="text-align: center;">Image Credit: CNET/Sarah Tew</p>

# The Dilemma 
I took the X230 apart, cleaned the fan, re-grease the cpu, upgraded the ram and ssd and replaced the screen. Then I thought about what OS I was going to boot on it. 

Definitely not Windows -- it would probably be too slow on the thinkpad and the OS alone would take up too much storage. 

Perhaps some Linux distro then. Kali? While its good for its wide range of penetration testing tools, it is unstable and insecure. Parrot Security OS? I actually haven't used it before and I wanted something that was familiar so I'll pass this time. 

Good Ol' Ubuntu it was then. But installing all the security tools would be such a chore and some tools aren't even in ubuntu's repositories! I still need Kali installed somehow. I tried to install Kali on a VM but the thinkpad starting heating up and generating so much noise. Instead, I decided to use Kali in a Docker Container in Ubuntu!

# Motivation
Following certain youtube tutorials online, I did manage to get a working kali docker container but I found that it was unoptimised for penetration testing. Thats why I am writing a guide here today just in case I forget in the future. 

# VM vs Containers 

## Running Kali on a Virtual Machine 
Here Kali runs in its own kernel, where the host and itself are isolated. However, that means its heavier on resources such as CPU, RAM and disk and slower startup as it requires booting the entire operating system. 

## Running Kali in a Docker Container
Kali shares the host OS kernel so its lightweight, uses less memory and storage. It launches in seconds and there is minimal overhead. 

> 🖥️ The biggest disadvantage to using the docker container is the loss of GUI. It is possible to configure a GUI on docker but then might as well use a VM. 

> ❌ The other disadvantage is that it is less isolated from the host compared to VM, so a compromised container can affect the host. You won't want to be running malware on this container. 

# The Setup 

1. Pull the Official Kali Linux Docker Image
```bash
docker pull kalilinux/kali-rolling
```

2. Run the container 
```bash
docker run -it kalilinux/kali-rolling
```
A root shell will spawn 

3. Install the Kali Tools (It might take around 30 minutes)
```bash
apt update && apt -y install kali-linux-headless
```

4. Make a folder called host in the `/` directory to allow file exchange from host to container
```bash
mkdir /host
```

5. CTRL+C to exit the container

6. Create a docker image of the container (might take a while too)
```bash
docker commit <container_id> <new_image_name>
```

7. Then we can shut down our current container and remove it 
```bash
docker rm <container_id> && docker image rm kalilinux/kali-rolling
```

8. Make a folder on the host to allow file exchange form host to container
```bash
docker run -it -v /mnt/kali:/host --net=host --cap-add=NET_RAW --cap-add=NET_ADMIN <new_image_name> /bin/bash
```

Flags Explained:

`-v /mnt/kali:/host` -> sync the /mnt/kali folder on the host with the /host folder. Use case: Say you are generating a msfvenom payload and need to upload it to a website, copy at payload to /host and then from host we can go to /mnt/kali and then get the payload and upload it on the website. 

`--net=host` -> that means container is using the host's network stack. So we don't need to do any network settings to be able to use the HTB VPN. We don't need to worry about having to port forward when listening for reverse shell etc

`--cap-add=NET_RAW --cap-add=NET_ADMIN` -> allow container to send and receive raw IP packets directly. Use Case: ping, nmap etc.

> 💡 Container compromise can lead to host compromise as the kernel is shared. Don't keep random ports open all the time and stop the container when you are not using it.

## Starting and Attaching to a container
When you CTRL+C to exit from the container, the container will be stopped. 

To restart it: 
```bash
docker start <container_id>
```

To attach to it so we can see the shell again: 
```bash
docker attach <container_id>
```

## Removing the container 
Say you want to remove the container to change some start up settings and you want to save the files and the new tools you installed in the docker, you have to commit an image of it. 
```bash
docker commit <container_id> <new_image_name>
```
then start up the container with the new image

remember to remove your old images because they do take up quiet a bit of space

# How I use it
Because the Kali installation is without any GUI, tools with GUI are installed on Ubuntu. Its not so bad because there are only so few like: 

- BurpSuite
- WireShark 
- BinaryNinja
- BloodHound

Then for the other non-GUI tools, I use it on the Docker container. 

# Whats next? 
In the same shelf I found the thinkpad, I also discovered a Dell OptiPlex 3040M. I might upgrade certain parts and use this mini-PC to run either a malware lab or enterprise environment (with an AD, MailServer, EDR, SIEM, etc...) But I should really get back to studying for CPTS so maybe after that, I'll start this new project. 