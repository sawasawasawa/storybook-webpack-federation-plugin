
# Storybook Webpack Federation Plugin
Exposes all the components in your Storybook as Webpack 5 federated components.

## Motivation
Design systems are all the fad these days, and we wanted to create an easy way to share them. Since Storybook has proven to be a great way do that, we figured why not also make the source of truth for the current state of components also be the place where you use them?

[Checkout the article we wrote about it here.](TODO) (coming very soon)

## Installation

### For your Storybook

Install the plugin.
```bash
yarn add storybook-webpack-federation-plugin -D
```

Storybook currently uses Webpack 4, which means we have to do a few extra steps to install Webpack 5 as that's where federation has been added. Once Storybook starts using Webpack 5 we won't need to do these steps.

First we need to install the latest Webpack5 directly from Github:
```bash
yarn add webpack@"git://github.com/webpack/webpack.git#dev-1" webpack-cli -D
```

Storybook has its own webpack configuration that you can normally extend, but we can't do that yet so we have to create a new `webpack.config.js` speficif for WP5. Here's an example confiugration which you might want to customize based on your setup.

```javascript
const path = require("path");

module.exports = {
  cache: false,

  mode: "development",
  devtool: "source-map",

  optimization: {
    minimize: false,
  },

  resolve: {
    extensions: [".jsx", ".js", ".json", ".tsx", ".ts"],
  },

  module: {
    rules: [
      {
        test: /\.tsx?$/,
        loader: require.resolve("babel-loader"),
      },
      {
        test: /\.css$/i,
        use: ["style-loader", "css-loader"],
      },
    ],
  },

  output: {},

  plugins: [],

};
```

Next we'll import the Storybook Webpack Federation Plugin:
```javascript
const {StorybookWebpackFederationPlugin} = require("storybook-webpack-federation-plugin")
```

Add a new endpoint as an `output` of Storybook:
```javascript
  output: {
    // location of where the compiled Storybook lives
    path: path.resolve(__dirname, "storybook-static/federation"),
    // the url where Storybook will be accessible from
    publicPath: "https://lab.xolv.io/federated-design-system/xolvio-ui",
  },
```

And finally configure the plugin itself in the `plugins` section:
```javascript
plugins: [
    new StorybookWebpackFederationPlugin({
      name: "xolvio-ui", // this will be used by the consuming federation host
      files: { // paths to the components
        paths: ["./src/**/*.ts{,x}"],
      },
    }),
  ],
```

For convenience you'll probably want to set npm scripts for building your storybook and federation like this:
```json
"scripts": {
  "start": "start-storybook",
  "build": "yarn build:storybook && yarn build:federation",
  "build:federation": "rm -rf storybook-static/federation && webpack --mode production",
  "build:storybook": "build-storybook"
}
```

Let's now build it:
```bash
yarn build
```

And that's all for the Storybook side!

### For your app

Install the plugin:

```bash
yarn add storybook-webpack-federation-plugin -D
```

We need to make sure we are using the beta version of Webpack 5 here as well:
```bash
yarn add webpack@"git://github.com/webpack/webpack.git#dev-1" -D
```

Go to your `webpack.config.js` and require the plugin at the top as before:

```javascript
const {StorybookWebpackFederationPlugin} = require("storybook-webpack-federation-plugin")
```

Use it in the plugins section:

```javascript
  plugins: [
    new StorybookWebpackFederationPlugin({
      remotes: ["xolvio-ui"],
    }),
  ],
```

Finally, we add the Storybook endpoint that we exposed above in the app's `index.html`:
```html
<head>
  <script src="https://lab.xolv.io/federated-design-system/xolvio-ui/federation/remoteEntry.js"></script>
</head>
```

And now you can start using components from your published Storybook!

```javascript
import { Background } from "xolvio-ui/elements/Background";
import { Title } from "xolvio-ui/components/Title";
import { CenteredContentWrapper } from "xolvio-ui/helpers/CenteredContentWrapper";

export const Services = () => (
  <CenteredContentWrapper>
    <Background/>
    <Title title="hello" subheading="world" />
  </CenteredContentWrapper>
);

```

You can also use lazy loading:

```javascript
const CenteredContentWrapper = React.lazy(() => import("xolvio-ui/CenteredContentWrapper"));
const Background = React.lazy(() => import("xolvio-ui/Background"));
const Title = React.lazy(() => import("xolvio-ui/Title"));

export const Services = () => (
    <React.Suspense fallback={"Loading Components from the Design System"}>
      <CenteredContentWrapper>
       <Background/>
       <Title title="hello" subheading="world" />
     </CenteredContentWrapper>
    </React.Suspense>
);
```

And that's all there is to it! Enjoy :)

We'll come back and update the docs to make this even easier once Storybook is using Webpack 5.

## API
Below you can find a description of the fields in the configuration for this plugin:

```javascript
{
  // The name that the consumers will reference as the remote
  name: "xolvio-ui",
  
  files: {
  
    // an array of globs to match your component files
    paths: ["./src/components/**/*.ts{,x}"], 
    
    // files with .stories. will get ignored, so they don't get exposed on the endpoints
    storiesExtension: ".stories.",
    
     // so your App can import "xolvio-ui/components/Title" instead of  "xolvio-ui/src/components/Title"
    removePrefix: "./src/",
    
  },
  
  // by default we share react and react-dom, you can add any aditional packages you would want to be shared
  shared: ["styled-components"], 
  
  // you can import modules from other federated remotes into your Storybook as well!
  remotes: ["first-remote", "second-remote"]
  
}
```
