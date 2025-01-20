# gh-makefile

Github actions workflows for build, test, check and deployment tasks based on  Makefiles

Aligned with the targets defined in [Common Targets](https://github.com/flaviostutz/common-targets)

For a complete example on how to use it, check https://github.com/flaviostutz/aws-serverless-static-website, specifically looking at the files at .github/workflows

Another good monorepo example can be found at https://github.com/flaviostutz/aws-serverless-monorepo-demo

## Usage

* Create a file on the repository of your project at .github/workflows with the workflow definitions, and make it use the workflow from this repo

```yml
name: dev-deploy

on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    name: Deploy development environment
    uses: flaviostutz/gh-makefile/.github/workflows/make-ci.yml@main
    with:
      working-directory: ./
      tooling: node
      tooling-version: 16
      stage: dev
      target-build: true
      target-lint: true
      target-unit-tests: true
      target-deploy: true
      environment: dev
      AWS_DEFAULT_REGION: us-east-1
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

* Create a file the the name 'Makefile' at the root of your project:

```
build:
	npm ci

lint:
	npm run lint

unit-tests:
	npm run test

deploy:
	npx sls deploy --stage ${STAGE}
```

* Create secrets "AWS_ACCESS_KEY_ID" and "AWS_SECRET_ACCESS_KEY" pointing to a key with permissions to deploy resources in your AWS account

* Create a branch, push it, then create PR

* Check in tab "Actions" of this task running and see log results

### Makefile targets

The environment variable 'STAGE' is available during all target executions.

The targets, if enabled in workflow input, are executed in the following order:

* build: build project. eg. "npm ci"
* lint: run linting rules. eg. "eslint"
* unit-tests: run unit tests. eg. "npm test"
* package: create bundle for the application. eg. "npx sls package --stage ${STAGE}"
* deploy: deploy or publish application. eg. "npx sls deploy --stage ${STAGE}"
* get-environment-url: echo the environment-url that may be used later by integration tests. Environment variable 'DEPLOY_STDOUT' with the contents of the deploy output will be available
* integration-tests: run any integration or e2e tests after deployment
  * Environment variable 'ENVIRONMENT-URL' is available
* undeploy: undeploy application. eg. "npx sls remove"

### Workflow inputs

* environment: Github environment to attach this job to
* environment-url: Value to set as environment URL
* working-directory: base dir used during builds. If using monorepo, specify the relative path of the service folder. ex.: ./services/example1'
* stage: "stage" used on sls operations as parameter --stage
* tooling: Tooling environment to setup. Can be "node", "golang" or "golang+node"
* tooling-version: Tooling version to setup. eg. 1.5.2 (for golang), 14 (for node)
* target-build: Enable target 'build'
* target-lint: Enable target 'lint'
* target-unit-tests: Enable target 'unit-tests'
* target-package: Enable target 'package'
* target-deploy: Enable target 'deploy'
* target-integration-tests: Enable target 'integration-tests'
* target-undeploy: Enable target 'undeploy'
* artifact-retention: Number of days for retaining the generated artifact. Defaults to 3.
* artifact-path: Upload file or folder as workflow artifact
* git-base-head: Discover last successful git sha BASE and HEAD and expose it in variables NX_BASE AND NX_HEAD. defaults to false
* AWS_DEFAULT_REGION: The AWS region to deploy resources to

### Workflow secrets

* AWS_ACCESS_KEY_ID: AWS Access Key ID with permission to deploy resources
* AWS_SECRET_ACCESS_KEY:AWS Secret Access Key

### GH Environment URL

* This workflow will do a best effort to get a environment URL, in this order:
  * If a value is set as workflow input in 'environment-url', use it
  * If target 'get-environment-url' returns a URL, use it
  * If a URL is found in deploy stdout contents, use it

* The environment url will be exposed via environment variable "ENVIRONMENT_URL" to your tests when running integration tests

## Examples

### https://github.com/flaviostutz/aws-serverless-static-website

In this example, we have a complete lifecycle of:

* deployment of an environment when a PR is opened
* removal of the PR environment when it is closed
* deployment of the production environment when something is merged into "main" branch

## Info about creating Makefiles

* https://opensource.com/article/18/8/what-how-makefile

* https://makefiletutorial.com/

