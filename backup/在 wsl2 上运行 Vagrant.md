---
date: 2021-11-14T23:52:32+08:00
title: "在 WSL2 上运行 Vagrant"
description: "Run Vagrant on Wsl2"
author: Berny Linville
tags: []
categories: []
externalLink: ""
series: []
---

这篇文章记录了如何在 WSL 2 上安装使用 Vagrant，并且使用 VirtualBox 做为 provider。

## 需求

* For x64 systems: **Version 1903** or higher, with **Build 18362** or higher.
* For ARM64 systems: **Version 2004** or higher, with **Build 19041** or higher.
* [VirtualBox](https://link.zhihu.com/?target=https%3A//www.virtualbox.org/wiki/Downloads)

## 安装 WSL 2

参考官方文档安装 WSL 2，本文不提供说明 [Install WSL on Windows 10​](https://docs.microsoft.com/en-us/windows/wsl/install-win10)

```powershell
PS C:\Users> wsl --list --verbose
  NAME      STATE           VERSION
* Ubuntu    Running         2

```

## 安装 VirtualBox

[VirtualBox downloads](https://www.virtualbox.org/wiki/Downloads)

## 安装 Vagrant

```bash
# 在 ubuntu 中运行：
# https://www.vagrantup.com/downloads
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install vagrant

```

装完之后需要添加本地环境变量

```bash
# https://www.vagrantup.com/docs/other/wsl
# 如果使用的不是 zsh 而是 bash，就添加到 bashrc
echo 'export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"' >> ~/.zshrc
echo 'export PATH="$PATH:/mnt/c/Program Files/Oracle/VirtualBox"' >> ~/.zshrc

# 重新加载 zshrc
source ~/.zshrc

```

## 启动 Vagrant

由于 Windows 权限问题，Vagrant 目录只能放在 Windows 目录下

```bash
cd /mnt/c/Users/name/
mkdir vagrant-demo
cd vagrant-demo

# 下载 box
vagrant box add hashicorp/bionic64

# 初始化 `Vagrantfile`
vagrant init hashicorp/bionic64

```

这时候如果直接 `vagrant up` 启动会报错，连接虚拟机超时，是因为默认转发 ssh 端口到 127.0.0.1:2222 ，而在 WSL 2 里面是无法访问到 Windows 主机 localhost。可以安装 [virtualbox_WSL2](https://github.com/Karandash8/virtualbox_WSL2) 插件解决。

```bash
vagrant plugin install virtualbox_WSL2

```

装完之后启动 Vagrant

```bash
vagrant up

Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'hashicorp/bionic64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'hashicorp/bionic64' version '1.0.282' is up to date...
==> default: Setting the name of the VM: vagrant-demo_default_1628143115787_58322
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 172.17.96.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Connection reset. Retrying...
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default:
    default: Guest Additions Version: 6.0.10
    default: VirtualBox Version: 6.1
==> default: Mounting shared folders...
    default: /vagrant => /mnt/c/Users/name/vagrant-demo

```

默认账户密码都是 vagrant

```bash
vagrant ssh
vagrant@172.17.96.1's password:
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-58-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Aug  5 06:29:26 UTC 2021

  System load:  0.01              Processes:           89
  Usage of /:   2.5% of 61.80GB   Users logged in:     0
  Memory usage: 13%               IP address for eth0: 10.0.2.15
  Swap usage:   0%

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

0 packages can be updated.
0 updates are security updates.


vagrant@vagrant:~$

```

接下来就是结合 ansible 去做测试啦~

## 待解决

* centos/7 启动时会出现挂载文件夹出错

```text
There was an error when attempting to rsync a synced folder.
Please inspect the error message below for more info.

Host path: /mnt/c/Users/name/vagrant-demo/
Guest path: /vagrant
Command: "rsync" "--verbose" "--archive" "--delete" "-z" "--copy-links" "--no-owner" "--no-group" "--rsync-path" "sudo rsync" "-e" "ssh -p 2222 -o LogLevel=FATAL   -o ControlMaster=auto -o ControlPath=/tmp/vagrant-rsync-20210805-19044-ojcjma -o ControlPersist=10m  -o IdentitiesOnly=yes -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i '/mnt/c/Users/name/vagrant-demo/.vagrant/machines/default/virtualbox/private_key'" "--exclude" ".vagrant/" "/mnt/c/Users/name/vagrant-demo/" "vagrant@172.17.96.1:/vagrant"
Error: vagrant@172.17.96.1: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
rsync: connection unexpectedly closed (0 bytes received so far) [sender]
rsync error: unexplained error (code 255) at io.c(235) [sender=3.1.3]

```

* 使用 Docker 做为 provider

* `vagrant ssh` 没有自动调用密钥，而是需要输入密码

## 参考链接

[how-to-run-vagrant-on-wsl-2](https://blog.thenets.org/how-to-run-vagrant-on-wsl-2/)

[Connection Refused in Vagrant using WSL 2​](https://stackoverflow.com/questions/65001570/connection-refused-in-vagrant-using-wsl-2)