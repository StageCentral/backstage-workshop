# The Scaffolder

The Scaffolder (or Software Templates) part of Backstage is a tool that allows to create Components for the Software Catalog.

By default, it has the ability to:
- load skeletons of code
- template in some variables
- publish the template to some locations like GitHub or GitLab

---

## Configuring the Templates

Scaffolder creates source code, so your Backstage application needs to be set up to allow repository creation.

This is done in `app-config.yaml` by adding Backstage integrations for the appropriate source code repository for your organization.

We've already added Github credentials for this workshop

---

## Publishing defaults

Software templates can define publish actions, such as `publish:github` - to create new repositories or submit pull /merge requests to existing repositories. You can configure the author and commit message through the scaffolder configuration in app-config.yaml:
```yaml
scaffolder:
  defaultAuthor:
    name: Rick Astley # Defaults to `Scaffolder`
    email: ricka@noletdown.com # Defaults to `scaffolder@backstage.io`
  defaultCommitMessage: "Never Gonna Give U Up" # Defaults to 'Initial commit'
```
---

## Adding Our Own Templates

Software templates are stored in the Software Catalog under the *Template* kind.

What you need for a template are:

- A `template.yaml` file

- Some files/folders to template from. (Optional)

---

## The Structure of a Template

Actually *Template* is a bit of a misnomer. In reality a `template.yaml` file defines a parameterized workflow that can optionally do some templating.

Basic parts of a `Template` entity:

- metadata - common to all catalog entities

- **spec.type** [required] - the type of entity created by the template, e.g. website. This is used for filtering templates.

- **spec.parameters** [required] - fields that are rendered in the frontend input form

- **spec.steps** [required] - The steps that are executed by the scaffolder in order to create a component from a template

- **spec.output** - displayed to the user after a successful execution

Here's the example Node.js [template](https://github.com/backstage/backstage/blob/master/packages/create-app/templates/default-app/examples/template/template.yaml)