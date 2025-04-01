# Helm

## Install

Lets check the current OS version

```sh
controlplane ~ ➜  cat /etc/os-release
PRETTY_NAME="Ubuntu 22.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.5 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```

Lets access the Helm doc page

```sh
https://helm.sh/docs/intro/install/#from-apt-debianubuntu
```

copy the snippet

```sh
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

Lets list the helm env variable

```sh
controlplane ~ ➜  helm env
HELM_BIN="helm"
HELM_BURST_LIMIT="100"
HELM_CACHE_HOME="/root/.cache/helm"
HELM_CONFIG_HOME="/root/.config/helm"
HELM_DATA_HOME="/root/.local/share/helm"
HELM_DEBUG="false"
HELM_KUBEAPISERVER=""
HELM_KUBEASGROUPS=""
HELM_KUBEASUSER=""
HELM_KUBECAFILE=""
HELM_KUBECONTEXT=""
HELM_KUBEINSECURE_SKIP_TLS_VERIFY="false"
HELM_KUBETLS_SERVER_NAME=""
HELM_KUBETOKEN=""
HELM_MAX_HISTORY="10"
HELM_NAMESPACE="default"
HELM_PLUGINS="/root/.local/share/helm/plugins"
HELM_QPS="0.00"
HELM_REGISTRY_CONFIG="/root/.config/helm/registry/config.json"
HELM_REPOSITORY_CACHE="/root/.cache/helm/repository"
HELM_REPOSITORY_CONFIG="/root/.config/helm/repositories.yaml"
```

Lets check the current helm version

```sh
controlplane ~ ✖ helm version
version.BuildInfo{Version:"v3.16.4", GitCommit:"7877b45b63f95635153b29a42c0c2f4273ec45ca", GitTreeState:"clean", GoVersion:"go1.22.7"}
```

## Helm concepts

```sh
controlplane ~ ➜  helm search hub wordpress
URL                                                    CHART VERSION   APP VERSION           DESCRIPTION                                       
https://artifacthub.io/packages/helm/wordpress-...     1.0.2           1.0.0                 A Helm chart for deploying Wordpress+Mariadb st...
https://artifacthub.io/packages/helm/kube-wordp...     0.1.0           1.1                   this is my wordpress package                      
https://artifacthub.io/packages/helm/bitnami/wo...     24.1.18         6.7.2                 WordPress is the world's most popular blogging ...
https://artifacthub.io/packages/helm/bitnami-ak...     15.2.13         6.1.0                 WordPress is the world's most popular blogging ...
https://artifacthub.io/packages/helm/shubham-wo...     0.1.0           1.16.0                A Helm chart for Kubernetes                       
https://artifacthub.io/packages/helm/skywordpre...     0.1.0           1.16.0                A Helm chart for WordPress                        
https://artifacthub.io/packages/helm/sikalabs/w...     0.2.0                                 Simple Wordpress                                  
https://artifacthub.io/packages/helm/devops/wor...     0.12.0          1.16.0                Wordpress helm chart                              
https://artifacthub.io/packages/helm/schichtel/...     0.8.0           6.7.2                 A Helm chart for WordPress                        
https://artifacthub.io/packages/helm/sikademo/w...     2020.11.18                            Example Wordpress Chart (with MySQL)              
https://artifacthub.io/packages/helm/camptocamp...     0.6.10          4.8.1                 Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/riftbit/wo...     12.1.16         5.8.1                 Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/groundhog2...     0.13.1          6.7.2-apache          A Helm chart for Wordpress on Kubernetes          
https://artifacthub.io/packages/helm/rock8s/wor...     0.12.4          v0.12.4               An open source content management system          
https://artifacthub.io/packages/helm/homeenterp...     0.5.0           5.9.3-php8.1-apache   Blog server                                       
https://artifacthub.io/packages/helm/mcouliba/w...     0.1.0           1.16.0                A Helm chart for Kubernetes                       
https://artifacthub.io/packages/helm/wordpress-...     6.1.1-3.0rc5    master                Lightweight Wordpress installation with additio...
https://artifacthub.io/packages/helm/wordpress-...     1.0.0           1.1                   This is a package for configuring wordpress and...
https://artifacthub.io/packages/helm/securecode...     4.14.0          4.0                   Insecure & Outdated Wordpress Instance: Never e...
https://artifacthub.io/packages/helm/wordpressm...     1.0.0                                 This is the Helm Chart that creates the Wordpre...
https://artifacthub.io/packages/helm/bitpoke/wo...     0.12.4          v0.12.4               Bitpoke WordPress Operator Helm Chart             
https://artifacthub.io/packages/helm/bitpoke/wo...     0.12.4          v0.12.4               Helm chart for deploying a WordPress site on Bi...
https://artifacthub.io/packages/helm/kube-wordp...     0.1.0           0.0.1-alpha           Helm Chart for Wordpress installation on MySQL ...
https://artifacthub.io/packages/helm/riotkit-or...     2.1.0-alpha4    v2.1-alpha4           Lightweight Wordpress installation with additio...
https://artifacthub.io/packages/helm/phntom/bin...     0.0.4           0.0.3                 www.binaryvision.co.il static wordpress           
https://artifacthub.io/packages/helm/gh-shessel...     2.1.12          6.1.1                 Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/rock8s/wor...     6.7.1                                                                                   
https://artifacthub.io/packages/helm/sikalabs/w...     0.1.2                                                                                   
https://artifacthub.io/packages/helm/gh-shessel...     6.3.1           6.3.1                 Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/wordpressh...     0.1.0           1.16.0                A Helm chart for Kubernetes                       
https://artifacthub.io/packages/helm/wordpress-...     0.1.0           1.1                   this is chart create the wordpress with suitabl...
https://artifacthub.io/packages/helm/wordpress-...     0.0.1                                 Helm Chart for wordpress-gatsby                   
https://artifacthub.io/packages/helm/bitpoke/bi...     1.8.19          1.8.19                The Bitpoke App for WordPress provides a versat...
https://artifacthub.io/packages/helm/sonu-wordp...     1.0.0           2                     This is my custom chart to deploy wordpress and...
https://artifacthub.io/packages/helm/uvaise-wor...     0.2.0           1.1.0                 Wordpress for Kubernetes                          
https://artifacthub.io/packages/helm/wordpress/...     0.2.0           1.1.0                 Wordpress for Kubernetes                          
https://artifacthub.io/packages/helm/wordpress-...     1.0.0           2                     This is my custom chart to deploy wordpress and...
https://artifacthub.io/packages/helm/bitpoke/stack     0.12.4          v0.12.4               Your Open-Source, Cloud-Native WordPress Infras...
https://artifacthub.io/packages/helm/securecode...     4.14.0          v3.8.28               A Helm chart for the WordPress security scanner...
https://artifacthub.io/packages/helm/wordpresss...     1.1.0           5.8.2                 Web publishing platform for building blogs and ...
https://artifacthub.io/packages/helm/viveksahu2...     1.0.0           2                     This is my custom chart to deploy wordpress and...
https://artifacthub.io/packages/helm/jinchi-cha...     0.2.0           1.1.0                 Wordpress for Kubernetes                          
https://artifacthub.io/packages/helm/six/wordress      0.2.0           1.1.0                 Wordpress for Kubernetes                          
https://artifacthub.io/packages/helm/projet-dev...     0.1.0           1.16.0                A Helm chart for wordpress deployed on Azure Ku...
https://artifacthub.io/packages/helm/wordpressm...     0.1.0           1.1                                                                     
```

Lets add the bitnami repo

```sh
controlplane ~ ➜  helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
```

Lets search joomla in the new repo

```sh
controlplane ~ ➜  helm search repo joomla
NAME            CHART VERSION   APP VERSION     DESCRIPTION                                       
bitnami/joomla  20.0.4          5.1.2           DEPRECATED Joomla! is an award winning open sou...
```

Lets list all repo on this node

```sh
controlplane ~ ✖ helm repo list
NAME            URL                                                 
bitnami         https://charts.bitnami.com/bitnami                  
puppet          https://puppetlabs.github.io/puppetserver-helm-chart
hashicorp       https://helm.releases.hashicorp.com   
```

Lets install drupal package from the bitnami repo

```sh
controlplane ~ ➜  helm install bravo bitnami/drupal
NAME: bravo
LAST DEPLOYED: Tue Apr  1 19:11:00 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: drupal
CHART VERSION: 21.2.0
APP VERSION: 11.1.5

Did you know there are enterprise versions of the Bitnami catalog? For enhanced secure software supply chain features, unlimited pulls from Docker, LTS support, or application customization, see Bitnami Premium or Tanzu Application Catalog. See https://www.arrow.com/globalecs/na/vendors/bitnami for more information.** Please be patient while the chart is being deployed **

1. Get the Drupal URL:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w bravo-drupal'

  export SERVICE_IP=$(kubectl get svc --namespace default bravo-drupal --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
  echo "Drupal URL: http://$SERVICE_IP/"

2. Get your Drupal login credentials by running:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default bravo-drupal -o jsonpath="{.data.drupal-password}" | base64 -d)

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
```

Lets list package installed with helm

```sh
controlplane ~ ✖ helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART            APP VERSION
bravo   default         1               2025-04-01 19:11:00.006010173 +0000 UTC deployed        drupal-21.2.0    11.1.5     
```

Lets uninstall bravo drupal

```sh
controlplane ~ ➜  helm uninstall bravo 
release "bravo" uninstalled
```

Lets download bitnami apache package but not install

```sh
controlplane ~ ➜  helm pull --untar bitnami/apache

controlplane ~ ➜  ls
apache
```


```sh
controlplane ~ ➜  helm install mywebapp ./apache
NAME: mywebapp
LAST DEPLOYED: Tue Apr  1 19:20:43 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: apache
CHART VERSION: 11.3.5
APP VERSION: 2.4.63

Did you know there are enterprise versions of the Bitnami catalog? For enhanced secure software supply chain features, unlimited pulls from Docker, LTS support, or application customization, see Bitnami Premium or Tanzu Application Catalog. See https://www.arrow.com/globalecs/na/vendors/bitnami for more information.

** Please be patient while the chart is being deployed **

1. Get the Apache URL by running:

** Please ensure an external IP is associated to the mywebapp-apache service before proceeding **
** Watch the status using: kubectl get svc --namespace default -w mywebapp-apache **

  export SERVICE_IP=$(kubectl get svc --namespace default mywebapp-apache --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
  echo URL            : http://$SERVICE_IP/


WARNING: You did not provide a custom web application. Apache will be deployed with a default page. Check the README section "Deploying your custom web application" in https://github.com/bitnami/charts/blob/main/bitnami/apache/README.md#deploying-a-custom-web-application.



WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
```

Lets Install the apache from the downloaded helm package.
Release name: mywebapp
Note: Do make changes accordingly so that 2 replicas of the webserver are running and the http is exposed on nodeport 30080.

```sh
controlplane ~ ➜  helm install mywebapp ./apache
NAME: mywebapp
LAST DEPLOYED: Tue Apr  1 19:20:43 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: apache
CHART VERSION: 11.3.5
APP VERSION: 2.4.63

Did you know there are enterprise versions of the Bitnami catalog? For enhanced secure software supply chain features, unlimited pulls from Docker, LTS support, or application customization, see Bitnami Premium or Tanzu Application Catalog. See https://www.arrow.com/globalecs/na/vendors/bitnami for more information.

** Please be patient while the chart is being deployed **

1. Get the Apache URL by running:

** Please ensure an external IP is associated to the mywebapp-apache service before proceeding **
** Watch the status using: kubectl get svc --namespace default -w mywebapp-apache **

  export SERVICE_IP=$(kubectl get svc --namespace default mywebapp-apache --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
  echo URL            : http://$SERVICE_IP/


WARNING: You did not provide a custom web application. Apache will be deployed with a default page. Check the README section "Deploying your custom web application" in https://github.com/bitnami/charts/blob/main/bitnami/apache/README.md#deploying-a-custom-web-application.



WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
```


