# Integrating Plugins

- Backstage (frontend) is a single-page application composed of a set of plugins.

- Some of the *frontend* plugins have a correlated *backend* plugin.

- Open source plugins that you can add to your Backstage deployment can be found in the [Plugin Marketplace](https://backstage.io/plugins)

- Plugins are basically NPM modules that need to be imported in the code of our IDP

- Let's see how this is done for the popular [Kubernetes Plugin](https://backstage.io/docs/features/kubernetes/) (frontend and backend)

---

## The Kubernetes Plugin

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

And add a whole new file:  `packages/backend/src/plugins/kubernetes.ts`

```typescript
import { KubernetesBuilder } from '@backstage/plugin-kubernetes-backend';
import { Router } from 'express';
import { PluginEnvironment } from '../types';
import { CatalogClient } from '@backstage/catalog-client';

export default async function createPlugin(
  env: PluginEnvironment,
): Promise<Router> {
  const catalogApi = new CatalogClient({ discoveryApi: env.discovery });
  const { router } = await KubernetesBuilder.createBuilder({
    logger: env.logger,
    config: env.config,
    catalogApi,
    permissions: env.permissions,
  }).build();
  return router;
}
```
---

## Adding Kubernetes Backend plugin

Finally we need to register the plugin with the backend.

Import the plugin to `packages/backend/src/index.ts`. There are three lines of code you'll need to add, and they should be added near similar code in your existing Backstage backend.

.lab[
```typescript
// ..
import kubernetes from './plugins/kubernetes';

async function main() {
  // ...
  const kubernetesEnv = useHotMemoize(module, () => createEnv('kubernetes'));
  // ...
  apiRouter.use('/kubernetes', await kubernetes(kubernetesEnv));
```
]

That's it! The Kubernetes frontend and backend have now been added to your Backstage app.

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

## Configure Kunernetes Access

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