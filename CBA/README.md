# Certified Backstage Associate

_This exam is an online, proctored, multiple-choice exam._

## Resources

## Topics

<details>
  <summary>Backstage Development Workflow</summary>

* Build and run Backstage projects locally
* Understand local development workflows
* Compile a Backstage project with TypeScript
* Download and install dependencies for a Backstage project with NPM/Yarn
* Use Docker to build a container image of a Backstage project

What is Backstage? It's an open source _framework_ for building developer _portals_. Powered by a centralized software catalog. Backstage unifies all your infrastructure tooling, services and documentation to create a streamlined development environment from end to end.

Backstage includes:

* Software catalog - for managing all your software
* Software templates - for spinning up new projects and standardizing your tooling
* TechDocs - making it easy to create, maintain and find and use technical documentation.

## Build and run Backstage projects locally

To run and build Backstage projects locally you need:

* Node.js
  * Using `nvm`
* `yarn`:

  ```
  corepack enable
  yarn set version 4.4.1
  ```

* Docker
* `git`

1. Run `npx @backstage/create-app@latest` to bootstrap and create your first backstage app.
2. `cd my-backstage-app` and run `yarn start`

Then it's time to use the Backstage app:

* Login
* Register a component
* Create a new component

Dictionary:

* `yarn` is a dependency mangament tool for JavaScript. Acts as an alternative for `npm` - Node Package Manager.
* `npm` is the node package manager.
* `npx` is a tool for executing Node.js package executables without having to install them globally.

## Local development workflows

_Note that i started of this exam prep on Arch Linux._

```
yay -S yarn npm nodejs-lts-jod (v22 as recommended by Backstage)
npx @backstage/create-app@latest
yarn start # in your app directory
```

Follow the guides here: <https://backstage.io/docs/getting-started/#admin> after setting up your first

## Use Docker to build a container image of a Backstage project

### Host build

1. Install dependencies with `yarn install`
2. Generate type definitions with `yarn tsc`, complies the current project. `tsc` stands for TypeScript compiler. Compiles all your TypeScript files into JavaScript files. Standard way to type-check and compile your TypeScript code as part of adevelopment or build process.
3. `yarn build:backend`, actually a script defined in `package.json` and executes: `yarn workspace backend build`. Packages everything up and bundles it into the `packages/backend/dist` directory.

From the root of the repo execute:

```
docker image build . -f packages/backend/Dockerfile --tag backstage
```

## The Backstage CLI

The Backstage CLI

## Deploy Backstage to Kubernetes for local testing and development

1. Create a `kind` cluster:

```
kind create cluster --config files/backstage-cluster.yaml
```

2. Follow this guide to install an ingress controller: <https://kind.sigs.k8s.io/docs/user/ingress>.

3. Install Backstage using `helm`:

```
helm repo add backstage https://backstage.github.io/charts
```

4. Install Backstage using `helm`:

```
helm upgrade --install \
  backstage backstage/backstage \
  --create-namespace \
  --namespace backstage \
  -f files/backstage-values.yaml \
  --version 2.6.1
```

</details>

<details>
  <summary>Backstage Infrastructure</summary>

* Understand the Backstage framework
* Configure Backstage
* Deploy Backstage to production
* Understand Backstage client-server architecture

</details>

<details>
  <summary>Backstage Catalog</summary>

* Understand how/why to use Backstage Catalog
* Populate Backstage Catalog
* Using annotations
* Working with manually registered entity locations
* Troubleshooting entity ingestion
* Working with automated ingestion

</details>

<details>
  <summary>Customizing Backstage</summary>

* Understand frontend versus backend plugins
* Customizing Backstage plugins
* Make changes to React code in Backstage App
* Using Material UI components

</details>
