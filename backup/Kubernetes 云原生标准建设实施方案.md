## 选择合适的 Kubernetes 版本

在线上环境中，选择一个合适的 Kubernetes 版本至关重要。Kubernetes 版本号遵循 `x.y.z` 的命名规范，其中 `x` 表示主版本，`y` 表示次版本，`z` 表示补丁版本。为了确保稳定性，建议参考各大云计算厂商（如 Google、AWS、DigitalOcean）的选择，因为他们提供的 Kubernetes 云平台通常比普通生产环境更稳定。

### 版本选择建议

- **避免小版本号低于 5 的版本**：这些版本可能存在漏洞或问题，不适合生产环境。
- **避免接近小版本 10 的版本**：这些版本更新维护较少，升级路径不稳定。

### Kubernetes 发布周期

根据 [Kubernetes 发布时间线](https://github.com/kubernetes/kubernetes/releases)，我们可以总结以下几点：

1. **小版本更新**：每个小版本（`z`）通常每月更新一次，但到 10 以后更新周期会变长。
2. **稳定版本周期**：每个版本从 alpha 到 stable 大约需要三个月，stable 版本后会继续维护约 12 个月。
3. **版本生命周期**：每个 `y` 版本的生命周期大约为 15 个月，前三个月为开发测试阶段，后 12 个月为修复阶段。
4. **当前维护版本**：目前维护的版本包括 1.32.x、1.31.x、1.30.x、1.29.x 以及 alpha 阶段的 1.33.x。

### 主要云服务商支持的 Kubernetes 版本

截至 2024-12-28，主要云服务商支持的 Kubernetes 版本如下：

| 提供商 | 支持的 Kubernetes 版本 | 参考链接 |
| --- | --- | --- |
| GKE | 1.31.3、1.30.6、1.30.5 | [GKE 发布说明](https://cloud.google.com/kubernetes-engine/docs/release-notes) |
| AKS | 1.30.5、1.29.9、1.28.14 | [AKS 发布说明](https://github.com/Azure/AKS/releases) |
| EKS | 1.31.2、1.30.6、1.29.10、1.28.15、1.27.16 | [EKS 用户指南](https://docs.aws.amazon.com/eks/latest/userguide/platform-versions.html) |

## 操作系统选择

### 国产信创操作系统

- openEuler 22.03 LTS
- Kylin Linux Advanced Server V10 (Sword)

### 无信创要求操作系统

- AlmaLinux OS 9
- Debian bookworm
- Ubuntu 22.04 LTS

## 部署工具选择

- [Kubespray](https://github.com/kubernetes-sigs/kubespray)
- [Kubekey](https://github.com/kubesphere/kubekey)

### Kubespray 部署概述

由于国内网络环境的原因，建议优先选择离线部署方式。可以参考 [Kubespray 离线部署指南](https://github.com/kubespray-offline/kubespray-offline)。

### 离线环境准备

```sh
./download-all.sh
```

所有文件将存储在 `./outputs` 目录中。该脚本会调用以下脚本：

- `prepare-pkgs.sh`：设置 Python、Podman 等。
- `prepare-py.sh`：设置 Python 虚拟环境并安装所需的 Python 包。
- `get-kubespray.sh`：下载并解压 Kubespray。
- `pypi-mirror.sh`：下载 PyPI 镜像文件。
- `download-kubespray-files.sh`：下载 Kubespray 离线文件。
- `download-additional-containers.sh`：下载额外的容器镜像。
- `create-repo.sh`：下载 RPM 或 DEB 仓库。
- `copy-target-scripts.sh`：复制目标节点的脚本。

### 离线部署准备

将 `outputs.tar.gz` 上传至 Ansible 服务器，并执行以下脚本：

- `setup-container.sh`：从本地文件安装 containerd。
- `start-nginx.sh`：启动 Nginx 容器。
- `setup-offline.sh`：设置 yum/deb 仓库配置和 PyPI 镜像配置。
- `setup-py.sh`：从本地仓库安装 Python3 和虚拟环境。
- `start-registry.sh`：启动 Docker 私有仓库容器。
- `load-push-images.sh`：加载所有容器镜像到 containerd 并推送到私有仓库。
- `extract-kubespray.sh`：解压 Kubespray 并应用所有补丁。

### 通过 Kubespray 部署集群

确保所有服务器开启 `firewalld` 服务，并将所有 IP 添加到 trusted 域中。使用 Kubespray 部署时，无需手动配置内核和模块参数。

#### 安装所需包

创建并激活虚拟环境：

```sh
$ python3.11 -m venv ~/.venv/3.11
$ source ~/.venv/3.11/bin/activate
$ python --version   # 检查 Python 版本
```

解压 Kubespray 并应用补丁：

```sh
$ ./extract-kubespray.sh
$ cd kubespray-{version}
```

安装 Ansible：

```sh
$ pip install -U pip                # 更新 pip
$ pip install -r requirements.txt   # 安装 Ansible
```

#### 创建 `offline.yml`

将 `offline.yml` 文件复制到你的 inventory 目录下的 `group_vars/all/offline.yml`，并编辑它。需要将 `YOUR_HOST` 替换为你的 registry/nginx 主机 IP。

#### 部署离线仓库配置

使用 Ansible 将离线仓库配置部署到所有目标节点：

```sh
$ cp -r ${outputs_dir}/playbook ${kubespray_dir}
$ cd ${kubespray_dir}
$ ansible-playbook -i ${your_inventory_file} offline-repo.yml
```

#### 运行 Kubespray

运行 Kubespray Ansible playbook：

```sh
$ ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
```

## 高可用说明

1. **云服务器**：使用云负载均衡器，负载 6443 和 2379 端口。
2. **私有云**：使用 haproxy + keepalived，负载 6443 和 2379 端口。

## CRI 和 CNI 选择

- **CRI**：优先选择 containerd。
- **CNI**：优先选择 Cilium（原因：性能优异，支持网络策略）。

## Ingress 选择

- **Nginx Ingress**：如果集群有可用的负载均衡器（如 MetalLB），可以直接使用；否则，使用 NodePort 暴露服务，并通过高可用方案提供对外服务。

## CSI 选择

- **NFS**：如果有 NFS 资源，可以使用 [CSI Driver for NFS](https://github.com/kubernetes-csi/csi-driver-nfs)。
- **自建持久化存储**：
  1. **Rook Ceph**：适用于有 3 台节点专用于 Ceph 存储服务的集群。
  2. **OpenEBS**：参考 [OpenEBS 文档](https://openebs.io/docs/quickstart-guide/installation)，但不适用于离线环境。
  3. **Gluster**：由于已停止维护，不建议使用。

## GitOps 模式

- **离线环境**：不适用于 GitOps 模式，但可以将 YAML 文件存放在 GitLab 仓库中，并手动同步到 Kubernetes 集群。
- **非离线环境**：通过 [ArgoCD Autopilot](https://github.com/argoproj-labs/argocd-autopilot) 实现 GitOps 模式，并使用 ArgoCD 进行发布。

## 其他工具

- [Prometheus Helm Charts](https://github.com/prometheus-community/helm-charts)
- [Grafana Helm Charts](https://github.com/grafana/helm-charts)
- [Harbor 离线镜像仓库](https://goharbor.io/docs/2.12.0/install-config/)
