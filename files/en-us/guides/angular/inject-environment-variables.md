---
title: How to inject environment variables into your Angular applications - Angular Tutorials | indepth.dev
author_name: Chihab Otmani
author_link: https://twitter.com/chihabotmani
discussion_link: https://github.com/indepth-dev/community/discussions/
---

# Introduction

Consuming environment variables in an Angular application is not natively supported. This guide describes will help you understand:

- Why using environment variables for the configuration of an Angular application is a good practice
- How to inject evnironment variables in an Angular application using a CLI custom builder

For the impatient, some ready to use solutions are listed at the end of the article.

# Configuring application environments

Angular CLI's configuration system through the environment.ts files is quite powerful. It relies on a file replacement mechanism at build time, which can be customized from the angular.json file in the following way:

```json
  "fileReplacements": [
    {
      "replace": "src/environments/environment.ts",
      "with": "src/environments/environment.**staging**.ts"
    }
  ]
```

This mechanism has the following shortcomings:

- The `environment.\*.ts` files are by the design part of your application source code that lives inside the version control. They might contain sensitive data, which is not recommended for obvious security reasons.

```ts
const environment = {
  API_KEY: "MY_SECRET_KEY", // Security issue.
};
```

- For each environment in our platform, we need to maintain the configuration files and their specific configurations within the application repository, meaning that any environment creation or configuration update would require a new commit to the repository.This can be tedious for both the project and the platform maintainers.

# Reading configuration at runtime

The usual way to get around the shortcomings above is to create configuration files as assets that will be fetched by the application at runtime from the browser.

The configuration file can thus live in the server and contain sensitive data. It can be created and updated by platform maintainers without bothering the application code maintainers.

However, this approach can be problematic because it has an impact on the initial loading time of the application. Indeed, we will have an additional resource to download which, moreover, can only be retrieved after the application bundle has been downloaded, parsed and bootstrapped. It cannot be downloaded asynchronously.

This approach can be required only if the configuration depends on some context at runtime (the user identity for example).

# Preparing the configuration at build time

A better alternative is to consume environment variables directly into the Angular application at build time as following:
`API_URL=https://dev.example.com/api npm run build`

The environment variables can explicitly be read from the command line or they can be read from the system we’re building our application in (Docker instance, CI/CD platform…)

This approach is all the more interesting when these environment variables are shared between several applications, especially in a monorepo architecture, in which case they are read from the same source.

### Consuming environment variables using `process.env`

In a Node.js application we consume environment variables using the global variable `process.env`.

```sh
API_URL=https://dev.example.com/api node -e 'console.log(process.env.API_URL)'; # https://dev.example.com/api
node -e 'console.log(process.env.HOME)';  # /home/chihab
```

What if we adopt a similar approach in our Angular Application and consume environment variables inside our TypeScript code using `process.env`?

```ts
const environment = {
  API_KEY: process.env.API_KEY,
};
```

Would that work? Do we have access to their values from an Angular Application?

Short answer, no.

When we execute a CLI command, CLI being a Node.js application, has access to all the system environment variables through `process.env`, however CLI when building our application does not provide this information to it.

Trying to access `process.env` will first generate a TypeScript error because the global process variable is unknown to TypeScript and even if we get around it, the resulting JavaScript will not contain any value because it is read at runtime from the global context (window in the browser) which does not contain the process global variable!

We wouldn't imagine Angular CLI injecting:

```js
var process = { env: {API_URL='https://dev.example.com/api'} }
```

in our JavaScript bundle code, would we?

How do we get the values of our environment variables in the JavaScript bundle executed in the browser? Webpack.

### Webpack and Define plugin

Define is a Webpack plugin that replaces any expression containing a specified string with the specified value.

```ts
new webpack.DefinePlugin({
  AngularJS: "'Angular'",
});
```

This configuration below tells Webpack through the Define Plugin to replace any `process.env.NODE_ENV` expression in our application with the value of `process.env.NODE_ENV` that Node.js runtime gets from the User environment.

```ts
const webpack = require("webpack");
exports.default = {
  plugins: [
    new webpack.DefinePlugin({
      "process.env.NODE_ENV": process.env.NODE_ENV,
    }),
  ],
};
```

But wait, why are we talking about webpack here?

## Angular CLI and webpack

Angular CLI uses webpack under the hood. Webpack roams over the application source code, looking for import statements, building a dependency graph, and emitting one or more bundles.
Since Angular CLI X.Y, it is possible to hook into the Angular build workflow through a custom builder that allows to override the webpack configuration before the execution, eventually by adding/overriding plugin and rules.

### Extending webpack configuration

To extend the webpack configuration, it would be necessary to create a custom builder. The custom builder will run our custom code then eventually executes the Angular native builder.
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

Without going into the details of implementation of a builder because it is not the purpose of this article, let's focus on the third parameter of the execution of the builder. It is a function that gets the webpack configuration then returns it. Let's add the Define plugin to the list of plugins:

```ts
{
  webpackConfiguration: async (webpackConfig: Configuration) => {
    webpackConfig.plugins.push(
      new webpack.DefinePlugin({
        "process.env.VERSION": JSON.stringify("1.0"),
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
  version = process.env.VERSION;
}
```

will be transformed inside the main.js file issued by running `VERSION=test npm run build` into:

```ts
// Ivy generated code
constructor() {
(this.version = "1.0")
}
// Ivy generated code
```

## Passing Environment Variables

In order to pass specific environment variables of the system to our application code we need the following configuration:

```ts
{
  const environment = secureEnvBeforePassingToAngular(process.env);
  return {
    webpackConfiguration: async (webpackConfig: Configuration) => {
      webpackConfig.plugins.push(
        new webpack.DefinePlugin({
          "process.env": JSON.stringify(environment),
        })
      );
      return webpackConfig;
    },
  };
}
```

When the CLI runs, it will:

- Get all the environment variables through `process.env`.
- Retrieve the desired environment keys in a secure way using our own secureEnvBeforePassingToAngular function.
- Add DefinePlugin to Angular CLI’s webpack configuration and map every key of `process.env`.X with the key stored in our secure environment object.

And that's it!

# Using existing solutions

Developing our own Angular builder can be tedious for such a simple need, an alternative would be to use:

## [@ngx-env/builder](https://github.com/chihab/ngx-env)

A builder I created while writing this article, it is based on the implementation described above, in which I added some interesting features:

- No webpack configuration needed, simply run `ng add @ngx-env/builder`.
- Overrides all CLI architect targets: `ng serve/build/test/server/extract-i18n`
- Secures environment variables by limiting them to those starting with `NG_APP` (inspired by `create-react-app` and `vue-cli`)
- Uses `dotenv` and `dotenv-expand` which allow loading environment and reference system environment variables from `.env.\*` files
- Possibility to have `.env.local`, `.env.development`, `.env.test` and `.env.production` files with a conventional hierarchy similar to the one used in `create-react-app`, `parcel` and `vue-cli`
- Supports the injection of an environment variable into a component template using the env pipe: and into `index.html`.

  ```html
  <!-- index.html -->
  <title>NgApp on %NG_APP_BRANCH_NAME%</title>

  <!-- footer.component.html -->
  'NG_APP_BRANCH_NAME' | env
  ```

## [custom-webpack](https://github.com/just-jeb/angular-builders)

A builder that allows customizing the webpack configuration through some CLI options.

## [ngx-build-plus](https://github.com/manfredsteyer/ngx-build-plus)

A custom builder that allows extending the Angular CLI's default build behavior.


# Conclusion

Perhaps one day the use of environment variables in code will be natively supported in `@angular/cli` as requested by the community in this [ong-standing issue](https://github.com/angular/angular-cli/issues/4318).

I would suggest that at least somehow only the use of `process.env` in the environment.ts file be natively replaced by the `Webpack Define` plugin. That way, our code will continue to consume the environment configuration as we do today without worrying about where the assigned values come from.

Until then, you now know how to do it yourself!

