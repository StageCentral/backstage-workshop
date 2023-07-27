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
--

- Edit your `app-config.yaml` to make your github user the default author
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

### Generated Form Fields

The parameter step in the previous slide will generate the following form:

![image alt ><](images/template-form.png)


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

Each Step is `JSONSchema` with some extra goodies for styling what it might look like in the frontend. For parameters it relies very heavily on the [react-jsonschema-form library](https://github.com/rjsf-team/react-jsonschema-form). They have some great docs and a playground where you can play around with some examples.

There's another option for that library called `uiSchema` which templates take advantage of. These are the little `ui:*` properties that you can see in the step definitions.

For all available `ui:*` properties - look [here](https://rjsf-team.github.io/react-jsonschema-form/docs/api-reference/uiSchema)

---

## Workflow Steps

After all the form fields have been filled out control goes over to the workflow steps defined by the `steps` array in template yaml:
```yaml
steps:
    # Each step runs an action. This one bring files into the workspace.
    - id: fetch-base
      name: Fetch Base
      action: fetch:template
      input:
        url: ./content
        values:
          name: ${{ parameters.name }}
    # This step publishes the contents of the working directory to GitHub.
    - id: publish
      name: Publish
      action: publish:github
      input:
        allowedHosts: ['github.com']
        repoUrl: ${{ parameters.repoUrl }}
```
---
## Referencing Parameter Values in Steps

As you may've noticed in the previous example - we can use the value received in the `parameters` steps when running our workflow.

The syntax for this is `${{ parameters.paramName }}`

Like this:

```yaml
      action: fetch:template
      input:
        url: ./content
        values:
          name: ${{ parameters.name }}
```

---

## Scaffolder Actions

- Actions are functions that can be executed in the workflow steps.

- Scaffolder comes with several built-in actions for:
    - fetching content (`fetch:plain`, `fetch:template`)
    - registering entities the catalog (`catalog:register`)
    - creating and publishing a git repository (`publish:github`, `publish:gitlab`)
    - and more...
- If you want to extend the functionality of the Scaffolder, you can do so by writing *custom actions* which can be used alongside the built-in actions.
    - Currently this requires writing typescript code and rebuilding your IDP app

- All available actions for your IDP can be seen at http://your-backstage/create/actions


---

## Scaffolder Outputs

Each step can output variables that can be shown in the frontend:

- linking to the entity and the created repo:

```yaml
output:
  links:
    - title: Repository
      url: ${{ steps['publish'].output.remoteUrl }} #link to the repo
    - title: Open in catalog
      icon: catalog
      entityRef: ${{ steps['register'].output.entityRef }
```

- showing Markdown text blobs:
```yaml
output:
  text:
    - title: Some text
      content: __Entity URL:__ `${{ steps['publish'].output.remoteUrl }}`
```
      
---

## Template Debug Actions

There are 2 built-in actions that can help us understand what's happening in the templating workflow:

`debug:log` - Writes a message into the log or lists all files in the workspace.

`debug:wait` - Waits for a certain period of time.

Let's use `debug:log` in an exercise!

---

## Let's do an exercise

- Write your own template in ~/backstage/examples/mytemplate.yaml

    - Parameters: 
        - Name: string
        - Type: string
    - Steps:
        - Log: use `debug:log` action to output the received parameters

--

- Add the `location` of your template to `app-config.yaml`

- Reload your Backstage app and test your template.

---

## Expected Output:
```
2023-07-27T13:05:43.000Z Beginning step Log
22023-07-27T13:05:43.000Z info: {
        "message": "Name is myname with type mytype",
        "listWorkspace": true
      }
62023-07-27T13:05:43.000Z Name is myname with type mytype
72023-07-27T13:05:43.000Z Workspace:82023-07-27T13:05:43.000Z Finished step Log
```
---

## Writing a real template

 - Let's write a real software template

 - Our plan:
    - start with a template repo
    - create a catalog-info.yaml 
    - push the resulting new repo to github
    - register the resuting new service in the catalog
---

## Start with a template

Use express-template


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
