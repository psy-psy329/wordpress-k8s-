# wordpress-k8s

将 WordPress + MySQL 从 Docker Compose 迁移到 Kubernetes 的实践项目。

## 原项目

- [wordpress-docker-ansible-deploy](https://github.com/psy-psy329/wordpress-docker-ansible-deploy)
- 基于 Ansible + Docker Compose 的单机部署方案

## 迁移目标

- 从单机 Docker 部署迁移到 Kubernetes 集群
- 通过 Kubernetes Service 实现 WordPress 与 MySQL 的服务发现
- 使用 PersistentVolumeClaim 持久化 MySQL 数据
- 使用 Deployment 管理应用实例并提供故障自愈能力
- 使用 Ingress 为 WordPress 提供统一的 HTTP 访问入口
- 使用 Secret 管理 MySQL 配置，避免将真实密码提交到仓库

## 项目架构

```text
Client
  |
  v
NGINX Ingress (blog.local)
  |
  v
WordPress Service
  |
  v
WordPress Deployment
  |
  v
MySQL Service
  |
  v
MySQL Deployment ---- MySQL PVC
```

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
`-- wordpress/
    |-- wordpress-deploy.yaml
    |-- wordpress-svc.yaml
    `-- wordpress-ingress.yaml
```

> `mysql/mysql-secret.yaml` 保存真实密码，已通过 `.gitignore` 排除，不会提交到仓库。

## 前置条件

- 已准备可用的 Kubernetes 集群
- 已安装并配置 `kubectl`
- 集群中已安装 NGINX Ingress Controller
- 集群提供可用的默认 StorageClass，用于动态创建 MySQL 持久卷

可以通过以下命令确认 Ingress Controller 是否已就绪：

```bash
kubectl get pods -n ingress-nginx
```

## 部署步骤

### 1. 创建 MySQL Secret

复制 Secret 模板：

```bash
cp mysql/mysql-secret.example.yaml mysql/mysql-secret.yaml
```

Windows PowerShell：

```powershell
Copy-Item mysql/mysql-secret.example.yaml mysql/mysql-secret.yaml
```

编辑 `mysql/mysql-secret.yaml`，将密码占位符替换为实际值，并确保数据库名称、用户名和密码与 WordPress Deployment 中的数据库配置一致。

### 2. 部署 MySQL

```bash
kubectl apply -f mysql/mysql-secret.yaml
kubectl apply -f mysql/mysql-pvc.yaml
kubectl apply -f mysql/mysql-deployment.yaml
kubectl apply -f mysql/mysql-svc.yaml
```

### 3. 部署 WordPress

```bash
kubectl apply -f wordpress/wordpress-deploy.yaml
kubectl apply -f wordpress/wordpress-svc.yaml
```

### 4. 创建 Ingress

```bash
kubectl apply -f wordpress/wordpress-ingress.yaml
```

当前 Ingress 使用域名 `blog.local`。在本地测试时，需要将 Ingress Controller 的访问地址映射到该域名。

先查看 Ingress 地址：

```bash
kubectl get ingress wordpress-ingress
```

然后在 hosts 文件中添加映射：

```text
<INGRESS_IP> blog.local
```

- Linux/macOS：`/etc/hosts`
- Windows：`C:\Windows\System32\drivers\etc\hosts`

完成后访问：<http://blog.local>

## 验证部署

```bash
kubectl get deployments,pods,svc,pvc,ingress
```

也可以分别检查 WordPress 和 MySQL 的运行日志：

```bash
kubectl logs deployment/wordpress-deploy
kubectl logs deployment/mysql-deploy
```

## Kubernetes 能力对应

| 目标 | Kubernetes 资源 |
| --- | --- |
| WordPress 和 MySQL 编排 | Deployment |
| 集群内服务发现 | Service |
| MySQL 数据持久化 | PersistentVolumeClaim |
| 故障后自动重建 Pod | Deployment Controller |
| HTTP 域名访问 | Ingress |
| MySQL 敏感配置 | Secret |
