# wordpress-k8s

将 WordPress + MySQL 从 Docker Compose 迁移到 Kubernetes 的实践项目。

## 原项目

- [wordpress-docker-ansible-deploy](https://github.com/psy-psy329/wordpress-docker-ansible-deploy)
- 基于 Ansible + Docker Compose 的单机部署方案

## 项目内容

本项目保留了原有的 Kubernetes YAML 部署方式，并新增了 Helm Chart，方便使用参数化配置部署 WordPress 和 MySQL。

- 使用 Deployment 管理 WordPress 和 MySQL
- 使用 Service 实现集群内服务发现
- 使用 PVC 持久化 MySQL 数据
- 使用 Secret 配置 MySQL 环境变量
- 使用 Ingress 通过 `blog.local` 访问 WordPress
- 使用 Helm Chart 统一管理 Kubernetes 资源

## 目录结构

```text
wordpress-k8s/
|-- README.md
|-- .gitignore
|-- mysql/
|   |-- mysql-secret.example.yaml
|   |-- mysql-secret.yaml
|   |-- mysql-pvc.yaml
|   |-- mysql-deployment.yaml
|   `-- mysql-svc.yaml
|-- wordpress/
|   |-- wordpress-deploy.yaml
|   |-- wordpress-svc.yaml
|   `-- wordpress-ingress.yaml
`-- wordpress-chart/
    |-- Chart.yaml
    |-- values.yaml
    |-- .helmignore
    `-- templates/
        |-- mysql-secret.yaml
        |-- mysql-pvc.yaml
        |-- mysql-deploy.yaml
        |-- mysql-svc.yaml
        |-- wordpress-deploy.yaml
        |-- wordpress-svc.yaml
        `-- wordpress-ingress.yaml
```

## 环境要求

- Kubernetes 集群
- `kubectl`
- Helm 3
- NGINX Ingress Controller
- 可用的默认 StorageClass

## 方式一：使用 Kubernetes YAML

### 1. 创建 MySQL Secret

Linux/macOS：

```bash
cp mysql/mysql-secret.example.yaml mysql/mysql-secret.yaml
```

Windows PowerShell：

```powershell
Copy-Item mysql/mysql-secret.example.yaml mysql/mysql-secret.yaml
```

编辑 `mysql/mysql-secret.yaml`，填写实际的数据库密码。该文件已被 `.gitignore` 排除，不会提交到仓库。

### 2. 部署 MySQL

```bash
kubectl apply -f mysql/mysql-secret.yaml
kubectl apply -f mysql/mysql-pvc.yaml
kubectl apply -f mysql/mysql-deployment.yaml
kubectl apply -f mysql/mysql-svc.yaml
```

### 3. 部署 WordPress 和 Ingress

```bash
kubectl apply -f wordpress/wordpress-deploy.yaml
kubectl apply -f wordpress/wordpress-svc.yaml
kubectl apply -f wordpress/wordpress-ingress.yaml
```

## 方式二：使用 Helm Chart

查看 Chart 信息：

```bash
helm show chart ./wordpress-chart
```

安装 Chart：

```bash
helm install wordpress ./wordpress-chart
```

如果之前已经安装过，可以使用升级命令：

```bash
helm upgrade wordpress ./wordpress-chart
```

查看 Helm 发布状态：

```bash
helm list
kubectl get deployments,pods,svc,pvc,ingress
```

卸载 Helm 发布：

```bash
helm uninstall wordpress
```

Chart 的主要配置位于 `wordpress-chart/values.yaml`，可以根据需要修改：

- MySQL 镜像和副本数
- MySQL 存储容量
- MySQL 数据库、用户和密码
- WordPress 镜像和副本数
- WordPress Service 端口
- Ingress 域名和 IngressClass

当前 `values.yaml` 中的密码是测试配置，实际使用时建议替换为自己的密码，不要将真实密码提交到公共仓库。

## Ingress 访问

当前 Ingress 使用域名：

```text
blog.local
```

查看 Ingress 地址：

```bash
kubectl get ingress wordpress-ingress
```

然后将 Ingress Controller 的访问地址写入 hosts 文件：

```text
<INGRESS_IP> blog.local
```

- Linux/macOS：`/etc/hosts`
- Windows：`C:\Windows\System32\drivers\etc\hosts`

配置完成后访问：

```text
http://blog.local
```

## 验证部署

```bash
kubectl get pods
kubectl get svc
kubectl get pvc
kubectl get ingress
```

查看日志：

```bash
kubectl logs deployment/wordpress-deploy
kubectl logs deployment/mysql-deploy
```

## 项目定位

这是一个用于学习 Kubernetes 和 Helm 的实践项目，重点展示从 Docker Compose 到 Kubernetes 的迁移过程，以及原生 YAML 和 Helm Chart 两种部署方式。

当前项目使用单副本 WordPress 和 MySQL，后续可以继续扩展健康检查、WordPress 文件持久化、TLS、备份恢复和 CI/CD。
