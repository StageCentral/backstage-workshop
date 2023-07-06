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
## Exploring Backstage UI
 
Backstage comes with the following UI components out of the box:
- The Catalog
- The API Explorer
- Tech Docs
- Scaffolder
![image alt ><](images/menu.png)
---

## The Catalog Page

The Catalog provides acces to all the catalog entities.

.lab[
- Click on **example-website** to browse the component information
- Note the entity relationships graph:
]

  ![image alt ><](images/relationships.png)
