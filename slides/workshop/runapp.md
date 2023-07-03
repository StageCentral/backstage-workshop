# Running Backstage in Development

## Initial configuration

A basic Backstage installation consists of two parts:
- A frontend React app served on  localhost:3000 (default)
- A backend Node.js app listening on  localhost:7007 (default)

This (and many other things) is configurable via the `app-config.yaml` file.

.lab[
  ```bash
  cd ~/backstage
  less app-config.yaml
  ```
]

---
## Configuring Backstage for Workshop

When running in development on your machine - the default configuration is sufficient - as everything is served from `localhost` and accessed through `localhost`

But in this workshop we wil be running Backstage dev version on a VM and accessing it remotely through our web browser.

This is also a great opprotunity to play with configuration

---
## Configuring Backstage for Our Workshop

The values in `app-config.yaml` can be:
- hard-coded
- expanded from environment variables

Let's use both options:
.lab[
```bash
export PUBLIC_IP=$(dig +short myip.opendns.com @resolver1.opendns.com)
```
]
---
## Configuring Backstage for Our Workshop

Now open `app-config.yaml` and edit the following lines:
.lab[
```yaml
app:
  title: StageCentral Workshop
  baseUrl: http://0.0.0.0:3000
...
backend:
  baseUrl: http://${PUBLIC_IP}:7007
  listen:
    port: 7007
    host: 0.0.0.0
...
  cors:
    origin: http://${PUBLIC_IP}:3000
```
]

---

## Time to Run!

Finally, we're ready to run Backstage!

.lab[
```bash
cd ~/backstage
yarn dev
```
]

This will run both the frontend and the backend applications.

Once you see: `[0] webpack compiled successfully`

Browse to <*YOUR-MACHINE-IP*>:3000

---
## An Experiment
 
An *Experiment* is a limited run of one or more ReplicaSets for the purposes of analysis. Experiments typically run for a pre-determined duration, but can also run indefinitely until stopped. 

Experiments may reference an AnalysisTemplate to run during or after the experiment. 

The canonical use case for an Experiment is to start a baseline and canary deployment in parallel, and compare the metrics produced by the baseline and canary pods for an equal comparison.

---

## Explore Argo Rollouts 

Let's see what Argo Rollouts components we have in our cluster
.exercise[
  ```bash
  kubectl get all -n argo-rollouts
  ```
]

And the custom resources used to configure the rollouts.

.exercise[
  ```bash
  kubectl get crds | grep argo
  ```
]

