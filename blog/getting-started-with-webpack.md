---
draft: true
date: 2022-06-14T00:00:00+05:30
title: Getting started with webpack
tags:
- build tools
- webpack
description: A gentle introduction to webpack.
cover: ''
coverImageCredits: ''

---
## Objectives

1. What is webpack?
2. What’s the advantage of it?
3. How to use webpack?
4. What are webpack loaders?
5. What are webpack plugins?
6. How to create multiple configurations?

## Introduction

So you might have used. `create-react-app`, `vue-cli` or `angular-cli` to generate the boilerplate of your app.

Even though these tools use webpack under the hood, they abstract most of the configuration for you.

But In some cases, you might have to customise the configuration based on the app you’re developing or the features you want. So it's better to understand how things work under the hood.

The first part of this tutorial covers basic webpack concepts. In the second part, we’ll be building a simple application with all the things we learned.

## What is webpack?

Webpack is a module bundler for JavaScript applications. Out of the box, webpack can process the following JavaScript module types.

* ES Modules
* CommonJS Modules
* AMD Modules

But we can extend it to use other types of files like CSS, images, etc., with Loaders' help. We’ll be looking into this article later.

## Core Concepts

Four key concepts are helpful when building applications with webpack are

1. Entry
2. Output
3. Loaders
4. Plugins

### Entry

An entry point is a JavaScript file that serves as a starting point for webpack to collect all the dependencies used by your application to build a dependency graph. Dependencies are libraries like React, jQuery etc.

From webpack >= 4, Out of the box `src/index.js` serves as the entry point.

But you can configure the entry point by using `entry` property in the config file, and you can also have multiple entry points.

    //webpack.config.jsmodule.exports = {    "entry": "src/app.js" //Custom entry point}

### Output

Output is the path where webpack will bundle all the dependencies and your app code into single or multiple files.

From webpack >= 4, Out of the box `dist/main.js` serves as the output path.

Like entry point, you can configure output using the `output` property.

    //webpack.config.jsmodule.exports = {    "entry": "src/app.js", //Custom entry point,    "output": "build/bundle.js" //Custom output path}

### Loaders

Loaders are the extensions that help webpack process other types of files like CSS, images, markdown files etc.

Loaders convert these files into JavaScript modules. These converted modules used by webpack.

For example, with the help of `css-loader` you can import CSS directly into your JavaScript file.

    //example.jsimport React from 'react'import './main.css' //behind the scenes css-loader converts it into a javascript module

Loaders have two properties:

1. `test` - It helps to identify the file types which need to be transformed
2. `use` - It helps to determine what type of loader to be used on this file type.

If you want to use multiple loaders on a single file you can pass `use` property as array `['loader-1', 'loader-2']`.

One thing to remember is loaders will be applied from right to left, So loader-2 will be used first, then loader-1.

    //webpack.config.jsmodule.exports = {    "entry": "src/app.js", //Custom entry point,    "output": "build/bundle.js", //Custom output path    "module": {        rules: [            {                test: /\.css$/ ,                use: ['style-loader', 'css-loader']            }        ]   },}

### Plugins

While loaders can transform certain types of modules other than JS, plugins can perform a broader range of tasks like bundle optimisation, asset management etc.

Webpack itself is made-up of plugins.

To keep things simple, let’s have a look at [HtmlWebpackPlugin.](https://webpack.js.org/plugins/html-webpack-plugin/)

Basically what `HtmlWebpackPlugin` will do was it creates an HTML template and add the script tags which point to bundled JS.

You can install it by running it.

     npm install html-webpack-plugin --save-dev

And add it config using `plugins` property

```jsx
//webpack.config.jsconst HtmlWebpackPlugin = require('html-webpack-plugin');module.exports = {    "entry": "src/app.js", //Custom entry point,    "output": "build/bundle.js", //Custom output path   "module": {        rules: [            {                test: /\.css$/ ,                use: ['style-loader', 'css-loader']            }        ]   },    plugins: [        new HtmlWebpackPlugin()    ]}
```

## Example App

Let’s build a simple app that calculates the age.

Create a folder with whatever name you want. Here, I’m calling it **age-calculator-webpack**.

    mkdir age-calculator-webpack
    cd age-calculator-webpack

After that run `npm init -y` to initialise the project

Now let’s install webpack