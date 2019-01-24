# Web App

This directory contains all of the code for the web interface.

## Development

 Set the required environment variables. (see the `.envrc.example` file), and then:

```
$ yarn run dev:start
```

###### or just run:
```
$ yarn run dev:start \
  --USER_SERVICE_PROXY_TARGET=http://identity.user.endpoints.meta-cloud-183420.cloud.goog \
  --USER_SERVICE_PROXY=/user_service_proxy
```

This will build the client and server and set webpack to watch for application code changes.
Any change will re-build the bundles and reload the server. You must refresh the browser to
see the changes.

The app is served on `localhost:3000` by default.

### Pushing changes.

There are a couple of caveats that need to be addressed with the current tooling.
Currently if you run arc diff, arcanist will lint and unit test the project.

If there are node_modules install anywhere in the source tree, bazel will throw an
error and fail the tests. If the node_modules are not installedthen the flow linter
will fail.

For the time being to make changes:

`arc diff --nounit`

The unit tests are run on the CI so we need to wait for CI to complete before landing anything.

## Testing

Bazel is configured to run all unit tests with mocha. All test files must be colocated
with the corresponding js file and have the extension `.test.js`.

## Tools

### Styling:

Use [styled-components](https://www.styled-components.com/)

Note: This still needs a bit of configuration once we start to implement inside the
`universal-react-redux` library.

[see here](https://www.styled-components.com/docs/advanced#server-side-rendering).

Use [grid-styled](http://jxnblk.com/grid-styled/) for the layouts.

### Flow:

Uses [flow](https://flow.org/) for static type checking. The flow config file lives under
`/devtools/flow`. When running flow you need to provide the relative location of the
.flowconfig file.

So from the project root: `$ flow devtools/flow`.

To run through arcanist (`$arc lint` or `$arc diff`) you need to be in the project root.

#### flow-typed

Implement [flow-typed](https://github.com/flowtype/flow-typed) in future to type check
third party libraries.

## Building and loading the Docker container image locally

To build and load the Docker image in one command:

```
$ bazel run //apps/onboarding/web:onboarding_web_image
```

To only build the Docker image:

```
$ bazel build //apps/onboarding/web:onboarding_web_image.tar
```

To load the built Docker image:

```
$ docker load -i bazel-bin/apps/onboarding/web/onboarding_web_image.tar
```

## Deploying the Onboarding Web app

To use the current `kubectl` context, run:

```
$ bazel run --define="cluster=$(kubectl config current-context)" //apps/onboarding/web:deploy.apply
```

### SSL

If you need to manually respond to a [LetsEncrypt][1] challenge, place it in `challenges` and it
will be served up at `/.well-known/acme-challenge`

[1]: https://letsencrypt.org/how-it-works/
