# Integrating Plugins

- Backstage (frontend) is a single-page application composed of a set of plugins.

- Some of the *frontend* plugins have a correlated *backend* plugin.

- Open source plugins that you can add to your Backstage deployment can be found in the [Plugin Marketplace](https://backstage.io/plugins)

- Plugins are basically NPM modules that need to be imported in the code of our IDP

- Let's see how this is done for the popular [Kubernetes Plugin](https://backstage.io/docs/features/kubernetes/) (frontend and backend)

