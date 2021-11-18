# cloud-batch-helm

## helm 安装 redis

## 

```
mkdir ~/redis
cd ~/redis
helm repo add bitnami https://charts.bitnami.com/bitnami
helm pull bitnami/redis
```

创建 values.yaml

参考： https://github.com/bitnami/charts/tree/master/bitnami/redis

```
master:
  nodeSelector:
    kubernetes.io/hostname: "master1.solarfs.k8s"
  tolerations:
    - effect: NoSchedule
      operator: Exists

replica:
  replicaCount: 1
  nodeSelector:
    kubernetes.io/hostname: "master1.solarfs.k8s"
  tolerations:
    - effect: NoSchedule
      operator: Exists
```

部署 redis

```
helm upgrade --install redis redis-15.3.2.tgz -n redis-system --create-namespace
```

## 安装 cert-manager

参考：[线下安装cert-manager](https://github.com/ss75710541/operator-env/blob/main/cert-manager/%E7%BA%BF%E4%B8%8B%E5%AE%89%E8%A3%85cert-manager.md)

## 安装 cloud-batch

```
mkdir ~/cloud-batch
cd cloud-batch
```

下载 helm chart cloud-batch-0.1.1.tgz

创建 values.yaml

```
# Default values for cloud-batch.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

logLevel: info
# Server jwt secret, Installation Please modify
serverJwtSecret: "123456"
serverListenPort: "5140"
cloudpodsBaseUrl: "https://172.26.117.99/"
cloudpodsUsername: "admin"
# cloudpods base64 encode password, please modify
cloudpodsPassword: "xxxxxxxxxx"
redis:
  host: "redis-master.redis-system.svc"
  # redis password, please modify
  password: "xxxxxx"

image:
  repository: quay.io/ss75710541/cloud-batch
  pullPolicy: Always
  tag: "v0.1.1"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: ""
  annotations:
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
  hosts:
    - host: cloud-batch.apps117104.hisun.k8s
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: cloud-batch-tls
      hosts:
        - cloud-batch.apps117104.hisun.k8s

resources:
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi

nodeSelector:
  kubernetes.io/hostname: "master1.solarfs.k8s"

tolerations:
  - effect: NoSchedule
    operator: Exists
```

执行安装 cloud-batch

```
helm upgrade --install cloud-batch cloud-batch-0.1.1.tgz -f values.yaml -n cloud-batch --create-namespace
```
