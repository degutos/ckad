# Customize


Lets consider the following files:

```sh
 ➜  tree k8s/
k8s/
├── db
│   ├── db-config.yaml
│   ├── db-depl.yaml
│   └── db-service.yaml
├── kustomization.yaml
├── message-broker
│   ├── rabbitmq-config.yaml
│   ├── rabbitmq-depl.yaml
│   └── rabbitmq-service.yaml
└── nginx
    ├── nginx-depl.yaml
    └── nginx-service.yaml

3 directories, 9 files
```

Lets create a kustomize yaml file for the above files

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# kubernetes resources to be managed by kustomize
resources:
  - db/db-config.yaml
  - db/db-depl.yaml
  - db/db-service.yaml
  - message-broker/rabbitmq-config.yaml
  - message-broker/rabbitmq-depl.yaml
  - message-broker/rabbitmq-service.yaml
  - nginx/nginx-depl.yaml
  - nginx/nginx-service.yaml
```

Lets create our kustomize file and apply to kubernetes

```sh
➜  kubectl apply -k k8s/
configmap/db-credentials created
configmap/redis-credentials created
service/db-service created
service/nginx-service created
service/rabbit-cluster-ip-service created
deployment.apps/db-deployment created
deployment.apps/nginx-deployment created
deployment.apps/rabbitmq-deployment created
```

Lets check the resources created

```sh
➜  kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
db-deployment-856558f969-95v6q         1/1     Running   0          40s
nginx-deployment-6fd6985867-cbn9b      1/1     Running   0          40s
nginx-deployment-6fd6985867-xps2c      1/1     Running   0          40s
nginx-deployment-6fd6985867-zmxpg      1/1     Running   0          40s
rabbitmq-deployment-56cbdbfd4c-2bpwj   1/1     Running   0          40s
```

Lets check the services created

```sh
➜  kubectl get services
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
db-service                  NodePort    10.43.70.64     <none>        27017:30666/TCP   7m25s
kubernetes                  ClusterIP   10.43.0.1       <none>        443/TCP           21m
nginx-service               NodePort    10.43.9.240     <none>        80:31973/TCP      7m25s
rabbit-cluster-ip-service   ClusterIP   10.43.152.140   <none>        5672/TCP          7m25s
```

## Creating a kustomize file for each directory

k8s/db/kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# kubernetes resources to be managed by kustomize
resources:
  - db-config.yaml
  - db-depl.yaml
  - db-service.yaml
```

k8s/message-broker/
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# kubernetes resources to be managed by kustomize
resources:
  - rabbitmq-config.yaml
  - rabbitmq-depl.yaml
  - rabbitmq-service.yaml
```

k8s/nginx/kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# kubernetes resources to be managed by kustomize
resources:
  - nginx-depl.yaml
  - nginx-service.yaml
```

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# kubernetes resources to be managed by kustomize
resources:
  - db/
  - message-broker
  - nginx/
```


