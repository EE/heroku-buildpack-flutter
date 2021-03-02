# Heroku Buildpack Flutter Light 🪶
A configurable Heroku Buildpack with a light footprint. Designed to be easily paired with other Buildpacks and integrated into existing projects.

## Why Heroku Buildpack Flutter Light?
This Buildpack was originally forked from Diego Zepeda's [Heroku Buildpack Flutter](https://github.com/diezep/heroku-buildpack-flutter). After testing, we realized that we needed to make different architectural tradeoffs than the original package. 

The scope of our *Heroku Buildpack Flutter Light* is smaller. Unlike the original package, it is not concerned with serving the compiled files. It tests and compiles them but leaves serving them via HTTP to other Buildpacks that are better suited for that purpose (that includes a very straightforward integration with Heroku's [heroku-buildpack-static](https://github.com/heroku/heroku-buildpack-static)).

This means that the Buildpack is able to deploy only the compiled files, stripping Flutter SDK, Dart SDK, and all other dependencies. That in turn makes the deploys much faster and less likely to hit [Heroku's Slug size limit of 500 MB](https://devcenter.heroku.com/articles/slug-compiler#slug-size) (in our tests we went from being above the limit to having slugs smaller than 10 MB).

Other key improvements include the [Heroku CI](https://devcenter.heroku.com/articles/heroku-ci) integration and the ability to configure the project structure (which includes support for repositories where the Flutter project isn't located directly in the root directory).

## Features
✅ Light footprint (deploys only the compiled files, stripping all sources, tools, and dependencies) \
✅ Configurable directory structure \
✅ Heroku CI integration \
✅ Caching of dependencies (like Flutter, Dart, and pub.dev packages) between builds 

## Instalation
### With `heroku-buildpack-static` (recommended)

Simply add the following two Buildpacks in your app configuration (in this order):

1. `https://github.com/ee/heroku-buildpack-flutter-light`
2. `https://github.com/heroku/heroku-buildpack-static`

You can do this using the web interface (in the "Settings" tab) or with the [`heroku buildpacks:set`](https://devcenter.heroku.com/articles/buildpacks#setting-a-buildpack-on-an-application) command.

After deploying you should see your Flutter Web application. You can optionally customize the way the application is built and deployed using [Heroku Buildpack Flutter Light's options](#options) and [heroku-buidlpack-static's configuration](https://github.com/heroku/heroku-buildpack-static#configuration).

### With a different server-side technology (advanced)

If your Flutter application is a part of a bigger project that already has its way of serving static files (for example with Node, Python, PHP, or Ruby), you can easily reuse it.

Add `https://github.com/ee/heroku-buildpack-flutter-light` to your Buildpacks, set `FLUTTER_SOURCE_DIR` to the directory (within your repository) where the Flutter project is located, and `FLUTTER_DEPLOY_DIR` to where the compiled application should be placed (i.e., a directory that your framework uses to serve static files).

## Heroku CI integration
The Buildpack can be easily integrated with Heroku's Continuous Integration service ([Heroku CI](https://devcenter.heroku.com/articles/heroku-ci)) to automatically run linters and tests. 

Use the following steps:

1. Create a Heroku [pipleine](https://devcenter.heroku.com/articles/pipelines)
2. Enable Heroku CI in the pipeline's configuration
3. Create an `app.json` file in the root of your repository

The `app.json` file should contain the app configuration, including the testing command that should be run by Heroku CI. All `app.json` options can be found in [Heroku's `app.json` documentation](https://devcenter.heroku.com/articles/app-json-schema). What follows is a very minimalistic (but working) `app.json` that runs Dart's built-in analyzer and your project's Flutter tests:

```
{
  "environments": {
    "test": {
      "scripts": {
        "test": "flutter analyze && flutter test"
      }
    }
  },
  "buildpacks": [
    {
      "url": "https://github.com/ee/heroku-buildpack-flutter-light"
    }
  ],
}
```

Alternatively, for repositories where the Flutter project is not located in the root of the repository, the following configuration can be used (where `flutter-directory` is the location of the Flutter project):

```
{
  "environments": {
    "test": {
      "scripts": {
        "test": "cd flutter-directory && flutter analyze && flutter test"
      }
    }
  },
  "buildpacks": [
    {
      "url": "https://github.com/ee/heroku-buildpack-flutter-light"
    }
  ],
  "env": {
    "FLUTTER_SOURCE_DIR": "flutter-directory"
  }
}
```

During tests, all your project's dependencies are present (including the Dart SDK, Flutter SDK, pub.dev requirements, and command-line tools), so you can use them when specifying the testing command. 

#### Options

Optionally, you can use [config variables](https://devcenter.heroku.com/articles/config-vars) to customize the Buildpack's behavior:

| Variable |  Default        |  Description
|----------|------------------| -------------------|
| FLUTTER_CLEANUP | `true` | Whether to remove sources, tools, and dependencies before deployment |
| FLUTTER_VERSION | *Last version in [beta channel](https://flutter.dev/docs/development/tools/sdk/releases?tab=linux).* | The **name of the version** used to compile the project
| FLUTTER_BUILD | `flutter build web --release --quiet` | The command used to build the project | 
| FLUTTER_SOURCE_DIR | `/` | The folder in your repository where the Flutter project is kept |
| FLUTTER_DEPLOY_DIR | `public_html` | The folder where the compiled app should be placed |