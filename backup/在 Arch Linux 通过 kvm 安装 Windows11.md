---
date: 2022-03-20T16:23:39+08:00
title: "Arch kvm 安装 Windows11"
description: "Arch Install windows11 on kvm"
slug: ""
author: Berny Linville
tags: []
categories: []
externalLink: ""
series: []
---

## 安装 kvm、qemu

```sh
❯ sudo pacman -Syu qemu libvirt virt-manager qemu-arch-extra swtpm edk2-ovmf

```

> swtpm 让虚拟机添加 TPM2

## 配置网卡桥接

```sh
❯ nmcli connection add type bridge ifname br0 stp no
❯ nmcli connection add type bridge-slave ifname enp34s0 master br0
❯ nmcli con down "Wired connection 1"
❯ nmcli connection up bridge-br0

```

```sh
❯ nmcli con show
NAME                    UUID                                  TYPE      DEVICE          
bridge-br0              e2dee4be-17b7-4059-8229-6a165158d519  bridge    br0             
br-446b33a13d24         4041fd8b-d776-4372-98fa-ecf4767cffac  bridge    br-446b33a13d24 
docker0                 fa037cc6-806d-428f-a1da-3b8c4197d4d4  bridge    docker0         
virbr0                  b689bd20-a8b3-4fc9-9c46-8bdc63e63f0b  bridge    virbr0          
vnet8                   f5056cc0-d47a-4cb1-abbd-1fdb487629c2  tun       vnet8           
bridge-slave-enp34s0    280629df-9489-49b2-b668-8f7533acf20d  ethernet  enp34s0         
bridge-slave-enp30s0    cf4563b3-1593-4fd8-9037-eb625ae2a3f3  ethernet  --              
bridge-slave-enp30s0-1  5aa64cf9-0d70-4c33-9a32-7d61d7bd8615  ethernet  --              
Wired connection 1      150d0277-48ca-3ea9-8441-6a90d0f742f4  ethernet  -- 

```

### 让 kvm 使用 br0 网卡

```xml
<network>
  <name>br0</name>
  <forward mode="bridge"/>
  <bridge name="br0" />
</network>

```

```sh
❯ virsh net-define br0.xml
❯ virsh net-start br0
❯ virsh net-autostart br0

```

```sh
❯ virsh net-list --all
 Name   State    Autostart   Persistent
-----------------------------------------
 br0    active   yes         yes

```