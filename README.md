# ArgoCD with Argo Rollouts POC

A simple example of running ArgoCD on KIND, and then triggering blue/green deployments of a sample application triggered by GitOps!

## Prerequisites
- Install Kind and start a cluster
```

Copy
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

## Usage
1. Fork this repository, so that you will have the ability to trigger rollouts

2. Clone your fork
```
git clone <repo>/argocd-test
cd argocd-test
```

3. Install ArgoCD
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
``` 

4. Expose UI
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

5. Copy down the Admin Password
```
k get secret argocd-initial-admin-secret -n argocd --template={{.data.password}} | base64 -d
```

6. Login with admin/<password>
```
argocd login localhost:8080 --insecure
```

7. Bootstrap Argo Applications. All ArgoCD Application definitions will live in the `argocd-applications/` directory. With this one command we can provision everything!
```
argocd app create bootstrap-applications --repo https://github.com/domenicbove/argocd-test.git \
--path argocd-applications --dest-server https://kubernetes.default.svc \
--dest-namespace default --sync-policy auto
```

