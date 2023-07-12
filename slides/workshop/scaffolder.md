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
    name: M.C. Hammer # Defaults to `Scaffolder`
    email: hammer@donthurtem.com # Defaults to `scaffolder@backstage.io`
  defaultCommitMessage: "U can't touch this" # Defaults to 'Initial commit'
```