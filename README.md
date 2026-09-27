# wordpress-k8s

将 WordPress + MySQL 从 Docker Compose 迁移到 Kubernetes 的实践项目，并使用 Helm 对 Kubernetes 资源进行参数化管理。

## 项目来源

- [wordpress-docker-ansible-deploy](https://github.com/psy-psy329/wordpress-docker-ansible-deploy)
- 原项目基于 Ansible + Docker Compose，面向单机部署
- 本项目进一步迁移到 Kubernetes，并补充 Ingress 和 Helm Chart

## 项目目标

- 从单机 Docker Compose 部署迁移到 Kubernetes 集群
- 使用 Deployment 管理 WordPress 和 MySQL
- 使用 Service 实现集群内服务发现
- 使用 PersistentVolumeClaim 持久化 MySQL 数据
- 使用 Secret 管理数据库凭据
- 使用 Ingress 提供 `blog.local` HTTP 访问入口
- 使用 Helm values 对镜像、副本数、存储大小和域名进行配置

## 架构

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

`mysql/mysql-secret.yaml` 用于保存本地真实密码，已被 `.gitignore` 排除。Helm Chart 中的密码仅为占位值，实际部署时应通过自定义 values 覆盖。

## 前置条件

- 已准备可用的 Kubernetes 集群
- 已安装并配置 `kubectl`
- 已安装 Helm 3
- 集群中已安装 NGINX Ingress Controller
- 集群提供可用的默认 StorageClass

检查工具版本：

```bash
kubectl version --client
helm version
```

## 方式一：使用原生 Kubernetes YAML

### 1. 创建 MySQL Secret

Linux/macOS：

```bash
cp mysql/mysql-secret.example.yaml mysql/mysql-secret.yaml
```

Windows PowerShell：

```powershell
Copy-Item mysql/mysql-secret.example.yaml mysql/mysql-secret.yaml
```

编辑 `mysql/mysql-secret.yaml`，将密码占位符替换为实际值。

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

### 1. 检查 Chart

```bash
helm lint ./wordpress-chart
helm template wordpress ./wordpress-chart
```

### 2. 准备本地配置

复制一份本地 values 文件，并修改数据库密码：

```bash
cp wordpress-chart/values.yaml wordpress-chart/values.local.yaml
```

Windows PowerShell：

```powershell
Copy-Item wordpress-chart/values.yaml wordpress-chart/values.local.yaml
```

编辑 `wordpress-chart/values.local.yaml` 中的以下字段：

```yaml
mysql:
  secret:
    rootPassword: "replace-with-a-strong-root-password"
    password: "replace-with-a-strong-wordpress-password"
```

不要将包含真实密码的 `values.local.yaml` 提交到仓库。

### 3. 安装或升级

```bash
helm upgrade --install wordpress ./wordpress-chart \
  --namespace wordpress \
  --create-namespace \
  -f ./wordpress-chart/values.local.yaml
```

如果只进行临时测试，也可以直接使用默认 values：

```bash
helm upgrade --install wordpress ./wordpress-chart \
  --namespace wordpress \
  --create-namespace
```

### 4. 查看 Helm 部署状态

```bash
helm list -n wordpress
kubectl get deployments,pods,svc,pvc,ingress -n wordpress
```

## Ingress 访问

当前 Ingress 默认配置为：

```yaml
wordpress:
  ingress:
    enabled: true
    host: blog.local
    className: nginx
```

确认 Ingress 地址：

```bash
kubectl get ingress -n wordpress
```

然后将 Ingress Controller 的访问地址写入 hosts 文件：

```text
<INGRESS_IP> blog.local
```

- Linux/macOS：`/etc/hosts`
- Windows：`C:\Windows\System32\drivers\etc\hosts`

完成后访问 <http://blog.local>。

如果暂时不需要 Ingress，可以在 Helm 部署时关闭：

```bash
helm upgrade --install wordpress ./wordpress-chart \
  --namespace wordpress \
  --create-namespace \
  --set wordpress.ingress.enabled=false
```

## 常用验证命令

```bash
kubectl get pods -n wordpress
kubectl get svc -n wordpress
kubectl get pvc -n wordpress
kubectl get ingress -n wordpress
kubectl logs deployment/wordpress-deploy -n wordpress
kubectl logs deployment/mysql-deploy -n wordpress
```

检查 Helm 渲染结果：

```bash
helm template wordpress ./wordpress-chart \
  --namespace wordpress \
  -f ./wordpress-chart/values.local.yaml
```

## Kubernetes 能力对应

| 能力 | Kubernetes / Helm 资源 |
| --- | --- |
| 应用编排 | Deployment |
| 集群内服务发现 | Service |
| MySQL 数据持久化 | PersistentVolumeClaim |
| 故障后自动重建 Pod | Deployment Controller |
| HTTP 域名访问 | Ingress |
| 数据库凭据管理 | Secret |
| 配置参数化和统一安装 | Helm Chart |

## 当前范围

这是一个用于学习和展示的 Kubernetes 迁移项目，当前使用单副本 WordPress 和 MySQL。项目暂未覆盖高可用数据库、TLS、备份恢复、健康检查和 CI/CD，这些可以作为后续扩展方向。
