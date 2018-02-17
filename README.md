# html-webpack-externals-plugin [![CircleCI](https://circleci.com/gh/mmiller42/html-webpack-externals-plugin.svg?style=svg)](https://circleci.com/gh/mmiller42/html-webpack-externals-plugin) [![Greenkeeper badge](https://badges.greenkeeper.io/mmiller42/html-webpack-externals-plugin.svg)](https://greenkeeper.io/)

Webpack plugin that works alongside [`html-webpack-plugin`](https://github.com/jantimon/html-webpack-plugin) to use pre-packaged vendor bundles.

## How it works

This plugin is very simple and just encapsulates two other Webpack plugins to do the heavy lifting. It:

1. modifies your Webpack config at runtime to add your vendor modules to the [`externals`](https://webpack.js.org/configuration/externals/) property.
1. runs the [`copy-webpack-plugin`](https://github.com/kevlened/copy-webpack-plugin) to copy your vendor module assets into the output path.
1. runs the [`html-webpack-include-assets-plugin`](https://github.com/jharris4/html-webpack-include-assets-plugin) to add your vendor module bundles to the HTML output.

## Installation

```sh
npm install --save-dev html-webpack-externals-plugin
```

## Usage

Require the plugin in your Webpack config file.

```js
const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin')
```

Then instantiate it in the `plugins` array, after your instance of `html-webpack-plugin`.

```js
plugins: [
  new HtmlWebpackPlugin(),
  new HtmlWebpackExternalsPlugin(
    // See API section
  ),
]
```

## API

The constructor takes a configuration object with the following properties.

| Property | Type | Description | Default |
| --- | --- | --- | --- |
| `externals` | array&lt;object&gt; | An array of vendor modules that will be excluded from your Webpack bundle and added as `script` or `link` tags in your HTML output. | *None* |
| `externals[].module` | string | The name of the vendor module. This should match the package name, e.g. if you are writing `import React from 'react'`, this would be `react`. | *None* |
| `externals[].entry` | string \| array&lt;string&gt; \| object \| array&lt;object \| string&gt; | The path, relative to the vendor module directory, to its pre-bundled distro file. e.g. for React, use `dist/react.js`, since the file exists at `node_modules/react/dist/react.js`. Specify an array if there are multiple CSS/JS files to inject. To use a CDN instead, simply use a fully qualified URL beginning with `http://`, `https://`, or `//`.<br><br>For entries whose type (JS or CSS) cannot be inferred by file extension, pass an object such as `{ path: 'https://some/url', type: 'css' }` (or `type: 'js'`). | *None* |
| `externals[].entry.path` | string | If entry is an object, the path to the asset. | *None* |
| `externals[].entry.type` | `'js'`\|`'css'` | The asset type, if it cannot be inferred. | *Inferred by extension when possible* |
| `externals[].entry.cwpPatternConfig` | object | The properties to set on the [`copy-webpack-plugin` pattern object](https://github.com/webpack-contrib/copy-webpack-plugin#patterns). This object is merged in with the default `from` and `to` properties which are generated by the externals plugin. | `{}` |
| `externals[].entry.attributes` | object.&lt;string,string&gt; | Additional attributes to add to the injected tag. | `{}` |
| `externals[].global` | string \| null | For JavaScript modules, this is the name of the object globally exported by the vendor's dist file. e.g. for React, use `React`, since `react.js` creates a `window.React` global. For modules without an export (such as CSS), omit this property or use `null`. | `null` |
| `externals[].supplements` | array&lt;string&gt; \| array&lt;object&gt; | For modules that require additional resources, specify globs of files to copy over to the output. e.g. for Bootstrap CSS, use `['dist/fonts/']`, since Glyphicon fonts are referenced in the CSS and exist at `node_modules/bootstrap/dist/fonts/`. Supplements can be specified as just an array of paths, or an array of objects with a path and copy plugin pattern object. | `[]` |
| `externals[].supplements[].path` | string | The glob path to copy assets from. | *None* |
| `externals[].supplements[].cwpPatternConfig` | object | The properties to set on the [`copy-webpack-plugin` pattern object](https://github.com/webpack-contrib/copy-webpack-plugin#patterns). This object is merged in with the default `from` and `to` properties which are generated by the externals plugin. | `{}` |
| `externals[].append` | boolean | Set to true to inject this module after your Webpack bundles. | `false` |
| `hash` | boolean | Set to true to append the injected module distro paths with a unique hash for cache-busting. | `false` |
| `outputPath` | string | The path (relative to your Webpack `outputPath`) to store externals copied over by this plugin. | `vendor` |
| `publicPath` | string \| null | Override Webpack config's `publicPath` for the externals files, or `null` to use the default `output.publicPath` value. | `null` |
| `files` | string \| array&lt;string&gt; \| null | If you have multiple instances of HtmlWebpackPlugin, use this to specify globs of which files you want to inject assets into. Will add assets to all files by default. | `null` |
| `cwpOptions` | object | The [options object](https://github.com/webpack-contrib/copy-webpack-plugin#options) to pass as the `copy-webpack-plugin` constructor's second parameter. | `{}` |
| `enabled` | boolean | Set to `false` to disable the plugin (useful for disabling in development mode). | `true` |

## Examples

### Local JS external example

This example assumes `jquery` is installed in the app. It:

1. adds `jquery` to your Webpack config's `externals` object to exclude it from your bundle, telling it to expect a global object called `jQuery` (on the `window` object)
1. copies `node_modules/jquery/dist/jquery.min.js` to `<output path>/vendor/jquery/dist/jquery.min.js`
1. adds `<script type="text/javascript" src="<public path>/vendor/jquery/dist/jquery.min.js"></script>` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'jquery',
      entry: 'dist/jquery.min.js',
      global: 'jQuery',
    },
  ],
})
```

### Local CSS external example

This example assumes `bootstrap` is installed in the app. It:

1. copies `node_modules/bootstrap/dist/css/bootstrap.min.css` to `<output path>/vendor/bootstrap/dist/css/bootstrap.min.css`
1. adds `<link href="<public path>/vendor/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet">` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'bootstrap',
      entry: 'dist/css/bootstrap.min.css',
    },
  ],
})
```

### Local external with supplemental assets example

This example assumes `bootstrap` is installed in the app. It:

1. copies `node_modules/bootstrap/dist/css/bootstrap.min.css` to `<output path>/vendor/bootstrap/dist/css/bootstrap.min.css`
1. copies all contents of `node_modules/bootstrap/dist/fonts/` to `<output path>/vendor/bootstrap/dist/fonts/`
1. adds `<link href="<public path>/vendor/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet">` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'bootstrap',
      entry: 'dist/css/bootstrap.min.css',
      supplements: ['dist/fonts/'],
    },
  ],
})
```

### CDN example

This example does not require the `jquery` module to be installed. It:

1. adds `jquery` to your Webpack config's `externals` object to exclude it from your bundle, telling it to expect a global object called `jQuery` (on the `window` object)
1. adds `<script type="text/javascript" src="https://unpkg.com/jquery@3.2.1/dist/jquery.min.js"></script>` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'jquery',
      entry: 'https://unpkg.com/jquery@3.2.1/dist/jquery.min.js',
      global: 'jQuery',
    },
  ],
})
```

### URL without implicit extension example

Some CDN URLs don't have file extensions, so the plugin cannot determine whether to use a `link` tag or a `script` tag. In these situations, you can pass an object in place of the `entry` property that specifies the path and type explicitly.

This example uses the Google Fonts API to load the Roboto font. It:

1. adds `<link href="https://fonts.googleapis.com/css?family=Roboto" rel="stylesheet">` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'google-roboto',
      entry: {
        path: 'https://fonts.googleapis.com/css?family=Roboto',
        type: 'css',
      },
    },
  ],
})
```

### Module with multiple entry points example

Some modules require more than one distro file to be loaded. For example, Bootstrap has a normal and a theme CSS entry point.

This example assumes `bootstrap` is installed in the app. It:

1. copies `node_modules/bootstrap/dist/css/bootstrap.min.css` to `<output path>/vendor/bootstrap/dist/css/bootstrap.min.css`
1. copies `node_modules/bootstrap/dist/css/bootstrap-theme.min.css` to `<output path>/vendor/bootstrap/dist/css/bootstrap-theme.min.css`
1. copies all contents of `node_modules/bootstrap/dist/fonts/` to `<output path>/vendor/bootstrap/dist/fonts/`
1. adds `<link href="<public path>/vendor/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet">` to your HTML file, before your chunks
1. adds `<link href="<public path>/vendor/bootstrap/dist/css/bootstrap-theme.min.css" rel="stylesheet">` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'bootstrap',
      entry: ['dist/css/bootstrap.min.css', 'dist/css/bootstrap-theme.min.css'],
      supplements: ['dist/fonts/'],
    },
  ],
})
```

### Appended assets example

Sometimes you want to load the external after your Webpack chunks instead of before.

This example assumes `bootstrap` is installed in the app. It:

1. copies `node_modules/bootstrap/dist/css/bootstrap.min.css` to `<output path>/vendor/bootstrap/dist/css/bootstrap.min.css`
1. adds `<link href="<public path>/vendor/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet">` to your HTML file, *after* your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'bootstrap',
      entry: 'dist/css/bootstrap.min.css',
      append: true,
    },
  ],
})
```

### Cache-busting with hashes example

You can configure the plugin to append hashes to the query string on the HTML tags so that, when upgrading modules, a new hash is computed, busting your app users' caches. **Do not use this in tandem with CDNs, only when using local externals.**

This example assumes `bootstrap` is installed in the app. It:

1. copies `node_modules/bootstrap/dist/css/bootstrap.min.css` to `<output path>/vendor/bootstrap/dist/css/bootstrap.min.css`
1. adds `<link href="<public path>/vendor/bootstrap/dist/css/bootstrap.min.css?<unique hash>" rel="stylesheet">` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'bootstrap',
      entry: 'dist/css/bootstrap.min.css',
    },
  ],
  hash: true,
})
```

### Customizing output path example

By default, local externals are copied into the Webpack output directory, into a subdirectory called `vendor`. This is configurable.

Do not include a trailing slash or leading slash in your output path, they are concatenated automatically by the plugin.

This example assumes `bootstrap` is installed in the app. It:

1. copies `node_modules/bootstrap/dist/css/bootstrap.min.css` to `<output path>/thirdparty/bootstrap/dist/css/bootstrap.min.css`
1. adds `<link href="<public path>/thirdparty/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet">` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'bootstrap',
      entry: 'dist/css/bootstrap.min.css',
    },
  ],
  outputPath: 'thirdparty',
})
```

### Customizing public path example

By default, local externals are resolved from the same root path as your Webpack configuration file's `output.publicPath`, concatenated with the `outputPath` variable. This is configurable.

You should include a trailing slash in your public path, and a leading slash if you want it to resolve assets from the domain root.

This example assumes `bootstrap` is installed in the app. It:

1. copies `node_modules/bootstrap/dist/css/bootstrap.min.css` to `<output path>/vendor/bootstrap/dist/css/bootstrap.min.css`
1. adds `<link href="/assets/vendor/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet">` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'bootstrap',
      entry: 'dist/css/bootstrap.min.css',
    },
  ],
  publicPath: '/assets/',
})
```

### Adding custom attributes to tags example

Sometimes you may want to add custom attributes to the link or script tags that are injected.

This example does not require the `jquery` module to be installed. It:

1. adds `jquery` to your Webpack config's `externals` object to exclude it from your bundle, telling it to expect a global object called `jQuery` (on the `window` object)
1. adds `<script type="text/javascript" src="https://code.jquery.com/jquery-3.2.1.js" integrity="sha256-DZAnKJ/6XZ9si04Hgrsxu/8s717jcIzLy3oi35EouyE=" crossorigin="anonymous"></script>` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'jquery',
      entry: {
        path: 'https://code.jquery.com/jquery-3.2.1.js',
        attributes: {
          integrity: 'sha256-DZAnKJ/6XZ9si04Hgrsxu/8s717jcIzLy3oi35EouyE=',
          crossorigin: 'anonymous',
        },
      },
      global: 'jQuery',
    },
  ],
})
```

### Passing an options argument to the `copy-webpack-plugin` constructor example

You can change the default context for all patterns that are copied by the `copy-webpack-plugin`, enable debug mode, etc. by passing additional options to the plugin.

This example assumes `bootstrap` is installed in the app via bower. It:

1. copies `bower_components/bootstrap/dist/css/bootstrap.min.css` to `<output path>/vendor/bootstrap/dist/css/bootstrap.min.css`
1. adds `<link href="<public path>/vendor/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet">` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'bootstrap',
      entry: 'dist/css/bootstrap.min.css',
    },
  ],
  cwpOptions: {
    context: path.resolve(__dirname, 'bower_components'),
  },
})
```

### Passing custom options to `copy-webpack-plugin` for an entry example

In certain instances, you might want to control the properties that are passed in with the pattern object provided to `copy-webpack-plugin`, in order to control context, caching, output path, etc.

This example assumes `bootstrap` is installed in the app via bower. It:

1. copies `bower_components/bootstrap/dist/css/bootstrap.min.css` to `<output path>/vendor/bootstrap/dist/css/bootstrap.min.css`
1. adds `<link href="/assets/vendor/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet">` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'bootstrap',
      entry: {
        path: 'dist/css/bootstrap.min.css',
        cwpPatternConfig: {
          context: path.resolve(__dirname, 'bower_components'),
        },
      },
    },
  ],
})
```

### Passing custom options to `copy-webpack-plugin` for a supplement example

In certain instances, you might want to control the properties that are passed in with the pattern object provided to `copy-webpack-plugin`, in order to control context, caching, output path, etc.

This example assumes `./mod` is a directory in your app that contains some dist files.

1. copies `mod/dist/mod.css` to `<output path>/vendor/mod/dist/mod.css`
1. copies all contents of `mod/dist/{a,b}/` to `<output path>/vendor/mod/dist/{a,b}/`, passing `nobrace` to `node-glob` so that `{a,b}` is matched literally instead of expanded into a glob
1. adds `<link href="<public path>/vendor/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet">` to your HTML file, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'mod',
      entry: 'dist/mod.css',
      supplements: [
        {
          path: 'dist/{a,b}/',
          cwpPatternConfig: {
            fromArgs: {
              nobrace: true,
            },
          },
        },
      ],
    },
  ],
  cwpOptions: {
    context: __dirname,
  },
})
```

### Specifying which HTML files to affect example

If you are using multiple instances of html-webpack-plugin, by default the assets will be injected into every file. This is configurable.

This example assumes `bootstrap` is installed in the app. It:

1. copies `node_modules/bootstrap/dist/css/bootstrap.min.css` to `<output path>/vendor/bootstrap/dist/css/bootstrap.min.css`
1. adds `<link href="/public/vendor/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet">` to *only the `about.html` file*, before your chunks

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'bootstrap',
      entry: 'dist/css/bootstrap.min.css',
    },
  ],
  files: ['about.html'],
})
```

### Disabling the plugin

Sometimes you only want the plugin to be activated in certain environments. Rather than create separate Webpack configs or mess with splicing the plugins array, simply set the `enabled` option to `false` to disable the externals plugin entirely.

```js
new HtmlWebpackExternalsPlugin({
  externals: [
    {
      module: 'jquery',
      entry: 'dist/jquery.min.js',
      global: 'jQuery',
    },
  ],
  enabled: process.env.NODE_ENV === 'production',
})
```
