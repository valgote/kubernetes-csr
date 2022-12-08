# Certificate Signing Requests

## Create private key

```
openssl genrsa -out user.key 2048
```
```
openssl req -new -key user.key -out user.csr
```

## Create CertificateSigningRequest 

csr.yaml
```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: user
spec:
  request: <is the base64 encoded value of the CSR file content>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
```

To get CSR content:
```
cat user.csr | base64 | tr -d "\n"
```
Next:
```
kubectl apply -f csr.yaml
```
```
kubectl get csr
```
```
kubectl certificate approve user
```

## Get the certificate
```
kubectl get csr user -o jsonpath='{.status.certificate}'| base64 -d > user.crt
```

## Create Role and RoleBinding
```
kubectl apply -f rbac/role.yaml
kubectl apply -f rbac/role-binding.yaml
```

## Setup kubeconfig

Add new credentials:

```
kubectl config set-credentials user --client-key=user.key --client-certificate=user.crt --embed-certs=true
```

Add the context:
```
kubectl config set-context user --cluster=kubernetes-cluster --user=user
```

Change the context to `user`:
```
kubectl config use-context user
```

Test with failure:
```
kubectl get pods -n kube-system
```

You will get the error:
`Error from server (Forbidden): pods is forbidden: User "user" cannot list resource "pods" in API group "" in the namespace "kube-system"`

Test with success:
```
kubectl get pods
```
`
you can see the pods running in the default namespace or the ones you setup in roles and role bindings
`
