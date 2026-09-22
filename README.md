# wordpress-k8s

使用 Kubernetes 部署 WordPress 和 MySQL 的基础配置。

## 目录结构

```text
wordpress-k8s/
├── README.md
├── mysql/
│   ├── mysql-secret.example.yaml
│   ├── mysql-secret.yaml
│   ├── mysql-pvc.yaml
│   ├── mysql-deployment.yaml
│   └── mysql-svc.yaml
├── wordpress/
│   ├── wordpress-deploy.yaml
│   └── wordpress-svc.yaml
└── .gitignore
```

## 使用方式

1. 创建 MySQL Secret：

   ```powershell
   Copy-Item mysql/mysql-secret.example.yaml mysql/mysql-secret.yaml
   ```

   编辑 `mysql/mysql-secret.yaml`，将占位符替换为实际密码。该文件已被 `.gitignore` 排除，不会提交到仓库。

2. 部署 MySQL：

   ```powershell
   kubectl apply -f mysql/mysql-secret.yaml
   kubectl apply -f mysql/mysql-pvc.yaml
   kubectl apply -f mysql/mysql-deployment.yaml
   kubectl apply -f mysql/mysql-svc.yaml
   ```

3. 部署 WordPress：

   ```powershell
   kubectl apply -f wordpress/wordpress-deploy.yaml
   kubectl apply -f wordpress/wordpress-svc.yaml
   ```

查看资源状态：

```powershell
kubectl get pods,svc,pvc
```

## 配置说明

- MySQL 数据通过 `mysql-pvc` 持久化。
- WordPress 通过 Kubernetes Service 访问 MySQL Service `mysql-deploy`。
- `mysql-secret.yaml` 只保留在本地，仓库中提交 `mysql-secret.example.yaml` 作为配置模板。
