# learn_helm

# section 2: exploring heml's nginx chart
```bash
# `artifact hub` bunch of different https://artifacthub.io/
# helm charts exist in a repository in compressed format (.tar.gz)
# helm doesn't have default repo so we must add our own
helm repo add bitnami https://charts.bitnami.com/bitnami

# check that it has been added by running
helm repo list

# find stuff in repo
helm search repo nginx

# find stuff in repo by version
helm search repo --versions nginx
helm search repo --versions "nginx servers"

#helm will use same context as kubectl
kubectl config get-contexts
# don't want helm to use the default so let's create a new one
kubectl create ns web
# now change default to web ns
kubectl config set-context --current --namespace=web

# now let's install the chart (installation name)(repo name)
helm install nginx01 bitnami/nginx 

# examine status of nginx01
kubectl get svc nginx01 # pending because minkube will never create a load balancer
minikube service -n web --url nginx01
kubect get all

# we want to add more than 1 instance of a installation
# need another nginx of different version for another project
kubectl config get-contexts # current context

# install nginx different version in another namespace
helm install nginx02 --version 9.3.5 -n default bitnami/nginx
kubectl get pods -n default # check
kubectl exec -n default -it nginx02-57696bd848-jzbl2 bash

# helm configures this stuff
kubectl get svc -n default # default port etc

# change service type and port 
# there ways to change values
# when you download a chart there will be defaults that
# you can override with values.yaml (1)
# (2) is using the `--set` flag on cli
helm install nginx03 --values values.yaml --set service.port=8080 bitnami/nginx

# if you want to see all the stuff you installed via helm
helm list # only shows in web namespace
helm list --all-namespaces

# want to upgrade existing chart to use upgraded helm values.yaml
# you upgrade heml chart when you upgrade the config or version
helm search repo "nginx server"
helm upgrade nginx01 --set service.type=NodePort --set service.port=8080 bitnami/nginx
kubectl get svc

# upgrade via values file
helm upgrade nginx01 --values values.yaml bitnami/nginx

# you can also upgrade installation version
helm upgrade nginx01 --version 9.3.5 bitnami/nginx

# to ensure that values are used to override default values
helm upgrade nginx01 --version 9.3.5 --reuse-values bitnami/nginx

helm uninstall nginx01
```
# section 3 helm testing the waters
```bash
helm install nginx03 --values section_2/values.yaml --set service.port=8080 --dry-run bitnami/nginx

# inspecting a release 
# how to get info about heml release
helm get notes nginx03 # deployment notes
helm get values nginx03 # user defined values
helm get values nginx03 --all # all values
helm get manifest nginx03 # manifest after it was applied to cluster

# history and rollback
# want to know release status
helm list 
helm history nginx03 # will show history of release attempts and will provide useful debug info
helm rollback nginx03 1 # rollback to previous release
# uninstalling a chart will remove all the history
# to keep history
helm uninstall nginx03 --keep-history
helm history nginx03 # now we can look at history still
# recover
helm rollback nginx03 3 # recover revision 3

# note that helm will mark a release `failed` if
# k8s rejects the manifest; however, something like
# `imagePullError` in k8s will get passed by helm
# even tho it will result in a failed release

# tips for helm install/upgrade release
# 1) autogenerate release name
helm install nginx01 bitnami/nginx
# helm uses release names to create
# k8s objects who names must be unq
kubectl get deployments
kubectl get svc 
# creating a unq name via a script everytime
# would be annoying
helm install bitnami/nginx --generate-name

# 2) similarly auto-generate the namespace
# can't release to ns that doesn't exist
helm install -n apps bitnami/nginx --generate-name --create-namespace

# 3) install or upgrade release if it exists
# CI/CD
helm upgrade --install nginx01 --set service.port=8008 bitnami/nginx # updates the release since it's already there
# release does not exist and will actually just install it
helm upgrade --install nginx10 --set service.port=8008 bitnami/nginx # updates the release since it's already there

# 4) verify that hte application is running
helm upgrade nginx01 --set image.registry=foobar bitnami/nginx --wait --timeout 10s
# --wait will ensure that helm verifies pods are in running state
# --timeout 10s
```

# section 4 helm chart creation 101
```bash
```