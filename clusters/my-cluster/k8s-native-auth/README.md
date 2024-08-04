# How-to 

Note: When running with local cluster, received generic 500 HTTP response from infisical API,
linking SA JWT tokens to MI in infisical UI needs automating

## Step 1

* Create SA to be used by infisical to authenticate with k8s api-server
* Bind the service account to the system:auth-delegator cluster role, this role allows delegated authentication and authorization checks
* Create a long-lived service account JWT token
```
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: infisical-auth-token
  annotations:
    kubernetes.io/service-account.name: "infisical-auth"
```
* Link the token secret to the SA
* Finally, retrieve the `token reviewer JWT token` from the secret 

## Step 2

* Create and configure MI -> Kubernetes Auth: Organization Settings > Access Control > Machine Identities and press Create identity
* Add MI to project: Project Settings > Access Control > Machine Identities and press Add identity
* Configure InfisicalSecret fields:
    * Add identity ID to authentication.kubernetesAuth.identityId
    * Add SA ref to authentication.kubernetesAuth.serviceAccountRef
