# EthicalTree Kubernetes

This is where we keep our helm charts and production kubernetes configs

## MySql


```
helm install --name ethicaltree-db -f ethicaltree-db/values.yaml stable/mysql --set myssqlUser={user} --set mysqlPassword={password}
```

## SSL (Let's Encrypt)

This chart will automatically get SSL certificates before the old ones expire

```
helm install --name kube-lego --namespace=kube-system stable/kube-lego --set config.LEGO_EMAIL={admin_email} --set config.LEGO_URL=https://acme-v01.api.letsencrypt.org/directory
```

## EthicalTree API


```
helm install --name ethicaltree-api ./ethicaltree-api
```

Make sure you set all the appropriate secrets

## EthicalTree Web

```
helm install --name ethicaltree-web ./ethicaltree-web
```




