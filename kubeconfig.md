# Kubeconfig 

```sh
➜  ls -lh .kube
total 12K
drwxr-x--- 4 root root 4.0K Mar 23 20:19 cache
-rw------- 1 root root 5.6K Mar 23 20:19 config
```

config
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS-REDACTEDklDQVRFLS0tLS0K
    server: https://controlplane:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJT-REDACTED-yQVM1eE9KazVscE5aUUc2bUo2MlcyVC9TbUlLCnVqK3kxUnRiN1BGZFJGdWVQQVF2cEtBRm0yR3ljS3pvUlJwMXJzd1ZvSzk1VFI1T2J3YU5RWGxLcUZhcjM3MEwKK0gyOGFsRmRnWlowaEMwY05QVTkyRVFyZC9NdHovczk4ZXZQQmk0L3RrUkx4UCtkMGN1MDZuTlJuM3M2Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    client-key-data: LS0tLS1CRU-REDACTED-FHL1pxZGEvZTdnSGVxWHVLODAKVXVreGo5d2tyOEMwMWFicmoyYXgvREJKY3psdWVHTFZBQytGdnk5OVlKaGRuU1UvT0NZSDVDZjBMdVNJY1RMVQp0OFZYQkZ4aWU4ZFhwT2tCNnJQNDhLOUwyeVFKRlBIdDFxZktuaWw3MHpMTnViVXNnZ0pZRUE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```

Lets see another config file

```sh
 ➜  ls -lh my-kube-config 
-rw-r--r-- 1 root root 1.5K Mar 23 20:28 my-kube-config
```


```yaml
apiVersion: v1
kind: Config

clusters:
- name: production
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: development
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: kubernetes-on-aws
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: test-cluster-1
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

contexts:
- name: test-user@development
  context:
    cluster: development
    user: test-user

- name: aws-user@kubernetes-on-aws
  context:
    cluster: kubernetes-on-aws
    user: aws-user

- name: test-user@production
  context:
    cluster: production
    user: test-user

- name: research
  context:
    cluster: test-cluster-1
    user: dev-user

users:
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key

current-context: test-user@development
preferences: {}
```

## Using different context 

```sh
➜  kubectl config --kubeconfig=/root/my-kube-config get-contexts
CURRENT   NAME                         CLUSTER             AUTHINFO    NAMESPACE
          aws-user@kubernetes-on-aws   kubernetes-on-aws   aws-user    
          research                     test-cluster-1      dev-user    
*         test-user@development        development         test-user   
          test-user@production         production          test-user 
```

Lets configure and use the context `research`

```sh
➜  kubectl config --kubeconfig=/root/my-kube-config use-context research
Switched to context "research".
```

```sh
➜  kubectl config --kubeconfig=/root/my-kube-config get-contexts
CURRENT   NAME                         CLUSTER             AUTHINFO    NAMESPACE
          aws-user@kubernetes-on-aws   kubernetes-on-aws   aws-user    
*         research                     test-cluster-1      dev-user    
          test-user@development        development         test-user   
          test-user@production         production          test-user   
```

```sh
➜  kubectl config --kubeconfig=/root/my-kube-config current-context
research
```

## Configuring KUBECONFIG variable

```sh
 ➜  vi .bashrc 
 ```

 add the following line

 ```sh
 export KUBECONFIG=/root/my-kube-config
 ```

 save the .bashrc file and exit
 then run 

 ```sh
 ➜  source .bashrc
 ```

 ```sh
 ➜  echo $KUBECONFIG 
/root/my-kube-config
```

## Fixing configuration file

```sh
➜  kubectl get pods
error: unable to read client-cert /etc/kubernetes/pki/users/dev-user/developer-user.crt for dev-user due to open /etc/kubernetes/pki/users/dev-user/developer-user.crt: no such file or directory
```

Checking the correct files 

```sh
✖ ls -lh /etc/kubernetes/pki/users/dev-user/
total 12K
-rw-r--r-- 1 root root 1.1K Mar 23 20:28 dev-user.crt
-rw-r--r-- 1 root root  924 Mar 23 20:28 dev-user.csr
-rw------- 1 root root 1.7K Mar 23 20:28 dev-user.key
```

We need to fix the configuration file name

```sh
➜  vi my-kube-config 
```

change the file from 
```sh
/etc/kubernetes/pki/users/dev-user/developer-user.crt
```

to 
```sh
/etc/kubernetes/pki/users/dev-user/dev-user.crt
```

