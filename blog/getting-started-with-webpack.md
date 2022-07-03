---
draft: true
date: 2022-07-04T00:00:00+05:30
title: Getting started with webpack
tags:
- build tools
- webpack
description: A gentle introduction to webpack and its friends.
cover: ''
coverImageCredits: ''

---
## Objectives

1. What is webpack?
2. What's the advantage of it?
3. How to use webpack?
4. What are webpack loaders?
5. What are webpack plugins?
6. How to create multiple configurations?

## Introduction

So you might have used create react app or angular-CLI to generate the boilerplate for your application.

Even though these tools use webpack under the hood, they abstract most of the configuration for you.

But sometimes, you might have to customize this configuration depending upon the requirements. So it's better to have a little understanding of how things work under the hood.

This article is part one of the webpack series. In this article, we go over basic webpack concepts. In the following article, we will go over some advanced concepts like code-splitting, HMR, source map configurations, etc.

The first part of this article covers basic webpack concepts. In the second part, we'll be building a simple application with everything we learned.

## What is webpack?

Webpack is a module bundler for JavaScript applications. In plain English, webpack creates a dependency graph of all files (using imports/require statements, plugins, loaders, etc.). Using this dependency graph, webpack outputs the bundled code.

![Webpack key steps illustation - webpack for beginners](/uploads/webpackbundle.png "Webpack illustration")

Webpack treats every file or asset as a module. Out of the box, webpack can only process JavaScript files (ES Modules, CommonJS modules, AMD Modules).

But we can extend it to other file types like CSS, images, etc., with the help of loaders and plugins. Which we will be looking at later in this article.

## Core Concepts

Key concepts which are helpful when building applications with webpack are

1. Entry
2. Output
3. Loaders
4. Plugins
5. Mode

### Entry

An entry point is a JavaScript file that serves as a starting point for webpack to collect all the dependencies used by your application to build a dependency graph. Dependencies are libraries like React, jQuery, etc., OR static assets like images, CSS files, etc.

From webpack >= 4, Out of the box `src/index.js` serves as the entry point.

But you can configure the entry point by using `entry` property in the config file, and you can also have multiple entry points.

    module.exports = {
      entry: 'src/app.js', //Custom entry point
    };

### Output

Output is the path where webpack will bundle all the dependencies and your app code into single or multiple files.

From webpack >= 4, Out of the box `dist/main.js` serves as the output path.

Like entry point, you can also configure output using the `output` property.

    module.exports = {
      entry: 'src/app.js', //Custom entry point,
      output: {
        filename: 'main.js',
        path: path.resolve(__dirname, 'build'), //custom output point
      },
    };

### Loaders

Loaders are kind of like extensions that help webpack process other types of files like CSS, images, markdown files, etc.

Loaders convert these files into JavaScript modules. These converted modules will be used by webpack.

For example, with the help of `ts-loader` you can transpile typescript files.

     module: {
        rules: [
          {
            test: /\.ts$/i,
            use: ["ts-loader"],
          },
        ],
      },

Loaders have two properties:

1. `test` - Used to identify the file types which need to be transformed.
2. `use` - User to determine what loader to use on this file type.

> If you want to use multiple loaders on a single file, you can pass `use` property an array `['loader-1', 'loader-2']`.

But keep in mind that loaders will be applied from **right to left** so that loader-2 will be used first, then loader-1.

     module: {
        rules: [
          {
            test: /\.css$/i,
            use: ["style-loader", "css-loader"],
          },
        ],
      },

In the above code,

1. `css-loader` - Adds support to import CSS files directly into a javascript file.
2. `style-loader` - Used to inject the styles generated from `css-loader`  into the DOM.

### Plugins

While loaders can transform specific modules other than JS, plugins can perform various tasks like bundle optimization, asset management, etc.

> Webpack itself is made-up of plugins.

To keep things simple, let's have a look at [HtmlWebpackPlugin.](https://webpack.js.org/plugins/html-webpack-plugin/)

Basically what `HtmlWebpackPlugin` will do was, it creates an HTML template and add the script tags which point to bundled JS.

You can install it by running it.

     npm install html-webpack-plugin --save-dev

And add it config using `plugins` property.

```jsx
{
	...,
      plugins: [
          new HtmlWebpackPlugin({
            title: 'My App',
            template: path.resolve(__dirname, 'src', 'index.html'),
          }),
  		],   
    ...,
}
```

### Mode

## Example App

Let's build a simple app that calculates the age.

Create a folder with whatever name you want. Here, I'm calling it **age-calculator-webpack**.

    mkdir age-calculator-webpack
    cd age-calculator-webpack

After that run `npm init -y` to initialize the project

Now let's install webpack.

```shell
npm install webpack webpack-dev-server webpack-cli --save-dev
```

Also, install plugins for CSS and HTML

```shell
npm install css-loader style-loader html-webpack-plugin
```

Now lets the following files.