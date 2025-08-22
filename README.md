# Centrlized argo app chart

## Introduction

This is a centrlized Apps of apps chart.

It asssumes a single repo with the directory structure
````
(root)
|
|----- helm
|       |----- charts
|       |        |----- chart-a
|       |        |----- chart-b
|       |
|       |----- values
|                |----- values-a
|                |----- values-b
````

## Example
### 3 tier app
#### Value
```
global:
  defaultProject: default
  defaultCommit: head
  defaultRepo: default
  paths:
    dirValues: ../../values
    dirCharts: helm/charts
  servers:
    default: https://kubernetes.default.svc
    qa-test: test-server
  repos:
    default: "git@github.com:MyOrg/infra/prod.git"
    qa-test: test repo

helmApplicationList:
  - name: postgresql
    source: {}
    destination: {}
    syncPolicy:
      automated: {}
      syncOptions:
        - CreateNamespace=true

  - name: be
    source: {}
    destination: {}
    syncPolicy:
      automated: {}
      syncOptions:
        - CreateNamespace=true

  - name: fe
    source: {}
    destination: {}
    syncPolicy:
      automated: {}
      syncOptions:
        - CreateNamespace=true
```
#### Result
```
---
# Source: argoprojects/templates/argoproject.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: postgresql
spec:
  project: default
  source:
    repoURL: "git@github.com:MyOrg/infra/prod.git"
    targetRevision: "head"
    path: "helm/charts/postgresql"
    helm:
      valueFiles:
      - "../../values/postgresql"
  destination:
    server: "https://kubernetes.default.svc"
    namespace: postgresql
  syncPolicy:
    automated: {}
    syncOptions:
    - CreateNamespace=true
---
# Source: argoprojects/templates/argoproject.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: be
spec:
  project: default
  source:
    repoURL: "git@github.com:MyOrg/infra/prod.git"
    targetRevision: "head"
    path: "helm/charts/be"
    helm:
      valueFiles:
      - "../../values/be"
  destination:
    server: "https://kubernetes.default.svc"
    namespace: be
  syncPolicy:
    automated: {}
    syncOptions:
    - CreateNamespace=true
---
# Source: argoprojects/templates/argoproject.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: fe
spec:
  project: default
  source:
    repoURL: "git@github.com:MyOrg/infra/prod.git"
    targetRevision: "head"
    path: "helm/charts/fe"
    helm:
      valueFiles:
      - "../../values/fe"
  destination:
    server: "https://kubernetes.default.svc"
    namespace: fe
  syncPolicy:
    automated: {}
    syncOptions:
    - CreateNamespace=true
```

## Value file structure
### Overview
```
global:
  defaultProject: default
  defaultCommit: head
  defaultRepo: default
  paths:
    dirValues: ../../values
    dirCharts: helm/charts
  servers:
    default: https://kubernetes.default.svc
    <additional server>: "<additional server url>"
  repos:
    default: "<Your default repo>"
    <additional repo>: "<additional repo url>"

helmApplicationList:
  - name: example
    # Override the default application argoproject
    overrideProject: myapp
    source: {}
      # Overrride the default repo based on the global.repos (default is default)
      # overrideRepo: "" # infra
      # # override the default target commit
      # overrideCommit: ""
      # # override the soruce path(default assumes <global.paths.dirCharts>/<app name>)
      # overrideSourcePath: ""
      # values:
      #   enable: true
      #   # fully override the source values list
      #   overrideSourceValuesList: []
      #   # add extravalues
      #   extraValuesList: []
    destination: {}
      # overrides the target namespace (default app name)
      # overrideNamespace: ""
      # override the default server based ont eh global.servers (default: default server)
      # overrideServer: ""
    syncPolicy:
      automated: {}
      syncOptions:
        - CreateNamespace=true
```
### Global
|Key | Format | Description |
--------------------------
|defaultProject|String|The default project name|
|defaultCommit|String|The default targetCommitRef|
|defaultRepo|String|The default repo key|
|paths.dirValues|String|The default value folder path|
|paths.dirCharts|String|The default chart folder path|
|servers|Dictionary<String>|Dcitionary of the servers|
|repos|Dictionary<String>|Dcitionary of the repositories|
### helmApplicationList
|Key|Format|Description|
--------------------------
| Key                                   | Format         | Description                                         |
|---------------------------------------|----------------|-----------------------------------------------------|
| name                                  | String         | Application name                                    |
| overrideProject                       | String         | Overrides the application project                   |
| source.overrideRepo                   | String         | Overrides the source repository key                 |
| source.overrideCommit                 | String         | Overrides the commitTargetRef                       |
| source.overrideSourcePath             | String         | Overrides the chart path                            |
| source.values.enable                  | bool           | Enables custom value folder (Default true)          |
| source.values.overrideSourceValuesList| List<String>   | Overrides the sources values list                   |
| source.values.extraValuesList         | List<String>   | Adds extra value files to the default option        |
| destination.overrideNamespace         | String         | Overrides the target namespace                      |
| destination.overrideServer            | String         | Overrides the target server key                     |
| syncPolicy                            | YAML Object    | Sync policy yaml object                             |
