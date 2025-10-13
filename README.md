# GitOps mono-repo PoC

## Repository structure

```bash
├── app-of-apps
│   ├── apps # App definitions
│   │   └── gda
│   │       └── app1.yml
│   ├── base
│   │   ├── apps  # App ApplicationSets
│   │   │   └── applications-app-set.yml 
│   │   └── infra # Infra ApplicationSets
│   └── clusters
│       └── c01
│           ├── cluster-conf.yml     # Cluster level conf
│           └── gda
│               ├── dev
│               │   ├── apps
│               │   │   └── app1.yml -> ../../../../../apps/gda/app1.yml # Symlink to default app definition
│               │   └── env-conf.yml # Environment level conf
│               ├── int
│               │   ├── apps
│               │   │   └── app1.yml # Custom app definition
│               │   └── env-conf.yml 
│               └── suite-conf.yml   # Suite level conf
└── config-apps    # App helm charts
    └── nginx-chart
        ├── Chart.yaml
        ├── templates
        │   ├── deployment.yml
        │   └── service.yml
        ├── values-dev.yml
        ├── values-int.yml
        ├── values-pre-prod.yml
        ├── values-prod.yml
        └── values.yml
```

## Pre-requisite

ArgoCD CLI login
```bash
argocd login --core
```

Add clusters to ArgoCD
```bash
argocd cluster add kube-ctx1 --yes --name c01
argocd cluster add kube-ctx2 --yes --name c02
```

Add Role Binding for each env and suite.
```bash
for e in {'dev','int','pre-prod','prod'}; do
    for s in {'gda','hr'}; do oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n $s-$e; done
done
```

## Create ApplicationSet
```bash
oc apply -f repositories/app-of-apps/base/apps/applications-app-set.yml
```

## Notes

1. [ArgoCD - Helm Value Precedence](https://argo-cd.readthedocs.io/en/latest/user-guide/helm/#helm-value-precedence)
```
    lowest  -> valueFiles
            -> values
            -> valuesObject
    highest -> parameters
```
2. [ArgoCD - Adding a cluster](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-management/#adding-a-cluster) 
3. [Sprig Function Documentation](https://masterminds.github.io/sprig/)