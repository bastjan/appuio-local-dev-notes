# Appuio Local DEV POC


```sh
kind create cluster --config=kind-with-oicd.yaml
# OR
k3d cluster create $CLUSTER_NAME \
    --k3s-arg "--kube-apiserver-arg=oidc-issuer-url=https://id.dev.appuio.cloud/auth/realms/appuio-cloud-dev" \
    --k3s-arg "--kube-apiserver-arg=oidc-client-id=local-dev-environment" \
    --k3s-arg "--kube-apiserver-arg=oidc-username-claim=email" \
    --k3s-arg "--kube-apiserver-arg=oidc-groups-claim=groups"
```

Setup cluster and kubectl

```sh
kubectl apply -f - <<EOF
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: oidc-cluster-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: Group
  name: admin
EOF

kubectl oidc-login setup \
    --oidc-issuer-url=https://id.dev.appuio.cloud/auth/realms/appuio-cloud-dev \
    --oidc-client-id=local-dev-environment

kubectl config set-credentials oidc-user \
    --exec-api-version=client.authentication.k8s.io/v1beta1 \
    --exec-command=kubectl \
    --exec-arg=oidc-login \
    --exec-arg=get-token \
    --exec-arg=--oidc-issuer-url=https://id.dev.appuio.cloud/auth/realms/appuio-cloud-dev \
    --exec-arg=--oidc-client-id=local-dev-environment \
    --exec-arg=--oidc-extra-scope="email offline_access profile openid"

kubectl get pods --user=oidc-user -n default

kubectl config set-context --current --user=oidc-user
```
