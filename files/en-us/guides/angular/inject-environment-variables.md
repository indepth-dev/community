---
title: How to inject environment variables into your Angular applications - Angular Tutorials | indepth.dev
author_name: Chihab Otmani
author_link: https://twitter.com/chihabotmani
discussion_link: https://github.com/indepth-dev/community/discussions/
---

## Introduction

Angular's configuration system through the `environment.ts` files is quite powerful. However, the values set in `environment.ts` are set statically and cannot be changed at build time, this can be problematic because they might be sensitive and should live outside version conrol or they might depend on the environment in which the application is deployed.

## Reading configuration at runtime

The usual way to get around this problem is to create/generate a configuration file as an asset that will be fetched by the application at runtime in the browser.

This approach can be problematic as it obviously impacts the LCP of the application (initial load time) as the application will only be visible when Angular is loaded, the JS parsed and executed and then the configuration file retrieved from the server as an asset.

It can be required only if the configuration depends on some context at runtime (the user for example).

## Preparing the configuration at build time

A better alternative is to inject envrionment variables directly into the Angular application at build time. 

This can be achieved using the webpack `Define` plugin that replaces any expression containing a specified string with the specified value.

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
mkdir src && code src/index.js
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

## Angular CLI and webpack

Angular CLI uses webpack under the hood. Webpack roams over the application source code, looking for import statements, building a dependency graph, and emitting one or more bundles.

Since Angular CLI X.Y, it is possible to hook into the Angular build workflow through a custom builder that allows to override the webpack configuration before the execution, eventually by adding/overriding plugin and rules.

To extend the webpack configuration, it would be necessary to create a custom builder. The custom will run our custom code then eventually execute the Angular native builder.

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

Without going into the details of implementation of a builder because it is not the purpose of this article, let's focus on the third parameter of the execution of the builder. It is a function that gets the webpack configuration then returns it. Let's add the `Define` plugin to the list of plugins:

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

The component below:

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

## Passing Environment Variables

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

Developing our own Angular builder can be tedious for such a simple need, an alternative would be to use:

### custom-webpack

A builder that allows customizing the webpack configuration through some CLI options.

### ngx-build-plus

An custom builder that allows extending the Angular CLI's default build behavior.

### @ngx-env/builder

A builder I created while writing this article, based on the implementation described below in which I added some features that might be interesting for you:
- No webpack configuration, simply run `ng add @ngx-env/builder`.
- Override of all CLI architect targets: `ng serve/build/test/server/extract-i18n`
- Securing environment variables by limiting them to those starting with `NG_APP` (inspired by create-react-app and vue-cli)
- Use of `dotenv` and `dotenv-expand` which allows to load environment and reference system environment variables from `.env.*` files
- Possibility to have `.env.local`, `.env.development`, `.env.test` and `.env.production` files with a conventional hierarchy similarily used in create-react-app, parcel and vue-cli
- Injection of an environment variables directly into the template using the `env` pipe: `'NODE_ENV' | env`
- Injection of an environment variable in the index.html `<title>NgApp on %NG_APP_BRANCH_NAME%</title>`

## Conclusion

Maybe one day the use of environment variables in code will be natively supported in `@angular/cli` as requsted by the community in this long-standing [issue](link_here). Until then, you know how to do it yourself!
