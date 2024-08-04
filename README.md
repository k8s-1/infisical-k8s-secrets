# infisical-k8s-secrets

## Install infisical operator

### Install helm repo
```
helm repo add infisical-helm-charts 'https://dl.cloudsmith.io/public/infisical/helm-charts/helm/charts/' 
helm repo update
```

### Install helm chart
```
helm install --generate-name infisical-helm-charts/secrets-operator 
# Example installing app version v0.2.0 and chart version 0.1.4
helm install --generate-name infisical-helm-charts/secrets-operator --version=0.1.4 --set controllerManager.manager.image.tag=v0.2.0
```

