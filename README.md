# EthicalTree Kubernetes

[![EthicalTree Project](https://img.shields.io/badge/site-EthicalTree-blue.svg)](https://ethicaltree.com)

This is where we keep our helm charts and production kubernetes configs

## Architecture Diagram

![Architecture Diagram](/ET_Infrastructure.svg?raw=true&sanitize=true)

## Labelling Nodes

It is best to attach storage volumes to separate nodes. Deployments will be deployed on the node with their respective storage labels

```
kubectl label nodes {db_node} dbstorage=true
kubectl label nodes {blog_node} blogstorage=true
kubectl label nodes {monitoring_node} monitorstorage=true
```

## SSL Certificates

We use Let's Encrypt for our SSL Certificates. In order to update certificates, do the following:

```
docker run --rm -it -v "$PWD/letsencrypt":/etc/letsencrypt ubuntu:16.04 bash
apt update && apt install wget
wget https://dl.eff.org/certbot-auto
chmod +x ./certbot-auto
certbot-auto certonly --agree-tos --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory -d "*.ethicaltree.com" -d "ethicaltree.com"

base64 ./letsencrypt/live/ethicaltree.com/fullchain.pem
base64 ./letsencrypt/live/ethicaltree.com/privkey.pem

# replace existing values with ones from two prev commands
kubectl edit secret ethicaltree-wildcard-tls
```

## Docker Registry Setup

```
kubectl create secret docker-registry docker-cloud --docker-server=https://index.docker.io/v1/ --docker-username={username} --docker-password={api_key} --docker-email={email}
```

## LoadBalancer

- Create a new DigitalOcean node
- Install HAProxy
- Copy `./loadbalancer/haproxy.cfg` -> `/etc/haproxy/haproxy.cfg` on node
- Replace `{ip_address}` for each kubernetes worker node in the cluster


## MySql

```
# Create Persistent Volume Claim (This will automatically provision a volume if it doesn't exist)
kubectl create -f db/pvc.yaml

# Install MySql
helm install --name ethicaltree-db -f db/values.yaml stable/mysql --set myssqlUser={user} --set mysqlPassword={password}

# Load Backup
kubectl port-forward ethicaltree-db-mysql-{pod-id} 3306:3306
mysql -u {user} -p{password} < backup.sql
```

## Redis

```
# Install Redis
helm install --name ethicaltree-redis -f redis/values.yaml stable/redis
```

## Nginx Ingress

```
# Install Nginx
helm install --name nginx-ingress --namespace nginx-ingress -f ingress/values.yaml stable/nginx-ingress
```

## EthicalTree API

```
# Config
kubectl create secret generic ethicaltree-api
kubectl edit secret ethicaltree-api

# Install
helm install --name ethicaltree-api ./api
helm install --name ethicaltree-sidekiq ./api -f sidekiq/values.yaml

# Push new image
bin/deploy_api
```

Make sure you set all the appropriate secrets

## EthicalTree Web

```
# Install
helm install --name ethicaltree-web ./web

# Push new image
bin/deploy_web
```

## EthicalTree Blog
```
# Create Persistent Volume Claim (volume will be created if it doesn't exist)
kubectl create -f blog/pvc.yaml

// The DB is shared with production ethicaltree.
// Restore a backup there before installing the blog

# Install
helm install --name ethicaltree-blog -f blog/values.yaml stable/wordpress
```

#### Test locally

```
cd blog
docker-compose up

## Restore DB
mysql -u root -h 127.0.0.1 -P3309 -p bitnami_wordpress < {et_backup}

## Copy Assets
cp -rf {backup_data} wordpress_data/
```

## Monitoring (Prometheus + Grafana)


```
helm install --name prometheus --namespace monitoring -f monitoring/prometheus.yaml stable/prometheus
helm install --name grafana --namespace monitoring -f monitoring/grafana.yaml stable/grafana
```


## Command Center

In order to test and maintain each system here, it's handy to have a pod preloaded with tools and volumes needed to help configure other systems.

```
kubectl create -f commandcenter/deployment.yaml
```


