---
title: "How to use Styled-Components correctly in Remix"
date: 2023-01-28T20:00:00+01:00
weight: 100
covercaption: "Sometimes you just want to look good - that's where styling comes in!"
coveralt: "Stylish woman with shopping bags"
author: barklund
categories: ["how-to"]
tags: ["react", "typescript", "remix"]
---

[Remix](https://remix.run/) is a great new React website framework, one of the few to challenge the dominance of NextJS. Remix comes with a new philosophy, and with that also some new challenges and complexities when it comes to integrating third-party libraries.

---

## Styled-Components and Remix don't mix well

One such library is [Styled-Components](https://styled-components.com/), which is a popular CSS-in-JS library, that is really excellent for co-location and writing minimal JSX. IT's used by a lot of huge development teams including those at Airbnb, BBC, and Reddit.

While it is mentioned in the [Remix guide on styling](https://remix.run/docs/en/v1/guides/styling#css-in-js-libraries), that you can use this library, and it even comes with a (so-called) _[runnable example](https://github.com/remix-run/examples/tree/main/styled-components)_, this example doesn't actually work!

Go ahead, try [the linked Codepen](https://codesandbox.io/s/github/remix-run/examples/tree/main/styled-components). It's broken. Okay, it does work **at first**, but try changing something, and it doesn't actually update!

It's because React hydration fails. You will get a bunch of errors in the console like this:

```
Warning: Prop `className` did not match. Server: "sc-iBYQkv hiCBsX" Client: "sc-bcXHqe fA-DmNg"
```

The example is written in React 17, but even if you upgrade to React 18 and the latest Remix, you still get all the same hydration errors.

So that example is less than usefull - it's actually completely useless. If you get hydration errors, you lose a ton of the benefits, that serverside rendering (SSR) gives you.

The problem is, that you have Styled-Components running on both the server and the client, but not synchronized. You need to make sure that they are synchronized with every build, so that class names are generated uniquely, deterministically, and most inmportantly, _identically_ on the two runtimes. 

So how can we fix this?

---

## Fixing Remix to allow CSS-in-JS

In order to fix it, you need to hack into the underlying build engine, esbuild, in order for it to properly include the hydratable stylesheet in the output. We're going to go through these five steps to make it work:

1. Install `remix-esbuild-override`
2. Install `styled-components` and optionally types for it.
3. Add an esbuild plugin file for bundling styled components.
4. Add the plugin to the build in the Remix configuration file.
5. Use Styled-Components as usual.

### 1. Install esbuild override

First step is quite easy - install [the `remix-esbuild-override` library](https://github.com/aiji42/remix-esbuild-override) using your package manager. This library does exactly what it says on the tin: it allows you to override the remix esbuild configuration:

```sh
npm install -D remix-esbuild-override
# or
yarn add -D remix-esbuild-override
```

In order to correctly hook into the Remix build system, we need to run a script after the installation, so add a `postinstall` section to `package.json` like this:

```json
"scripts": {
  "postinstall": "remix-esbuild-override"
}
```

And then run the install command using your package manager again, e.g. just `npm install` or `yarn install`. This should trigger the post-install script and allow the plugin access to the guts of Remix.

### 2. Install Styled-Components and types

Next up, install Styled-Components:

```sh
npm install -S styled-components
# or
yarn add styled-components
```

And optionally install types as well, if you're using TypeScript:

```sh
npm install -D @types/styled-components
# or
yarn add -D @types/styled-components
```

### 3. Add an esbuild plugin

Next, we're going to add the esbuild plugin file. Create a new file at the root of the repository named `/styled-components-esbuild-plugin.js`.

You can copy the contents from below or from [this source on Github](https://raw.githubusercontent.com/aiji42/remix-esbuild-override/main/examples/styled-components/styled-components-esbuild-plugin.js):

```js
const babel = require("@babel/core");
const styled = require("babel-plugin-styled-components");
const fs = require("node:fs");
const path = require("path");

function styledComponentsPlugin() {
  return {
    name: "styled-components",
    setup({ onLoad }) {
      const root = process.cwd();
      onLoad({ filter: /\.[tj]sx$/ }, async (args) => {
        let code = await fs.promises.readFile(args.path, "utf8");
        let plugins = [
          "importMeta",
          "topLevelAwait",
          "classProperties",
          "classPrivateProperties",
          "classPrivateMethods",
          "jsx",
        ];
        let loader = "jsx";
        if (args.path.endsWith(".tsx")) {
          plugins.push("typescript");
          loader = "tsx";
        }
        const result = await babel.transformAsync(code, {
          babelrc: false,
          configFile: false,
          ast: false,
          root,
          filename: args.path,
          parserOpts: {
            sourceType: "module",
            allowAwaitOutsideFunction: true,
            plugins,
          },
          generatorOpts: {
            decoratorsBeforeExport: true,
          },
          plugins: [styled],
          sourceMaps: true,
          inputSourceMap: false,
        });
        return {
          contents:
            result.code +
            `//# sourceMappingURL=data:application/json;base64,` +
            Buffer.from(JSON.stringify(result.map)).toString("base64"),
          loader,
          resolveDir: path.dirname(args.path),
        };
      });
    },
  };
}

module.exports = styledComponentsPlugin;
```

Just save the file with that content, and you're ready to load it up in the next step.

### 4. Update the Remix configuration file

Open up the file `/remix.config.js` at the root of the repository. Here we want to make sure we load our new plugin. After updating this file, it should look like this:

```js
const { withEsbuildOverride } = require("remix-esbuild-override");
const styledComponentsPlugin = require("./styled-components-esbuild-plugin");

withEsbuildOverride((option) => {
  option.plugins.unshift(styledComponentsPlugin());

  return option;
});

/**
 * @type {import('@remix-run/dev').AppConfig}
 */
module.exports = {
  ignoredRouteFiles: [".*"],
  appDirectory: "app",
  assetsBuildDirectory: "public/build",
  serverBuildPath: "build/index.js",
  publicPath: "/build/",
};
```

Now spin up your local Remix server using `npm run dev` or `yarn dev` to see your styled application in action.

### 5. Enjoy using CSS-in-JS with Styled-Components

And that's it. you're good to go. You will get proper hydration, no error messages, and it just works both in the local development environment as well as in the production build.

You can now write your routes like this:

```tsx
import { Form, Link } from "@remix-run/react";
import styled from "styled-components";

const Main = styled.main`
  background-color: hotpink;
`;
const Title = styled.h1`
  font-family: monospace;
`;

export default function Index() {
  return (
    <Main>
      <Title>Welcome to the hot zone!</Title>
    </Main>
  );
}
```

You can of course also use Styled-Components in non-route components, just like you would in any real-world sizable project.

## Conclusion

Using Styled-Components in Remix seems like a no-brainer, but unfortunately it is not supported directly _out-of-the-box_ from the Remix team. However, we can still use it, if we run through a few extra hoops.

It's bit troublesome to do, but once you know what it takes, it should take you no more than 5 minutes to set up! With this short guide, you should be good to go!

This is confirmed to work as of Remix 1.11. Let me know if that changes in future versions!