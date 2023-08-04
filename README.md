# ArgoCD with Argo Rollouts POC

A simple example of managing a Helm deployment with ArgoCD (and GitOps), and then triggering a Argo Rollouts blue/green deployment with a single Git commit

## Prerequisites
- Any kubernetes cluster, I used minikube
```
minikube start
```
- The ArgoCD CLI
```
brew install argocd
```
- The Argo Rollouts CLI
```
brew install argoproj/tap/kubectl-argo-rollouts
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

5. Login to the ArgoCD API
```
argocd login localhost:8080 --insecure --username admin \
  --password $(kubectl get secret argocd-initial-admin-secret -n argocd \
  --template={{.data.password}} | base64 -d)
```

6. Bootstrap Argo Applications. All ArgoCD Application definitions will live in the `argocd-applications/` directory. With this one command we can provision everything!
```
argocd app create bootstrap-applications --repo https://github.com/domenicbove/argocd-test.git \
--path argocd-applications --dest-server https://kubernetes.default.svc \
--dest-namespace default --sync-policy auto
```

7. In the browser view your applications syncing, navigate to https://localhost:8080 and for user enter `admin`

8. Copy the Admin Password and paste into browser
```
kubectl get secret argocd-initial-admin-secret -n argocd --template={{.data.password}} | base64 -d | pbcopy
```

9. Argo will deploy everything automatically because `--sync-policy auto` - you can watch the sync status in the ui. Eventually lets try our podinfo sample app. After running below command go to http://localhost:9898
```
kubectl -n podinfo port-forward svc/podinfo 9898:9898
```

10. No suppose you dislike the background color, lets update that. Change the color in `podinfo/values.yaml`. And push the change
```
ui:
  color: "#bf193d" # update this color hex
```

```
git add . && git commit -m 'changed color' && git push origin main
```

11. Because I have setup the podinfo chart to use an Argo `Rollout` object, and sync is automatic we will get new pods spun up! Lets port-forward the preview service!
```
kubectl -n podinfo port-forward svc/podinfo-preview 9999:9898
```

12. Check out the new color! Go to http://localhost:9999

13. If you like it lets promote it, but first lets watch the promotion
```
kubectl argo rollouts get rollout podinfo --watch
```

14. Next promote it!
```
kubectl argo rollouts promote podinfo
```
