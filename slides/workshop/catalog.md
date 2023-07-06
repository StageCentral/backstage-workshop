# Populating the Software Catalog
---
## The Catalog 

The Backstage Software Catalog is a centralized system that keeps track of ownership and metadata for all the software in your ecosystem (services, websites, libraries, data pipelines, etc). The catalog is built around the concept of metadata YAML files stored together with the code, which are then harvested and visualized in Backstage.

---

## Initial Catalog

Our catalog already has:
- 1 component (example-website)
- 1 api
- 1 user
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
    
    - Integrating with an external source via a **custom entity provider**, or a **custom processor** 

---

## Integrating with Source Control (Github) - for Catalog

In order to retrieve data from an SCM provider service we need to authenticate with existing credentials.

Let's define this for Github.

We'll need to have a Personal Access Token with at least reading permission for all repos we own.

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
