# The Kubernetes Plugin

The feature is made up of two plugins: 

- @backstage/plugin-kubernetes 
- @backstage/plugin-kubernetes-backend


The frontend plugin exposes information to the end user in a digestible way.

The backend wraps the mechanics to connect to Kubernetes clusters to collect the relevant information.

---

## Adding the Kubernetes frontend plugin

The first step is to add the Kubernetes frontend plugin to your Backstage application.

.lab[

```bash
#change to your Backstage root directory
cd ~/backstage
#install the module
yarn add --cwd packages/app @backstage/plugin-kubernetes
```
]

Once the package has been installed, you need to import the plugin in your app by adding the "Kubernetes" tab to the respective catalog pages.

---

## Adding the 'Kubernetes' tab to entity pages

In `packages/app/src/components/catalog/EntityPage.tsx` - add the following:

.lab[
```typescript
//first - the import
import { EntityKubernetesContent } from '@backstage/plugin-kubernetes';
// now we can add the tab to any number of pages
//the service page is shown as an
// example here
const serviceEntityPage = (
  <EntityLayout>
    {/* other tabs... */}
    <EntityLayout.Route path="/kubernetes" title="Kubernetes">
      <EntityKubernetesContent refreshIntervalMs={30000} />
    </EntityLayout.Route>
  </EntityLayout>
);
```
]

---

## Adding the 'Kubernetes' tab to entity pages

That's it for the frontend!

But now, we need the Kubernetes Backend plugin for the frontend to work.

---

## Adding Kubernetes Backend plugin

We need to install another NPM package:

.lab[

```bash
#navigate to your Backstage root directory
cd ~/backstage
#install the module
yarn add --cwd packages/backend @backstage/plugin-kubernetes-backend
```
]

---
## Adding Kubernetes Backend plugin

In `packages/backend/src/index.ts` :
```
const backend = createBackend();

// Other plugins...
// Add this line:
backend.add(import('@backstage/plugin-kubernetes-backend/alpha'));

backend.start();
```
---

## Checking if it works

Restart Backstage now:
.lab[
```
yarn dev
```
]

Browse to your Backstage UI and check the "Kubernetes" tab on one of your existing services.

The tab should show: 

  *Missing Annotation*

  *The annotation backstage.io/kubernetes-id is missing. You need to add the annotation to your component if you want to enable this tool.*

---

## Configuring Kubernetes integration


Configuring the Backstage Kubernetes integration involves two steps:

- Enabling the backend to collect objects from your Kubernetes cluster(s).
- Surfacing your Kubernetes objects in catalog entities

---

## Getting a Kubernetes Cluster

In our labs we already have `k3d` preinstalled.

If you don't have k3d - install it with:

`curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | TAG=v5.5.1 bash`

Create a cluster:

.lab[
```bash
k3d cluster create mycluster
#verify your cluster
kubectl cluster-info
```
]

---

## Configure Kubernetes Access

Backstage Kubernetes plugin supports various ways of locating K8S clusters and a number of authentication providers.

We will be using the `clusterLocatorMethod.type='config'` and `cluster.LocatorMethod.clusters.authProvider='serviceAccount'`

So we need to create a service account and assign a role to it first.

---

## Creating a Service Account

The Kubernetes plugin requires read-only cluster-wide access. We will create a Kubernetes `serviceAccount` and bind it to `clusterRole` named `view`.

.lab[
```bash
  kubectl create sa -n kube-system backstage
  kubectl create clusterrolebinding backstage \
    --clusterrole=view \
    --serviceaccount=kube-system:backstage
  export K8S_CONFIG_SA_TOKEN=$(kubectl create token -n kube-system backstage)
  export K8S_CONFIG_URL=$(kubectl config view \ 
    -ojsonpath="{ .clusters[0].cluster.server }")
  export K8S_CONFIG_CA_DATA=$(kubectl config view --raw \ 
    -ojsonpath="{ .clusters[0].cluster.certificate-authority-data }")
```
]

---

## Add cluster information to your app-config,yaml

.lab[
In your app-config.yaml add:
```yaml
kubernetes:
  serviceLocatorMethod:
    type: 'multiTenant'
  clusterLocatorMethods:
    - type: 'config'
      clusters:
        - url: ${K8S_CONFIG_URL} #cluster API url
          name: mycluster
          authProvider: 'serviceAccount'
          serviceAccountToken: ${K8S_CONFIG_SA_TOKEN} #the token
          caData: ${K8S_CONFIG_CA_DATA} #CAData for the cluster
```
]

---

## Let's check if it's working

- Browse to your `backster` service in Backstage

- Go to 'Kubernentes' tab

- You should now see the cluster but no Kubernetes objects found.

- We now need to annotate our entity and label our objects.

---

## Finding K8S Resources Related to Our Entity

There are two ways to surface your Kubernetes components as part of an entity. 

- Using the label selector annotation

- Using the standard backstage annotation

---

## Using the Standard Annotation

In order for Backstage to detect that an entity has Kubernetes components, the following annotation should be added to the entity's catalog-info.yaml:

```yaml
annotations:
  'backstage.io/kubernetes-id': backster
```

---

## Adding the Standard Annotation

.lab[
```bash
#Assuming we already have the backster repo code on our VM
cd ~/backster
#add the annotation from the previous slide to catalog-info.yaml
git commit -a -m "Add k8s annotation"
git push origin main
```
]

---

## Labelling K8S objects

We now need to:

- Create some pods

- Add the annotation to see them in Backstage

.lab[
```bash
kubectl create deployment backster --image=otomato/httpenv --replicas=3
kubectl label pods,deployments --all backstage.io/kubernetes-id=backster
```
]

---

## Check if this worked

- Go back to Backstage UI

- Browse to backster->Kubernetes and see the related pods

---

## Using Label Selector for K8S Discovery

- It's possible to use existing Kubernetes labels for finding the objects related to the entiy

- The annotation on the entity should look like:
```yaml
annotations:
  'backstage.io/kubernetes-label-selector': 'app=my-app,component=front-end'
```

- This is a better approach if you already have well-known labels on your objects

---

## Finding K8s Resources with a Selector

.lab[

Let's expose our deployment:
```bash
kubectl expose deployment backster --port=8000
#the deployment already has the 'app=backster' label
#and now the service has it too
```

And change the annotation in our `backster/catalog-info.yaml`:

```yaml
annotations:
  backstage.io/kubernetes-label-selector: 'app=backster'
```
]

Reload the Kubernetes tab of your 'backster' component.

---
