# monorepo-pravvi
A monorepo application written in Golang, CDK... and a little bit of that stuff ;)

![Screenshot (624)](https://github.com/mcgigglepop/codeforge/blob/main/resources/Web%20App%20Reference%20Architecture.png)

# Tech Stack - `Golang`, `RDS`

Bio tbd  
Readme tbd
  
this is just boilerplate...

<br />

> Roadmap & Features

| Status | Item | info | 
| --- | --- | --- |
| ✅ | **Version Control** | `github` |
| ❌ | **-** | - |
| ❌ | **-** | - |
| ❌ | **-** | - |

<br />

## Running a Demo  
Navigate to Demo folder  
```cd demo``` 
  
Compile and run the demo  
```go run ./cmd/web```

## Technology Stack

### Installation Prerequisites

- [NodeJs](https://nodejs.org/en/) is the platform that runs the Backend and builds the Frontend
  - [NPM](https://www.npmjs.com/) is the most popular package manager for the NodeJs ecosystem; to make use of the workspace monorepo architecture this project is built on, make sure you have at least version 7 of NPM installed
- Java JRE is needed by the OpenAPI generator; if you don't want to install Java, you can still use the generator, via the [Docker image](https://openapi-generator.tech/docs/installation#docker):

```
docker run --rm \
  -v ${PWD}:/local openapitools/openapi-generator-cli generate \
  -i /local/api-bundle.yaml \
  -g typescript-fetch \
  -o /local/packages/generated
  --additional-properties=typescriptThreePlus=true,withInterfaces=true
```

then compile using:

```
tsc --project tsconfig.openapi.json 
```

<br />

### Backend

- [Express](https://expressjs.com/) is the most popular NodeJs web framework; the following commonly used plugins have been included:
  - [helmet](https://github.com/helmetjs/helmet) is a utility for setting HTTP headers
  - [morgan](https://expressjs.com/en/resources/middleware/morgan.html) is a logging utility
  - [cors](http://expressjs.com/en/resources/middleware/cors.html) is a utility for configuring cross origin request security
  - [express-openapi-validator](https://github.com/cdimascio/express-openapi-validator) is a utility for validating API requests against the OpenAPI specification
- [TypeORM](https://typeorm.io/#/) is an ORM that supports the `DataMapper` pattern, which makes is more attractive in combination with the "API first" approach
- [Express JWT](https://github.com/auth0/express-jwt) and [JWKS-RSA](https://github.com/auth0/node-jwks-rsa) are two utilities for verifying a JWT token authenticity, in an OAuth2 / OpenID connect context

<br />

### Frontend

- [Create React App](https://reactjs.org/docs/create-a-new-react-app.html#create-react-app) is the most popular way to start with React; the Seed provides three UI alternatives:
  - [Material UI](https://mui.com/)
  - [Chakra UI](https://chakra-ui.com/)
  - [React Bootstrap](https://react-bootstrap.github.io/)
  - [Ant Design](https://ant.design/)
- [React Redux](https://react-redux.js.org/) is a state container using the immutable state / action / reducer pattern
- [Recharts](https://recharts.org/en-US/) is a charts library built with React and D3
- [React I18Next](https://react.i18next.com/) is a popular internationalization framework for React
- [OpenID AppAuth-Js](https://github.com/openid/AppAuth-JS) is an OAuth2 / OpenID connect flow library 

Both Backend and Frontend use [Typescript](https://www.typescriptlang.org/).

<br />

## Project anatomy

The project consists of 3 major components:

- the __OpenAPI specification__
- the __Backend__
- the __Frontend__

<br />

### The OpenAPI specification

This is found in the [spec](./spec) folder.

The up-to-date OpenAPI specification can be found [here](https://swagger.io/specification/).

To improve maintainability, the spec is not provided as a single file, but broken into smaller pieces (read more about this [here](https://davidgarcia.dev/posts/how-to-split-open-api-spec-into-multiple-files/)).

To assemble the API bundle, run this in the project root folder:

```
npm run generate:spec
```

This will combine all the smaller files into a single `api-bundle.yaml`. This file is in `.gitignore`, because we want to always have a single "source of truth" published.

To generate Typescript code from the OpenAPI spec, run the following command in the project root folder:

```
npm run generate:api
```

This will create a folder called `generated` in `packages/frontend`, based on the [typescript-fetch](https://github.com/OpenAPITools/openapi-generator/blob/master/docs/generators/typescript-fetch.md) generator. Note that this folder is also present in `.gitignore`, because the intention is to create it every time, during the CI/CD cycles.

The generated code will include:

- Models
- API Interfaces
- API Client implementation using the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)

The Frontend will use these directly.

The Backend will use them as follows:

- The TypeORM Entity classes will implement the OpenAPI Models
- The Services can implement the API interfaces (not possible yet due to [this](https://github.com/OpenAPITools/openapi-generator/issues/10237) bug)

This will allow both the Frontend and the Backend to fail fast as soon as the API specification is changed.

<br />

#### Note

Upgrade OpenAPI generator version in [openapitools.json](./openapitools.json) when [this bug](https://github.com/OpenAPITools/openapi-generator/issues/10164) gets resolved.

<br />

### The Backend

The application setup, with the various middleware, the routes, as well as error handling, can be found in [index.ts](./packages/backend/src/index.ts).

The ORM section, including Entities, Repositories and Migrations, can be found in [data](./packages/backend/src/data).

The controllers are in [controller](./packages/backend/src/controller).

There's also a [middleware](./packages/backend/src/middleware) folder, which contains any custom middleware you need to add (currently it only contains the Authorization middleware that verifies the Access Token received cryptographically).

The Backend configuration is in the [.env](./packages/backend/src/.env) file. 

This seed is configured with a SQLite persistence.

<br />

### The Frontend

In addition to the standard CRA output, the Frontend is organized as follows:

Each feature has a folder inside the [features](./packages/frontend/src/features) folder; each feature folder should contain a React component (`.tsx`), a Redux slice, with the actions, reducers and side effects, and a test file. 

Note that Redux is only necessary for "smart" components, that hold and manipulate a state. "Dumb" components, like [nav](./packages/frontend/src/features/nav) don't need it.

Don't forget to add your Redux slice in the [store](./packages/frontend/src/app/store.ts) to make it functional.

The Frontend also provides a [ui](./packages/frontend/ui) folder, that contains the components drawn using the three UI alternatives.

To select one of them, use the corresponding script, in the `packages/frontend` folder:

```
npm run select-ui:mui
npm run select-ui:chakra
npm run select-ui:bootstrap
npm run select-ui:antd
```

This will effectively replace the necessary components, as well as the theme, with the ones from the selected UI.

Feel free to clean this folder once you have decided upon a UI provider (don't forget to also cleanup the [package.json](./packages/frontend/package.json), removing unnecessary dependencies).

The Frontend configuration is in the [.env](./packages/frontend/src/.env) file. 

<br />

#### Authentication

A special note for the authentication flow: this project uses the [Authentication Code flow with PKCE verification](https://auth0.com/docs/authorization/flows/authorization-code-flow-with-proof-key-for-code-exchange-pkce). The implementation is in [authService.ts](./packages/frontend/src/features/auth/authService.ts) and the routes can be protected using the [GuardedRoute.tsx](./packages/frontend/src/GuardedRoute.tsx) wrapper component.

The project is configured with a demo Auth0 client.

Modify the backend [.env](./packages/backend/.env) and frontend [.env](./packages/frontend/.env) to contain your IDP configuration.

Here are a few major providers of OpenID Connect / OAuth2 platforms (IDPs):

- Microsoft - https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-protocols-oidc
- Okta - https://developer.okta.com/docs/guides/implement-grant-type/authcodepkce/main/#next-steps
- Auth0 - https://auth0.com/docs/get-started

For the Authentication Code flow, you'll need the following items:
- the `issuer uri`, which is a URL provided by the IDP for your tenant
- the `client id`, which is an id associated with a certain application where the user performs the authentication (i.e. on the same tenant, you can connect multiple applications and users can authenticate for any of these applications with the same credentials / SSO)
- the `audience`, which is the id associated with a certain application that receives and verifies the Access Tokens (i.e. your backend in this case)
- the `scopes`, i.e. the privileges you can assign to various users (e.g. `write_access`, `admin_access`, etc.)
- the `jwks uri`, which is a public resource associated with your tenant, containing the JSON Web Keys (public keys) that can be used to verify the signatures on your JWTs; 

Note that all OpenID providers have a `.well-known` path containing many of the information you require, once you are onboarded (e.g. https://node-seed.eu.auth0.com/.well-known/openid-configuration)

If you want to test the solution locally, you can also use Keycloak, the most popular open source IDP - for example using their [Docker guide](https://www.keycloak.org/getting-started/getting-started-docker).

<br />

---
Full-Stack Generator - `Express` & `React`, free starter provided by [AppSeed](https://appseed.us/)