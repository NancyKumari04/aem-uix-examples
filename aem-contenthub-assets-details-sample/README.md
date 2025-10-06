# SampleApp

Welcome to my Adobe I/O Application!

## Setup

- Populate the `.env` file in the project root and fill it as shown [below](#env)

## Local Dev

- `aio app run` to start your local Dev server
- App will run on `localhost:9080` by default

By default the UI will be served locally but actions will be deployed and served from Adobe I/O Runtime. To start a
local serverless stack and also run your actions locally use the `aio app run --local` option.

## Test & Coverage

- Run `aio app test` to run unit tests for ui and actions
- Run `aio app test --e2e` to run e2e tests

## Deploy & Cleanup

- `aio app deploy` to build and deploy all actions on Runtime and static files to CDN
- `aio app undeploy` to undeploy the app

## Config

### `.env`

You can generate this file using the command `aio app use`. 

```bash
# This file must **not** be committed to source control

## please provide your Adobe I/O Runtime credentials
# AIO_RUNTIME_AUTH=
# AIO_RUNTIME_NAMESPACE=
```

### `app.config.yaml`

- Main configuration file that defines an application's implementation. 
- More information on this file, application configuration, and extension configuration 
  can be found [here](https://developer.adobe.com/app-builder/docs/guides/appbuilder-configuration/#appconfigyaml)

#### Action Dependencies

- You have two options to resolve your actions' dependencies:

  1. **Packaged action file**: Add your action's dependencies to the root
   `package.json` and install them using `npm install`. Then set the `function`
   field in `app.config.yaml` to point to the **entry file** of your action
   folder. We will use `webpack` to package your code and dependencies into a
   single minified js file. The action will then be deployed as a single file.
   Use this method if you want to reduce the size of your actions.

  2. **Zipped action folder**: In the folder containing the action code add a
     `package.json` with the action's dependencies. Then set the `function`
     field in `app.config.yaml` to point to the **folder** of that action. We will
     install the required dependencies within that directory and zip the folder
     before deploying it as a zipped action. Use this method if you want to keep
     your action's dependencies separated.

## Debugging in VS Code

While running your local server (`aio app run`), both UI and actions can be debugged, to do so open the vscode debugger
and select the debugging configuration called `WebAndActions`.
Alternatively, there are also debug configs for only UI and each separate action.

## Typescript support for UI

To use typescript use `.tsx` extension for react components and add a `tsconfig.json` 
and make sure you have the below config added
```
 {
  "compilerOptions": {
      "jsx": "react"
    }
  } 
```

## Workfront Integration

This sample adds a Content Hub side panel that can create a Workfront task and link the currently selected asset to that task.

- **UI panel**: `src/aem-contenthub-assets-details-1/web-src/src/components/PanelWorkfrontExtensionTab.js`
- **Backend action**: `src/aem-contenthub-assets-details-1/actions/generic/index.js`

### Prerequisites

- **Adobe App Builder project** with Runtime enabled.
- **Adobe IMS integration** with Workfront scopes (metascopes) enabled.
- **Workfront API access** and base URL for your Workfront instance.
- **AEM Content Hub repo host** you will allow this extension to run on.

### Configure allowed Content Hub repo host

Edit `src/aem-contenthub-assets-details-1/web-src/src/components/ExtensionRegistration.js` and update `allowedRepos` to include your Content Hub domain.

### Required environment variables

These are read by the action (see `ext.config.yaml -> runtimeManifest.packages.aem-contenthub-assets-details-1.actions.generic.inputs`). Provide them via `.env` so `aio` can inject them at build/deploy time.

```bash
# Logging
LOG_LEVEL=info

# IMS / JWT integration (used to obtain a Workfront access token via IMS)
IMS_ENDPOINT=ims-na1.adobelogin.com
METASCOPES=<comma-separated Workfront metascopes>
TECHNICAL_ACCOUNT_CLIENT_ID=<client_id>
TECHNICAL_ACCOUNT_CLIENT_SECRET=<client_secret>
TECHNICAL_ACCOUNT_EMAIL=<tech_account_email>
TECHNICAL_ACCOUNT_ID=<tech_account_id>
ORGANIZATION_ID=<org_id>
PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
PUBLIC_KEY="-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----\n"
CERTIFICATE_EXPIRATION_DATE=<optional>

# Workfront configuration
WORKFRONT_BASE_URL=https://<your-workfront-domain>.my.workfront.com/attask/api/v15.0
AEM_AUTHOR=<your-aem-author-host>
DOCUMENT_PROVIDER_ID=<provider_id_configured_in_workfront>
DEFAULT_PROJECT_ID=<target_project_id_for_new_tasks>

# Runtime namespace (used by UI to call your action endpoint)
AIO_runtime_namespace=<your_runtime_namespace>
```

Notes:
- The UI computes the action URL using `AIO_runtime_namespace` and the package/action path: `https://<AIO_runtime_namespace>.adobeio-static.net/api/v1/web/aem-contenthub-assets-details-1/generic`.
- Ensure your Workfront API version in `WORKFRONT_BASE_URL` matches your tenant (the example uses `v15.0`).

### How it works

1. The UI tab gathers Task Name/Description and calls the web action with `action: "createTaskAndLinkAsset"` and current asset id.
2. The action exchanges a JWT for an IMS access token using your integration.
3. It creates a Workfront task (project = `DEFAULT_PROJECT_ID`).
4. It links the current asset to that task using `DOCUMENT_PROVIDER_ID` and `AEM_AUTHOR`.

### Run locally

- `aio app run`
- Open Content Hub with this extension installed and navigate to an asset. Use the "Workfront Extension Tab".

### Deploy

- `aio app deploy`

### Troubleshooting

- **Missing env vars**: The action will return 500 with a message listing missing configuration elements.
- **401/403 from Workfront**: Verify `METASCOPES`, IMS integration credentials, and `WORKFRONT_BASE_URL`.
- **Unknown action**: Ensure the UI sends `action: createTaskAndLinkAsset` and your action is deployed.
- **Panel hidden**: Make sure your repo host is in `allowedRepos` in `ExtensionRegistration.js`.
