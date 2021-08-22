---
title: How to inject environment variables into your Angular applications - Angular Tutorials | indepth.dev
author_name: Chihab Otmani
author_link: https://twitter.com/chihabotmani
discussion_link: https://github.com/indepth-dev/community/discussions/
---

## Introduction

Angular's configuration system through the environment.ts files is quite powerful. However, it does not cover the use case of determining the values of certain parameters of the application at build time because they may be sensitive or because they depend on the environment in which the application is deployed.

## Reading configuration at runtime

The classic way to get around this problem is to create/generate a configuration file as an asset that will be fetched by the application, at runtime in the browser, once it is loaded.

This approach can be problematic as it obviously impacts the LCP of the application (initial load time) as the application will only be visible when Angular is loaded, the JS executed and then the JSON file retrieved from the server as an asset to finally load the components based on the retrieved configuration.

It can be required only if the configuration depends on some context at runtime (the user for example).

## Preparing the configuration at build time

The solution proposed in this article is to inject envrionment variables directly into the Angular application at build time using the webpack `Define` plugin. The webpack `Define` plugin allows to replace any expression containing a specified string with the specified value.

```js
new webpack.DefinePlugin({
  "AngularJS": "'Angular'",
}),
```

Let's first look at how this plugin works independently of an Angular application.

```sh
mkdir webpack-project && cd _
yarn init -y
yarn add -D webpack webpack-cli
mkdir src && code index.js
```

```js
console.log(process.env.NODE_ENV);
```

Running `NODE_ENV=test yarn webpack` will emit a main.js file that contains

```js
console.log(process.env.NODE_ENV);
```

Now let's add the webpack configuration for the `Define` Plugin:

```js
const webpack = require("webpack");

exports.default = {
  plugins: [
    new webpack.DefinePlugin({
      "process.env.NODE_ENV": "'test'",
    }),
  ],
};
```

This configuration tells Webpack through the `Define` Plugin to replace any `process.env.NODE_ENV` expression with the value `test`

`NODE_ENV=test yarn webpack` will now produce a main.js

```js
console.log("test");
```

We can conclude that the replacement is done statically at build/bundling time. The same approach is to be adopted in Angular CLI.

# Angular CLI and webpack

Angular CLI uses webpack under the hood. Webpack roams over your application source code, looking for import statements, building a dependency graph, and emitting one or more bundles.

Since Angular CLI X.Y, it is possible to hook into the Angular build workflow through a custom builder that allows you to override the webpack configuration before execution by adding/overriding plugin and rules.

To extend the webpack configuration, it would be necessary to create a specific builder that will execute the Angular builder downstream after having performed a specific processing.

Here is the signature of the Angular builder to invoke from our custom builder.

```ts
export declare function buildWebpackBrowser(
  options: BrowserBuilderSchema,
  context: BuilderContext,
  transforms: {
    webpackConfiguration: ExecutionTransformer<webpack.Configuration>;
    logging: WebpackLoggingCallback;
    indexHtml: IndexHtmlTransform;
  }
): Observable<BrowserBuilderOutput>;
```

Without going into the details of implementation of a builder because it is not the purpose of this article, let's focus on the third parameter of the execution of the builder, it can take the following value:

```ts
{
  webpackConfiguration: async (webpackConfig: Configuration) => {
    webpackConfig.plugins.push(
      new webpack.DefinePlugin({
          "process.env.NODE_ENV": JSON.stringify('test'),
      })
    );
    return webpackConfig;
  },
};
```

By applying our builder to an Angular Architect target in the `angular.json` configuration, we will have an equivalent execution of the native Angular builder augmented with the prepared webpack configuration.

Thus the component below:

```ts
@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.css"],
})
export class AppComponent {
  version = process.env.NODE_ENV;
}
```

will be transformed inside the `main.js` file issued by runing `NODE_ENV=test npm run build` into:

```ts
// Ivy Code
constructor() {
  (this.env = "test")
}
// Ivy Code
```

##

In order to pass specific environment variables of the system to our application code we need the following configuration:

```ts
{
  const environment = secureEnvBeforePassingToAngular(process.env);
  webpackConfiguration: async (webpackConfig: Configuration) => {
    webpackConfig.plugins.push(
      new webpack.DefinePlugin({
        "process.env": JSON.stringify(environment),
      })
    );
    return webpackConfig;
  },
};
```

When the CLI runs, the Node runtime will pass all the environment variables in process.env to our code. Then we are free to retrieve the desired environment keys in a secure way in our `secureEnvBeforePassingToAngular` function.

And that's it!

## Using existing solutions

Developing our own Angular builder can be heavy for such a simple need, an alternative would be to use:

### custom-webpack:

Allows customizing build configuration without ejecting webpack configuration

### ngx-build-plus:

Extends the Angular CLI's default build behavior without ejecting

### @ngx-env/builder:

A builder I created while writing this article, based on the implementation described below in which I added some features that might be interesting for you:

- Override of all CLI architect targets: `ng serve/build/test/server/extract-i18n`
- Securing environment variables by limiting them to those starting with `NG_APP` (inspired by create-react-app and vue-cli)
- Use of `dotenv` and `dotenv-expand` which allows to load environment and reference system environment variables from the .env file
- Possibility to have `.env.local`, `.env.development`, `.env.test` and `.env.production` files with a conventional hierarchy namely used in create-react-app parcel and vue-cli
- Injection of an environment variable directly into the template using the `env` pipe: `'NODE_ENV' | env`.
- Injection of an environment variable in the index.html `<title>NgApp on %NG_APP_BRANCH_NAME%</title>`

## Conclusion

Maybe one day the use of environment variables in code will be natively supported in `@angular/cli` as requsted by the community in this long-standing [issue](link_here). Until then, you know how to do it yourself!
