
# Adding Our Own Templates

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
    - registering entities in the catalog (`catalog:register`)
    - creating and publishing a git repository (`publish:github`, `publish:gitlab`)
    - and more...
- If you want to extend the functionality of the Scaffolder, you can do so by writing *custom actions* which can be used alongside the built-in actions.
    - Currently this requires writing typescript code and rebuilding your IDP app

- All available actions for your IDP can be seen at http://your-backstage/create/actions

---

## Inspect Available Scaffolder Actions

- All available actions for your IDP can be seen at http://your-backstage/create/actions

.lab[

  - Go to http://${YOUR_LAB_DNS}:3000/create/actions

  - Explore the actions you currently have

]

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
      entityRef: ${{ steps['register'].output.entityRef }}
```

- showing Markdown text blobs: (will be available in scaffolder/next)

```yaml
output:
  text:
    - title: Some text
      content: __Entity URL:__ \`${{ steps['publish'].output.remoteUrl }}`
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

## Templating Files

The actual *templating* occurs in the `fetch:template` action. It looks at the files at its `input.url` and replaces the placeholders (by default in files with `.njk` extension, but that's modifiable) using the powerful [Nunjucks library](https://mozilla.github.io/nunjucks/) based on values from its `input.values` as shown in the following example:

---

## Templating Example
```yaml
  - id: fetch
      name: Fetch 
      action: fetch:template
      input:
        url: ./content
        values:
         name: ${{ parameters.name }}
         enabledDB: ${{ parameters.enabledDB }}
```
In ./content/somefile.njk
```javascript
{% if enabledDB %}
DB is enabled for ${{ values.name }}
{% else }
DB is disabled for ${{ values.name }}
{% endif %}
```

---

# Writing a template - Exercise

 - Let's write a real software template

 - Our plan:
    - start with a template repo
    - create a catalog-info.yaml 
    - push the resulting new repo to github
    - register the resuting new service in the catalog
---

## Create a template repo

It's usually a good idea to have one templates repo for a number of templates.
Let's create one.

.lab[
```bash
cd ~
gh repo create *myuser*/bs-templates -c \ 
   --public
cd bs-template
```
]

---

## Create Our Template Code

Let's generate a basic Expressjs app and add some templating to it:

.lab[
```bash
npx express-generator express
 # reply 'yes' at the prompt
 # and move index.js to index.js.njk - for templating
mv express/routes/index.js express/routes/index.js.njk
```
]

Now open express/routes/index.js.njk and replace:
```javascript
res.render('index', { title: 'Express' });
```
with
```javascript
res.render('index', { title: '${{ values.name }}' }} });
```

---

## Create the Template Definition

Let's now create a file named `templates.yaml` with all our template definitions:

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: expressjs-template
  title: Express.js Template
  description: Creates a an Expressjs app for the PerfectScale Workshop
```
Continues on the next slides...
---

## Define Template Parameters

```yaml
spec:
  owner: user:guest
  type: app
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
```
---
## Define Template Parameters
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
---
## Define workflow steps - fetch:template
```yaml
  steps:
    - id: fetch-template
      name: Fetch Template Repo
      action: fetch:template
      input:
        url: ./express
        #this defines that only .njk files will be templated
        templateFileExtension: true
        values:
          name: ${{ parameters.name }}
```
---

## Define workflow steps - catalog:write
```yaml
    - id: write-catalog-info
      name: Write Catalog Info
      action: catalog:write
      input:
        entity: 
          apiVersion: backstage.io/v1alpha1
          kind: Component
          metadata:
            name: ${{ parameters.name }}
            namespace: default
            description: The ${{ parameters.name }} service
          spec:
            type: app
            lifecycle: production
            owner: group:guests
``` 
---

## Define workflow steps - publish and register

```yaml
    - id: publish
      name: Publish
      action: publish:github
      input:
        allowedHosts: ['github.com']
        description: This is ${{ parameters.name }}
        repoUrl: ${{ parameters.repoUrl }}

    - id: register
      name: Register
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps['publish'].output.repoContentsUrl }}
        catalogInfoPath: '/catalog-info.yaml'
```
---
## Define template outputs

```yaml
output:
    links:
      - title: Repository
        url: ${{ steps['publish'].output.remoteUrl }}
      - title: Open in catalog
        icon: catalog
        entityRef: ${{ steps['register'].output.entityRef }}
```
---
## Exercise: Create Your Own Template for a FastApi Service

- You can start with [our fastApi template] (https://github.com/PerfectScale/python-fastapi-template)

- Store the template code in 'fastAPI' folder in your templates repo

- Add Nunjucks templating for main.py 

- Add a template definition to 'templates.yaml'

- Refresh the location

- Run the template to verify



