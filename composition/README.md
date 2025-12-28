# crossplane-learnings
This repo is used to learn crossplane in order to create infra using APIs.

# Link to install crossplane
Crossplane installs into an existing Kubernetes cluster, creating the Crossplane pod.
https://docs.crossplane.io/latest/get-started/install/ 

# Get Started With Composition
This guide shows how to create a new kind of custom resource named App. When a user calls the custom resource API to create an App, Crossplane creates a Deployment and a Service.
https://docs.crossplane.io/latest/get-started/get-started-with-composition/

## Create the custom resource

Follow these steps to create a new kind of custom resource using Crossplane:

1. Define the schema of the App custom resource
2. Install the function you want to use to configure how Crossplane composes apps
3. Configure how Crossplane composes apps

```bash
kubectl apply -f deployments/xrd.yaml
```
Before installation of XRD,
```bash
✗ kg App
error: the server doesn't have a resource type "App"
```
After installation of XRD,
```bash
✗ kg Apps
No resources found in crossplane-system namespace.
```

## Install the function 
```bash
kubectl apply -f deployments/fn.yaml
```
Once the pod is created for this function, you can see the function will be healthy. 
```bash
✗ kg function
NAME                                              INSTALLED   HEALTHY   PACKAGE                                                                     AGE
crossplane-contrib-function-patch-and-transform   True        True      xpkg.crossplane.io/crossplane-contrib/function-patch-and-transform:v0.8.2   11s
```

## Configure the composition
A composition tells Crossplane what functions to call when you create or update a composite resource (XR).

Create a composition to tell Crossplane what to do when you create or update an App XR.

```bash
kubectl apply -f composition/composition.yaml
```

## Use the custom resource
Crossplane now understands App custom resources.

```bash
kubectl apply -f composition/app.yaml
```
This created both a deployment and a service.
```bash
✗ kg app
NAME     SYNCED   READY   COMPOSITION   AGE
my-app   True     True    app-yaml      3m15s
(base) (⎈|k8s-dev-eks-hsalluri:test)➜  crossplane-learnings git:(main) ✗ kg svc -l "example.crossplane.io/app"=my-app
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
my-app-03059366d266   ClusterIP   172.20.137.68   <none>        8080/TCP   3m23s
(base) (⎈|k8s-dev-eks-hsalluri:test)➜  crossplane-learnings git:(main) ✗ kgp -l "example.crossplane.io/app"=my-app
NAME                                   READY   STATUS    RESTARTS   AGE
my-app-12f0490e6768-7b88f748fb-rntsz   1/1     Running   0          80s
my-app-12f0490e6768-7b88f748fb-zzvfq   1/1     Running   0          78s
```