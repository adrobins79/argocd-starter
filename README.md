
[Guide to ArgoCD and Kind](https://medium.com/@chirayukapoor/running-argo-cd-locally-with-kind-and-nginx-ingress-26b31cece300)

Create [Cluster](https://kind.sigs.k8s.io/docs/user/configuration/)

```kind create cluster --config=kind-argocd.yaml```

Add [Ingress](https://docs.nginx.com/nginx-ingress-controller/)

```kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml```

Add [ArgoCD](https://argo-cd.readthedocs.io/en/stable/getting_started/)

```kubectl create namespace argocd```

```kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml```


```cd man``` (assumes pwd == project root)

```kubectl apply -f ingress-argocd.yaml```

Get Password

```kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo```

Open Admin

```open http://argocd-server.local:8080/ ``` -- Does not work due to protocol switching at ingress

```open https://argocd-server.local:8443/applications```

[Installing Argo Rollouts](https://codefresh.io/learn/argo-rollouts/)

```kubectl create ns argo-rollouts```

Adding Rollouts

[Install Rollout CLI](https://argo-rollouts.readthedocs.io/en/stable/installation/)

```brew install argoproj/tap/kubectl-argo-rollouts```

```helm repo add argo https://argoproj.github.io/argo-helm```

```helm install argo-rollout-release --set dashboard.enabled=true argo/argo-rollouts -n argo-rollouts ```

Open Dashboard
Note: will be stuck loading until there is a rollout in the namespace

```kubectl port-forward -n argo-rollouts service/argo-rollout-release-argo-rollouts-dashboard 3100:3100```


Deploy the rollout (defines rollout strategy)

```cd man``` (assumes pwd == project root)

```kubectl apply -n argo-rollouts -f rollout.yaml```

Deploy svc that references rollout

```cd man``` (assumes pwd == project root)

```kubectl apply -n argo-rollouts -f rollouts-demo-service.yaml```

Watch rollout

```kubectl argo rollouts -n argo-rollouts get rollout rollouts-demo --watch```

Port forward to demo app

```kubectl port-forward -n argo-rollouts service/rollouts-demo 8100:80```

View demo app (should be blue now)

```open localhost:8100```

Update Rollout 

```kubectl argo rollouts -n argo-rollouts set image rollouts-demo rollouts-demo=argoproj/rollouts-demo:yellow```

Watch rollout again 
Note: its now paused at 20% for operator intervention

```kubectl argo rollouts -n argo-rollouts get rollout rollouts-demo --watch```

Promote the new update to 100%

```kubectl argo rollouts -n argo-rollouts promote rollouts-demo```

Add ingress to rollouts demo

Add host entry 

```127.0.0.1 rollouts-demo.local```

Create ingress

```cd man``` (assumes pwd == project root)

```kubectl apply -n argo-rollouts -f rollouts-demo-ingress.yaml```

View demo app 

```open [Rollout demo](http://rollouts-demo.local:8080/)```


Add Analysis Template

```cd man``` (assumes pwd == project root)

```kubectl apply -n argo-rollouts -f rollouts-demo-analysis.yaml```

Replace rollout canary strategy with one that refers to the analysis template

```
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: {duration: 10}
      - analysis:
          templates:
          - templateName: canary-check
      - setWeight: 40
      - pause: {duration: 10}
      - setWeight: 60
      - pause: {duration: 10}
      - setWeight: 80
      - pause: {duration: 10}
```
Apply rollout update

```cd man``` (assumes pwd == project root)

```kubectl apply -n argo-rollouts -f rollout.yaml```

Check status of rollout

```kubectl argo rollouts get rollout -n argo-rollouts rollouts-demo```

Update rollout to watch progress

Note: Change to opposite tag of what is currently deployed either blue or yellow

```kubectl argo rollouts -n argo-rollouts set image rollouts-demo rollouts-demo=argoproj/rollouts-demo:yellow```

Watch status of rollout 
Note: Confirm analysis runs as expected

```kubectl argo rollouts get rollout -n argo-rollouts rollouts-demo --watch```

Optionally Cancel Rollout

```kubectl argo rollouts abort -n argo-rollouts rollouts-demo```