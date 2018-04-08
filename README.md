# EthicalTree Kubernetes

[![EthicalTree Project](https://img.shields.io/badge/site-EthicalTree-blue.svg)](https://ethicaltree.com)

This is where we keep our helm charts and production kubernetes configs

## Architecture Diagram

![Architecture Diagram](/ET_Infrastructure.svg?raw=true&sanitize=true)

## MySql


```
# Install MySql
helm install --name ethicaltree-db -f ethicaltree-db/values.yaml stable/mysql --set myssqlUser={user} --set mysqlPassword={password}

# Load Backup
kubectl port-forward ethicaltree-db-mysql-{pod-id} 3306:3306
mysql -u {user} -p{password} < backup.sql
```

## SSL (Let's Encrypt)

This chart will automatically get SSL certificates before the old ones expire

```
helm install --name kube-lego --namespace=kube-system stable/kube-lego --set config.LEGO_EMAIL={admin_email} --set config.LEGO_URL=https://acme-v01.api.letsencrypt.org/directory
```

## EthicalTree API


```
# Install
helm install --name ethicaltree-api ./ethicaltree-api

# Push new image
helm upgrade --recreate-pods ethicaltree-api ./ethicaltree-api
```

Make sure you set all the appropriate secrets

## EthicalTree Web

```
# Install
helm install --name ethicaltree-web ./ethicaltree-web

# Push new image
helm upgrade --recreate-pods ethicaltree-web ./ethicaltree-web
```




