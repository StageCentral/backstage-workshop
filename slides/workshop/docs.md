# Managing Tech Docs with Backstage

*TechDocs* is Spotify’s homegrown docs-like-code solution built directly into Backstage. Engineers write their documentation in Markdown files which live together with their code - and with little configuration get a nice-looking doc site in Backstage.

- Discover your Service's technical documentation from the Service's page in Backstage Catalog.

- Create documentation-only sites for any purpose by just writing Markdown.

- Take advantage of the TechDocs Addon Framework to add features on top of the base docs-like-code experience.

- Explore and take advantage of the large ecosystem of MkDocs plugins to create a rich reading experience.

- Search for and find docs.
---

## Configuring TechDocs

### Should TechDocs Backend generate docs?

In app-config.yaml:
```yaml
techdocs:
  builder: 'local'
```
Note that it's recommended generating docs on CI/CD instead. But if we want to get started quickly - set `techdocs.builder` to 'local' so that TechDocs Backend is responsible for generating documentation sites. If set to 'external', Backstage will assume that the sites are being generated on each entity's CI/CD pipeline, and are being stored in a storage somewhere.

When techdocs.builder is set to 'external', TechDocs becomes more or less a read-only experience where it serves static files from a storage containing all the generated documentation.

---

## Choosing Storage (publisher)

TechDocs needs to know where to store generated documentation sites and where to fetch the sites from. This is managed by a [Publisher](https://backstage.io/docs/features/techdocs/concepts#techdocs-publisher). 
Examples: Google Cloud Storage, Amazon S3, or local filesystem of Backstage server.

It is okay to use the local filesystem in a "basic" setup when you are trying out Backstage for the first time. At a later time, review [Using Cloud Storage](https://backstage.io/docs/features/techdocs/using-cloud-storage).

```yaml
techdocs:
  builder: 'local'
  publisher:
    type: 'local'
```
---

## Docker Requirements

By default - the TechDocs Backend plugin runs a docker container with **mkdocs** installed to generate the frontend of the docs from source files (Markdown). 

That means we either need Docker installed on Backstage server (our setup script already took care of that) or we need **mkdocs** installed locally

You can set a value in your app-config.yaml that tells the techdocs generator if it should run the local mkdocs or run it from docker. This defaults to running as docker if no config is provided.
```yaml
techdocs:
  builder: 'local'
  publisher:
    type: 'local'
  generator:
    runIn: local
```

---
# Creating and Publishing Docs

We'll now create some documentation for our service using [MkDocs](https://www.mkdocs.org/) and see how it gets represented in Backstage.

---

## Configuration
.lab[
    In the root of our `backster` repo let's add the `mkdocs.yaml` config file:
```bash
cat << EOF > mkdocs.yaml
site_name: 'backster-docs'
nav:
  - Home: index.md
plugins:
  - techdocs-core
EOF
```
    And in `catalog-info.yaml` - specify where the docs are located:
```yaml
metadata:
  annotations:
    backstage.io/techdocs-ref: dir:.
```
]

---
## Creating the Docs

Create a `/docs` folder in the root of your repository with at least an `index.md` file in it. (If you add more markdown files, make sure to update the nav in the `mkdocs.yml` file to get a proper navigation for your documentation.)

Note - Although docs is a popular directory name for storing documentation, it can be renamed to something else and can be [configured by mkdocs.yml](https://www.mkdocs.org/user-guide/configuration/#docs_dir).

In `docs/index.md`  put the following content:
```markdown
&#8203;# A great little service

This service was created at StageCentral Backstage workshop
```
Commit and push your changes.

---

## Viewing the Docs in Backstage

In Backstage UI browse to our `backster` service and click on `Docs`. Docker will get executed in the background and after a short while your documentation will appear.

![image alt](images/docs.png)


---
## Adding pages to our docs

.lab[
```bash
cd ~/backster
curl 'https://jaspervdj.be/lorem-markdownum/markdown.txt' > docs/about.md
```
and in `mkdocs.yaml`:
```yaml
nav:
  - About: about.md
```
```bash
git add . && git commit -m "About"
git push origin main
```
Refresh your component in Backstage and wait for the docs to be rebuilt.
]

For more detail on how to build great tech docs consult the [MkDocs website](https://mkdocs.org) 




