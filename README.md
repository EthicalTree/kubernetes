# EthicalTree Kubernetes

[![EthicalTree Project](https://img.shields.io/badge/site-EthicalTree-blue.svg)](https://ethicaltree.com)

This is where we keep our helm charts and production kubernetes configs

## Architecture Diagram

![Architecture Diagram](/ET_Infrastructure.svg?raw=true&sanitize=true)

## SSL Certificates

We use Let's Encrypt for our SSL Certificates. In order to update certificates, do the following:

```
docker run --rm -it -v "$PWD/letsencrypt":/etc/letsencrypt ubuntu:16.04 bash
apt update
wget https://dl.eff.org/certbot-auto
chmod +x ./certbot-auto
./certbot-auto renew
```

## Docker Registry Setup

```
kubectl create secret docker-registry docker-cloud --docker-server=https://index.docker.io/v1/ --docker-username={username} --docker-password={api_key} --docker-email={email}
```


## MySql

```
# Install MySql
helm install --name ethicaltree-db -f ethicaltree-db/values.yaml stable/mysql --set myssqlUser={user} --set mysqlPassword={password}

# Load Backup
kubectl port-forward ethicaltree-db-mysql-{pod-id} 3306:3306
mysql -u {user} -p{password} < backup.sql
```

## Redis

```
# Install Redis
helm install --name ethicaltree-redis stable/redis --set usePassword=true
```

## Nginx Ingress

```
# Install Nginx
helm install --name nginx-ingress --namespace nginx-ingress stable/nginx-ingress --set rbac.create=true
```

## EthicalTree API

```
# Config
kubectl create secret generic ethicaltree-api
kubectl edit secret ethicaltree-api

# Install
helm install --name ethicaltree-api ./ethicaltree-api
helm install --name ethicaltree-sidekiq ./ethicaltree-api -f ethicaltree-sidekiq/values.yaml

# Push new image
bin/deploy_api
```

Make sure you set all the appropriate secrets

## EthicalTree Web

```
# Install
helm install --name ethicaltree-web ./ethicaltree-web

# Push new image
bin/deploy_web
```




