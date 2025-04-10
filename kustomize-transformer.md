# Kustomize Transformers


```sh
➜  tree k8s
k8s
├── db
│   ├── db-config.yaml
│   ├── kustomization.yaml
│   ├── NoSql
│   │   ├── db-depl.yaml
│   │   ├── db-service.yaml
│   │   └── kustomization.yaml
│   └── Sql
│       ├── db-depl.yaml
│       ├── db-service.yaml
│       └── kustomization.yaml
├── kustomization.yaml
├── monitoring
│   ├── grafana-depl.yaml
│   ├── grafana-service.yaml
│   └── kustomization.yaml
└── nginx
    ├── kustomization.yaml
    ├── nginx-depl.yaml
    └── nginx-service.yaml

5 directories, 15 files
```

k8s/kustomatization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - db/
  - monitoring/
  - nginx/

commonLabels:
  sandbox: dev
```

k8s/db/kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - NoSql/
  - Sql/
  - db-config.yaml

namePrefix: data-
```


k8s/monitoring/kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - grafana-depl.yaml
  - grafana-service.yaml

namespace: logging
```

We can use examples like this:

```yaml
commonAnnotations:
  owner: bob@gmail.com

images:
  - name: nginx
    newTag: "1.23"

namespace: logging

namePrefix: data-

images:
  - name: postgres
    newName: mysql
```

