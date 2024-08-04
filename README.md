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

## Configure secrets

### Install CRD - InfisicalSecret
```
apiVersion: secrets.infisical.com/v1alpha1
kind: InfisicalSecret
...
```

The CRD is propagated to a managed k8s secret by the infisical operator
```
apiVersion: v1
data: ...
kind: Secret
```

### Configure deployment spec, options:
```
- envFrom
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        envFrom:
        - secretRef:
            name: managed-secret # <- name of managed secret
        ports:
        - containerPort: 80
```
- env
```
env:
  - name: SECRET_NAME # The environment variable's name which is made available in the container
    valueFrom:
      secretKeyRef:
        name: managed-secret # managed secret name
        key: SOME_SECRET_KEY # The name of the key which  exists in the managed secret
```
- volumes
```
volumes:
  - name: secrets-volume-name # The name of the volume under which secrets will be stored
    secret:
      secretName: managed-secret # managed secret name
```




