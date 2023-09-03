# Populating the Software Catalog

The Backstage Software Catalog is a centralized system that keeps track of ownership and metadata for all the software in your ecosystem (services, websites, libraries, data pipelines, etc). 

The catalog is built around the concept of metadata YAML files stored together with the code, which are then harvested and visualized in Backstage.

---

## Initial Catalog

Our catalog already has:
- 1 component (example-website)
- 1 api
- 2 users (we added one in the previous chapter)
- 1 group
- 1 template
- 1 system
- 3 locations

---

## Locations - where catalog entities are born 

The initial locations are defined in the app-config.yaml.

They can be local folders (usually only for testing) or SCM repositories.

.lab[
```bash    
    less app-config.yaml | grep -A20 location
```
]
--

```yaml

locations:
    # Local example data, file locations are relative to the backend process, typically `packages/backend`
    - type: file
      target: ../../examples/entities.yaml

```
---

## The Entities

Let's look at the entity definitions:

.lab[
```bash
    less examples/entities.yaml
```
]

The entity descriptor format is explained in much detail
[here]( https://backstage.io/docs/features/software-catalog/descriptor-format/)

---

## Adding Our Own Components

The source of truth for the components in your software catalog are **metadata YAML files** stored (usually) in source control.

The YAML files can be called anything but the conventional name is **catalog-info.yaml**

---

## Adding Our Own Components


- There are 4 ways of adding a component to the catalog:

    - Static configuration (in app-config.yaml)

    - Manually registering in Backstage UI

    - Creating a new component in Backstage from a template

    - Integrating with an external source via a **custom entity provider**, or a **custom processor** (More on that later...)

---

## Integrating with Source Control (Github) - for Catalog

In order to retrieve data from an SCM provider service we need to authenticate with existing credentials.

Let's define this for Github.

- Wait! Haven't we already integrated with Github for authentication?  🤔  

We did! But that was to authenticate sign-in. Now we need our Backstage app to retrieve data from Github. Even when no users are signed in. 

We'll need to have a Personal Access Token with (at least) read permission for all repos we own.

Note: if all our repos are public - there's no need to define a token. Anonymous access will be used by default.

---

## Creating a PAT in Github

- In the upper-right corner of any page, click your profile photo, then click Settings.

- In the left sidebar, click  Developer settings.

- In the left sidebar, under  Personal access tokens, click Tokens (classic).

- Select Generate new token, then click Generate new token (classic).

- In the "Note" field, give your token a descriptive name.

- Select **repo** scope

- Click Generate token.

- Copy the new token to your clipboard.

---
### Adding the Token to Backstage

.lab[
```bash
export GITHUB_TOKEN=*your-github-personal-access-token*
```
]

In `app-config.yaml` we have:
```yaml
integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}
```

So basically all we need now is to restart Backstage!

- Control-C twice (to stop both frontend and backend)
- `yarn dev`

---

## Using Github Apps for Catalog Integration

As an alternative to PATs - Backstage can be configured to use GitHub Apps for backend authentication. This comes with advantages such as higher rate limits and that Backstage can act as an application instead of a user or bot account.

It also provides a much clearer and better authorization model as a opposed to the OAuth apps and their respective scopes.

See more [here](https://backstage.io/docs/integrations/github/github-apps)
---

## Adding a New Component (Service)

In order to add an entity we need to connect a git repo with a metadata *YAML* file in it.
The file is usually called `catalog-info.yaml`

Here's an example of a basic catalog info file:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: my-component
  description: a simple component
spec:
  type: service
  lifecycle: production
  owner: antweiss
```

---

## Preparing a Repository
.lab[
```bash
cd ~
gh repo create *myuser*/backster -c \ 
   -p stagecentral/python-fastapi-template \
   --public
cd backster
cat << EOF > catalog-info.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: backster
  description: a simple Backstage managed service
spec:
  type: service
  lifecycle: production
  owner: stagecentral
  system: backstage-workshop
EOF
```
]
---
## Preparing a Repository - Push

.lab[
```bash
git add catalog-info.yaml
git commit -m "Adding catalog info"
git push origin main
```
]

---
## Register the Component in Backstage UI

- In Backstage UI go to "Create"

- Click on "Register Existing Component" 

- Select URL: "https://github.com/*your_user*/backster/blob/main/catalog-info.yaml"

- Analyze

- Import

---

## Create a new Component from Template

Backstage provides the Scaffolder functionality which allows us to create software templates for engineers to get quickly started on new developemnt efforts.

We'll look into creating our own templates later.

Right now let's use the existing "Example Node.js Template" to add a new node.js repo to the catalog.

---

## Persisting Our Changes

Components created from templates are added directly to the database.

Out of the box Backstage is configured to use sqlite as its database engine. This means our changes to catalog won't be persisted between app reruns.

Let's replace sqlite with a PostgreSQL container for ( a bit more ) consistency.

Note: in production setup you'll want to use a highly-available, persistent DB solution, of course.

---

## Let's Run PostrgeSQL

.lab[
```bash
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=stagecentral postgres
```

Add this in `app-config.yaml`:
```yaml
database:
    client: pg
    connection:
      host: localhost
      port: 5432
      user: postgres
      password: stagecentral
```
And - rerun Backstage:
```bash
yarn dev
```
]

---

## Create a new Component from Template

- In Backstage UI go to "Create"

- Click "CHOOSE" on "Example Node.js Template"

- Name: mynodesvc

- Owner: your github user (the one you generated the PAT for )

- Repository: mynodesvc

- Create

After creation - the new repo gets automatically added to the Backstage catalog.

---

## Refreshing the Component Data from SCM

The entities in the catalog get automatically refreshed from their respective *Locations* every 10 minutes.

Sometimes we want to schedule the refresh for an entity to occur sooner.

This can be done in the UI

---
## Refreshing the Component Data from SCM

Let's add some metadata to the repo we created from template.

.lab[
- Go to the **mynodesvc** repo you created earlier
- In `catalog-info.yaml` add the following:

```yaml
spec:
    system: backstage-workshop
```
- Commit your changes
- Back in Backstage UI - go to **mynodesvc** and click on ⟳
- Click on "Home" - your changes should be now registered

]

---

## Unregistering an Entity

- Browse to an entity page

- Click on the three dots in the top right corner

- Choose "Unregister Entity"