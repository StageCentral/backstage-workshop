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

---

## More on Template Parameters

Parameters are template variables which can be modified in the frontend as a sequence. Example:
```yaml
parameters:
  - title: Ask for name
    properties:
      name:
        title: Name
        type: string
        description: Unique name of the component
```

Each `properties` entry represents one field in the input form.

All properties can be in one `parameters` Step if you just want one big list of different fields in the frontend, or it can be broken up into multiple different steps which would be rendered as different steps in the scaffolder plugin frontend.

---

### Defining mandatory input values

Some values may be defined as mandatory by setting them in the `required` property of the parameters step:

```yaml
- title: Ask for name and password
    required:
      - name
      - password
    properties:
      name:
        title: Name
        type: string
      passsword:
        title: Password
        type: string
        ui:widget: password #will print '******' as value for property 'password'
```

---

## The Repository Picker

This is a custom field type which makes it easy to select a repository provider, and insert a project or owner, and repository name:

```yaml
- title: Choose a location
  required:
    - repoUrl
  properties:
    repoUrl:
      title: Repository Location
      type: string
      ui:field: RepoUrlPicker
      ui:options:
        allowedHosts:
          - github.com
```

The `allowedHosts` part should be set to where you wish to enable this template to publish to. And it can be any host that is listed in your integrations config in app-config.yaml.

---

## Limiting repo choice for RepoUrlPicker

`allowedOwners` - only publish to repos owned by specific users

`allowedRepo` - to a specific set of repository names:

```yaml
- title: Choose a location
  required:
    - repoUrl
  properties:
    repoUrl:
      title: Repository Location
      type: string
      ui:field: RepoUrlPicker
      ui:options:
        allowedHosts:
          - github.com
        allowedOwners:
          - backstage
          - someGithubUser
        allowedRepos:
          - backstage
```
---

## More Pickers

- Beside `RepoUrlPicker` there are also the `EntityPicker` and `OwnerPicker`

- For all UI options for these Backstage field pickers see [here](https://backstage.io/docs/features/software-templates/ui-options-examples/)


---


## Understanding the Template Schema

Each Step is `JSONSchema` with some extra goodies for styling what it might look like in the frontend. For parameters it relies very heavily on this [library](https://github.com/rjsf-team/react-jsonschema-form). They have some great docs and a playground where you can play around with some examples.

There's another option for that library called `uiSchema` which templates take advantage of. These are the little `ui:*` properties that you can see in the step definitions.

For all available `ui:*` properties - look [here](https://rjsf-team.github.io/react-jsonschema-form/docs/api-reference/uiSchema)

---



kind: Template
metadata:
  name: example-nodejs-template
  title: Example Node.js Template
  description: An example template for the scaffolder that creates a simple Node.js service
spec:
  owner: user:guest
  type: service

  # These parameters are used to generate the input form in the frontend, and are
  # used to gather input data for the execution of the template.
  parameters:
    - title: Fill in some steps
      required:
        - name
      properties:
        name:
          title: Name
          type: string
          description: Unique name of the component
          ui:autofocus: true
          ui:options:
            rows: 5
    - title: Choose a location
      required:
        - repoUrl
      properties:
        repoUrl:
          title: Repository Location
          type: string
          ui:field: RepoUrlPicker
          ui:options:
            allowedHosts:
              - github.com
  steps:
    - id: fetch-base
      name: Fetch Base
      action: fetch:plain
      input:
        url: ./content
        target-path: ${{ parameters.name }}
    - id: wait
      name: Wait
      action: debug:wait
      input:
        minutes: 2
