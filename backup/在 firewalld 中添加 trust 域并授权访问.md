在某些情况下，你可能需要信任特定的 IP 地址，允许其访问服务器上的所有端口。`firewalld` 是一个强大的防火墙管理工具，它允许你通过创建自定义的防火墙域来实现这一目标。本文将详细介绍如何在 `firewalld` 中添加一个 `trust` 域，并信任特定的 IP 地址 `10.5.1.38` 访问所有端口。

## 1. 启动 `firewalld` 服务

首先，确保 `firewalld` 服务已经启动并设置为开机自启。如果尚未启动，可以使用以下命令启动并启用 `firewalld`：

```sh
sudo systemctl start firewalld
sudo systemctl enable firewalld
```

## 2. 创建 `trust` 域

接下来，我们需要创建一个新的防火墙域，命名为 `trust`。使用以下命令创建并重新加载 `firewalld` 配置：

```sh
sudo firewall-cmd --permanent --new-zone=trust
sudo firewall-cmd --reload
```

## 3. 将 IP 地址添加到 `trust` 域

现在，我们将 IP 地址 `10.5.1.38` 添加到 `trust` 域中。使用以下命令：

```sh
sudo firewall-cmd --permanent --zone=trust --add-source=10.5.1.38
```

## 4. 允许 `trust` 域中的所有流量

为了允许 `trust` 域中的所有流量，我们需要将 `trust` 域的目标设置为 `ACCEPT`。使用以下命令：

```sh
sudo firewall-cmd --permanent --zone=trust --set-target=ACCEPT
```

## 5. 重新加载 `firewalld` 以应用更改

完成上述配置后，重新加载 `firewalld` 以应用更改：

```sh
sudo firewall-cmd --reload
```

## 6. 验证配置

最后，验证配置是否正确应用。使用以下命令查看 `trust` 域的配置：

```sh
sudo firewall-cmd --zone=trust --list-all
```

你应该会看到类似以下的输出：

```
trust (active)
  target: ACCEPT
  icmp-block-inversion: no
  interfaces: 
  sources: 10.5.1.38
  services: 
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

## 完整命令

以下是完整的命令列表，供参考：

```sh
sudo firewall-cmd --permanent --new-zone=trust
sudo firewall-cmd --reload
sudo firewall-cmd --permanent --zone=trust --add-source=10.5.1.38
sudo firewall-cmd --permanent --zone=trust --set-target=ACCEPT
sudo firewall-cmd --reload
```

## 注意事项

- 确保你信任的 IP 地址是安全的，避免将不受信任的 IP 地址添加到 `trust` 域。
- 如果你需要撤销对某个 IP 地址的信任，可以使用以下命令将其从 `trust` 域中移除：

  ```sh
  sudo firewall-cmd --permanent --zone=trust --remove-source=10.5.1.38
  sudo firewall-cmd --reload
  ```

- 定期检查 `firewalld` 的配置，确保没有不必要的安全风险。
