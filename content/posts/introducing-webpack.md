---
title: "Webpack: Pack Your Web Assets"
date: "2015-08-22"
description: "Introduce Webpack - a module bundler for web apps."
categories:
  - "webpack"
  - "nodejs"
  - "bower"
  - "gulp"
  - "es6"
  - "typescript"
hero: "/img/webpack.png"
heroalt: "Webpack"
---

I just had a presentation to introduce Webpack to our UI Guild in my company. Webpack is a module bundler to pack all kinds of web assets (JavaScript, styles, images, HTML partials, etc.) into one or more bundle JS files.
<!--more-->

Here is [my presentation](https://docs.google.com/presentation/d/1frBHDkSLvarbjFukhe3EuTuijpyTMHXwJDKu65OSVx8/edit?usp=sharing).

I should have highlighted the difference between Webpack and Require JS. Some developers are confused by these two tools.

* Require JS (without using any plugin) sends a separate HTTP request to retrieve each dependency module.
* However, Webpack is a pre-processor that bundles all required modules into one file before the bundle file is downloaded to users' browsers.
* Require JS apps load dependency on demand, but complicate apps with many dependencies tend to send a lot of requests which might impact performance.
* Webpack merges dependencies into one or more files, so the number of HTTP requests will be dramatically reduced, however, the initial download payload might be big. Webpack can be fine tuned to find a balance between the number of requests and download file size.

I love how we can use Webpack to leverage Common JS module system to encapsulate dependency declarations in their requiring modules instead of flattening the dependency graph in the index.html.

Moreover, because tasks such as packaging, minification, source mapping, and transpiling can be done in Webpack, Gulp/Grunt scripts will be greatly simplified. And, we probably don't even need Bower since almost all JS libraries can be installed from NPM.

Something remained to be tested are:

* How will Webpack solve library version conflicts? For instance, if Module A requires Angular 1.3 and Module B requires Angular 1.4, whether and how will Webpack help us solve the conflict? (Bower forces developers to manually resolve conflicts.)

* UI Performance. With Webpack, style sheets will be imported through dynamically injected `<style>` tags, will this introduce flickering in UI? We have to test it.

I will come back to these questions once I run some tests.

<br />
