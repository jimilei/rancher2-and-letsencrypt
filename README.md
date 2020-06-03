# Use Rancher 2 and Letsencrypt for Ingress

## Prerequesite
- Check for existing cert-manager
    ```
    ~$ kubectl get all -n cert-manager
    No resources found.
    ```

- Check for cluster-issuers
    ```
    ~$ kubectl describe clusterissuers letsencrypt-staging
    error: the server doesnt have a resource type "clusterissuers"
    ```

## Installing cert-manager
Also visit the [official installation guide](https://cert-manager.io/docs/installation/kubernetes/)

1. Install the cert-manager
    ```
    $ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.1/cert-manager.yaml
    ```
2. Install the CRD
    ```
    $ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.1/cert-manager.crds.yaml
    ```
3. Verify the installation

    `kubectl get pods --namespace cert-manager`
4. Configure [letsencrypt certificate issuer](https://cert-manager.io/docs/configuration/acme/)
    ```
    ~$ cat << EOF > cluster-issuer.yml
    apiVersion: cert-manager.io/v1alpha2
    kind: ClusterIssuer
    metadata:
    name: letsencrypt
    spec:
    acme:
        ...
        solvers:
        - dns01:
            cloudflare:
            email: user@example.com
            apiKeySecretRef:
                name: cloudflare-apikey-secret
                key: apikey
        selector:
            matchLabels:
            "use-cloudflare-solver": "true"
            "email": "user@example.com"
    EOF
    ~$ kubectl create -f cluster-issuer.yml
    ```

## Configure ingress

Create an ingress in rancher ui and add the following configuration:
- Rules: `Specify a hostname to use: domain.com`
- SSL/TLS Certificates: `Use default ingress controller certificate`
- Annotations:
    - `kubernetes.io/tls-acme: '"true"'`
    - `cert-manager.io/cluster-issuer: letsencrypt`

Edit the ingress and add a `secretName` under `spec` - `tls` as such
```yaml
spec:
  tls:
  - hosts:
    - domain.com
    secretName: domain-crt
```

## Troubleshooting
Lookup the output of the cert-manager log

```
~$ kubectl logs  -n cert-manager cert-manager-webhook-8c5db9fb6-5h9bx | grep controller
```
