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

This is also a great opprotunity to start playing with configuration.

---
## Configuring Backstage for Our Workshop

The values in `app-config.yaml` can be:
- hard-coded
- expanded from environment variables

Let's use both options:
.lab[
```bash
export PUBLIC_DNS=$(curl -s http://169.254.169.254/latest/meta-data/public-hostname)
```
]
---
## Configuring Backstage for Our Workshop

Now open `app-config.yaml` and edit the following lines:
.lab[
```yaml
app:
  title: StageCentral Workshop
  baseUrl: http://${PUBLIC_DNS}:3000
...
backend:
  baseUrl: http://${PUBLIC_DNS}:7007
  listen:
    port: 7007
    host: 0.0.0.0
...
  cors:
    origin: http://${PUBLIC_DNS}:3000
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

Browse to <*YOUR-MACHINE-PUBLIC-DNS*>:3000

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

---

# Adding Authentication

- Initially Backstage creates an un-authenticated IDP

- Everybody can log in

- Usually that's not what we want

- We want to only allow authenticated access

- Let's see how to do it.

---

## Authentication Providers

The authentication system in Backstage serves two distinct purposes: 
- sign-in and identification of users
- delegating access to third-party resources.

It is possible to configure Backstage to have any number of **authentication providers**, but only one of these will typically be used for sign-in, with the rest being used to provide access to external resources.

Backstage comes with many [common authentication providers](https://backstage.io/docs/auth/#built-in-authentication-providers) in the core library.

---

## Sign-in Configuration

Sign-in is configured by providing a custom `SignInPage` app component. It gets rendered before any other routes in the app and is responsible for providing the identity of the current user. 

The SignInPage can render any number of pages and components, or just blank space with logic running in the background. In the end however it must provide a valid Backstage user identity through the `onSignInSuccess` callback prop, at which point the rest of the app is rendered.

We can use the `SignInPage` component that is provided by `@backstage/core-components`, which takes either a provider or providers (array) prop of `SignInProviderConfig` definitions.

---

## Configuring Authentication with Github

Each built-in provider has a configuration block under the auth section of `app-config.yaml`. 

.lab[
In `app-config.yaml` under `auth` add : 
```yaml
auth:
  environment: development
  providers:
    github:
      development:
        clientId: ${AUTH_GITHUB_CLIENT_ID}
        clientSecret: ${AUTH_GITHUB_CLIENT_SECRET}
```
]

---

## Where Github Credentials Come From

To add GitHub authentication, you must create either a *GitHub App*, or an *OAuth App* from the GitHub developer settings. 

The *Homepage URL* should point to Backstage's frontend, while the *Authorization callback URL* will point to the auth backend.

Settings for our lab:

Application name: StageCentralWorkshop

Homepage URL: http://${YOUR_LAB_PUBLIC_DNS}:3000

Authorization callback URL: http://${YOUR_LAB_PUBLIC_DNS}:7007/api/auth/github/handler/frame

---

## Create an OAuthApp in Github

In the upper-right corner of any Github page, click your profile photo, then click **Settings**.

In the left sidebar, click  Developer settings.

In the left sidebar, click OAuth apps.

Click New OAuth App.

*Note: If you haven't created an app before, this button will say, Register a new application.*

In "Application name", type "StageCentralWorkshop"

In "Homepage URL", type "http://${YOUR_LAB_PUBLIC_DNS}:3000"

In "Authorization callback URL", type "http://${YOUR_LAB_PUBLIC_DNS}:7007/api/auth/github/handler/frame"

Click Register application. 

---

## Generate a Client Secret for Your App

- Browse to your OAuth app page on Github

- Note the "Client ID" token

- Under it -  click on "Generate a new client secret"

- Copy the new client secret

---

## Add Github Credentials to Backstage

.lab[
```bash
export AUTH_GITHUB_CLIENT_ID=your_app_client_id
export AUTH_GITHUB_CLIENT_SECRET=your_app_clent_secret
```
]

Configuration is done!!!

Now we need to add the `SignInPage` to our app

---
## In `~/backstage/packages/app/src/App.tsx`:

.lab[
```typescript
import { githubAuthApiRef } from '@backstage/core-plugin-api';
import { SignInPage } from '@backstage/core-components';
const app = createApp({
  components: {
    SignInPage: props => (
      <SignInPage
        {...props}
        auto
        provider={{
          id: 'github-auth-provider',
          title: 'GitHub',
          message: 'Sign in using GitHub',
          apiRef: githubAuthApiRef,
        }}
      />
    ),
  },
```
]

---

## Running Backstage with Github Auth

Restart your Backstage app:

Hit Control-C twice to stop the fronted and the backend.

.lab[
```bash
yarn dev
```
]

--


Now go back to http://YOUR_PUBLIC_DNS:3000 and log in with Github app credentials.

---

## Adding an actual sign-in resolver

- Great, we are now authenticated. 

- Let's check this by going to ⚙️ Settings (Bottom left corner)

- You should see your github user profile

- But 'Backstage Identity' says:
 
  -  "User Entity: user:default/guest"
  -  "Ownership Entities: user:default/guest"

- Users are Backstage catalog entities.

- In order to connect your auth provider users to Backstage user entities we need a **sign-in resolver**

---

## A sign-in resolver for Github

- Let's add a sign-in resolver for Github

- In `packages/backend/src/plugins/auth.ts` - uncomment the commented line:

```typescript
github: providers.github.create({
        signIn: {
          resolver(_, ctx) {
            const userRef = 'user:default/guest'; // Must be a full entity reference
            return ctx.issueToken({
              claims: {
                sub: userRef, // The user's own identity
                ent: [userRef], // A list of identities that the user claims ownership through
              },
            });
          },
          // resolver: providers.github.resolvers.usernameMatchingUserEntityName(),
        },
```

---

## Adding a Backstage user entity

- Let's rerun our app with `yarn dev`

- We're now getting "User not found"!

- Makes sense - our catalog only has the guest user.

- Let's add your github user to the catalog.

---

## Adding a Backstage user entity

- Open `~/backstage/examples/org.yaml`

- Add the following yaml snippet:

```yaml
apiVersion: backstage.io/v1alpha1
kind: User
metadata:
  name: <your-github-username>
spec:
  memberOf: [guests]
```

- Restart the app

- Now in "Backstage Identity" you should see: "User Entity: user:default/your-github-username"

- Now you can be the owner of catalog entities.

---
