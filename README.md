# infisical-k8s-secrets


## Configuring operator and secrets

### Install infisical operator

#### Install helm repo
```
helm repo add infisical-helm-charts 'https://dl.cloudsmith.io/public/infisical/helm-charts/helm/charts/' 
helm repo update
```

#### Install helm chart
```
helm install --generate-name infisical-helm-charts/secrets-operator 
# Example installing app version v0.2.0 and chart version 0.1.4
helm install --generate-name infisical-helm-charts/secrets-operator --version=0.1.4 --set controllerManager.manager.image.tag=v0.2.0
```

### Setup machine ID
* UI app.infisical.com -> access control -> machine ids
* create a secret
```
kubectl create secret generic universal-auth-credentials --from-literal=clientId="<your-identity-client-id>" --from-literal=clientSecret="<your-identity-client-secret>"
```
* add secret to InfisicalSecret resource via field authentication.universalAuth.credentialsRef


### Configure secrets

#### Install CRD - InfisicalSecret
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

#### Configure deployment spec, options:

##### envFrom
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

#### env
```
env:
  - name: SECRET_NAME # The environment variable's name which is made available in the container
    valueFrom:
      secretKeyRef:
        name: managed-secret # managed secret name
        key: SOME_SECRET_KEY # The name of the key which  exists in the managed secret
```

#### volumes
```
volumes:
  - name: secrets-volume-name # The name of the volume under which secrets will be stored
    secret:
      secretName: managed-secret # managed secret name
```
Next, mount the volume to the FS
```
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          volumeMounts:
            - name: secrets-volume-name
              mountPath: /etc/secrets
              readOnly: true
          ports:
            - containerPort: 80
```




## Other know-how

### Enable auto-redeploy on secret change
Annotate the deployment spec:
```
secrets.infisical.com/auto-reload: "true"
```

### Configure global settings for infisical operator
All global configurations must reside in a Kubernetes ConfigMap named infisical-config in the namespace infisical-operator-system.
```
apiVersion: v1
kind: Namespace
metadata:
  name: infisical-operator-system
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: infisical-config
  namespace: infisical-operator-system
data:
  hostAPI: https://example.com/api # <-- global hostAPI
```

### Troubleshoot
kubectl get infisicalSecrets
kubectl describe infisicalSecret infisicalsecret-sample

## Uninstall operator
Managed secrets will NOT be uninstalled.
```
helm uninstall <release name>
```

