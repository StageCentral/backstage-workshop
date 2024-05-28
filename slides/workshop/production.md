# Running Backstage in Production

- Throughout our training we've been running Backstage in development mode

- Running in production usually involves the following;
    - Building a Docker image
    - Storing the Docker image on a container registry
    - Referencing the image in a Kubernetes Deployment YAML
    - Applying that Deployment to a Kubernetes cluster

---

## Production Database

- Backstage uses PostgreSQL as its DB backend

- Some users mentioned replacing PostgreSQL with MySQL but it's not officially supported

- Optionally a Redis-based cache can be used for performance optimisation

- When running on cloud - consider using managed DB and cache for ease of operation

---

## Change Management

- As we've seen - almost every change requires code rebuild and redeploy

- As your live Backstage instance usually has code changes - upgrades become challenging. See the [upgrade helper](https://backstage.github.io/upgrade-helper/).

- Orgs usually have to implement a CI/CD pipeline to test and redeploy Backstage changes

- The community is working to resolve this in order to allow:
    - Dynamic plugin loading
    - Dynamic (live) configuration changes
    - Seamless updates

---
## Scaling

- The  straight-forward way to scale Bacsktage is to deploy multiple identical instances and distribute incoming requests across them. These instances must share the same database and optionally - cache and search.  

- Another method for scaling Backstage is to break apart the backend into multiple different services, each running a different set of plugins. This requires us to route requests to the appropriate backends based on the plugin ID. Both for ingress, but also internal traffic between Backstage backends, which is done by creating a custom implementation of the [DiscoveryService](https://backstage.io/docs/reference/backend-plugin-api.discoveryservice) interface.

- And if you're planning to run Backstage on Kubernetes and want to optimize your clusters - check out https://perfectscale.io