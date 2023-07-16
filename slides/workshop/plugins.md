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



