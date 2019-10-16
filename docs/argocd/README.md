![](argocd-logo.png)
# ARGOCD

## Introduction

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.

## Architecture

![Istio](argocd_arch.png)

## Features

* Automated deployment of applications to specified target environments
* Support for multiple config management/templating tools (Kustomize, Helm, Ksonnet, Jsonnet, plain-YAML)
* Ability to manage and deploy to multiple clusters
* SSO Integration (OIDC, OAuth2, LDAP, SAML 2.0, GitHub, GitLab, Microsoft, LinkedIn)
* Multi-tenancy and RBAC policies for authorization
* Rollback/Roll-anywhere to any application configuration committed in Git repository
* Health status analysis of application resources
* Automated configuration drift detection and visualization
* Automated or manual syncing of applications to its desired state
* Web UI which provides real-time view of application activity
* CLI for automation and CI integration
* Webhook integration (GitHub, BitBucket, GitLab)
* Access tokens for automation
* PreSync, Sync, PostSync hooks to support complex application rollouts (e.g.blue/green & canary upgrades)
* Audit trails for application events and API calls
* Prometheus metrics
* Parameter overrides for overriding ksonnet/helm parameters in Git

## Installation

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Successful installation should run all the argocd modules.

```
$ kubectl get po -n argocd                                                                                                                                            
NAME                                             READY   STATUS      RESTARTS        AGE
argocd-application-controller-68f8bf79d8-579nj   1/1     Running          0          11s
argocd-dex-server-5994988c7f-2fkp9               1/1     Running          0          11s
argocd-redis-78c9595d44-lr72z                    1/1     Running          0          11s
argocd-repo-server-775496b8dd-bnjnj              1/1     Running          0          11s
argocd-server-56db6f6cb6-24zw8                   1/1     Running          0          10s

```

## Deploy an app

```
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: hsi
  namespace: argocd
spec:
  description: argocd project
  sourceRepos:
  - 'https://github.com/HealthStarInformatics/kube-hsi.git'
  destinations:
  - namespace: '*'
    server: https://kubernetes.default.svc
  clusterResourceWhitelist:
  - group: '*'
    kind: '*'
```

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: hsi-app
  namespace: argocd
spec:
  project: hsi
  source:
    repoURL: 'https://github.com/HealthStarInformatics/kube-hsi.git'
    targetRevision: HEAD
    path: hsi
    helm:
      valueFiles:
      - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: hsi
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Once you execute above manifests ArgoCD will automatically deploy the apps to kubernetes cluster.

![](https://argoproj.github.io/argo-cd/assets/argocd-ui.gif)
