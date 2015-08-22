---
title: "Router: Dynamic Templates"
date: "2015-08-15"
description: "Use router's template provider to create dynamic templates."
categories:
  - "angular"
  - "test"
hero: "/img/dynamic-templates.png"
heroalt: "Dynamic templates"
---

This post discusses how to create dynamic templates by leveraging the templateProvider configuration provided by Angular's built-in router or the third-party UI Router.
<!--more-->

<br />


## PROBLEM

For Single Page Applications (SPAs), we often need to switch views or states inside containers. This is usually done through routers. With either Angular's built-in router or the popular UI Router, we are able to define the relationship between states and their templates. For instance, here we defined a state `home` and its template URL `app/home/home.html`:

~~~js
app.config(function($stateProvider) {
  $stateProvider.state('home', {
    url: '/',
    templateUrl: 'app/home/home.html'
  });
});
~~~

In some cases, this state-to-template relationship can not be determined beforehand at the `config` time. The decision of what the template or template URL will be used for a state has to wait for the availability of run-time data. For example:

* User's account type, e.g. show Home version A for members, and version B for public users.
* A/B testing, e.g. a A/B testing service randomly picks from two versions -- A or B.

In either scenario, the template cannot be fixed to `app/home/home.html`, and has be to resolved using run-time data.

Router's `templateUrl` configuration accepts a function which can be used to create dynamic template URL. However, we are not able to inject run-time dependencies (e.g. user services, or A/B test services) into the templateUrl function. The only available argument of the `templateUrl` function is `$stateParams`.

~~~js
$stateProvider.state('home', {
  templateUrl: function($stateParams) { // Can not inject dependencies
    return 'app/home.' + $stateParams.option + '.html';
  }
});
~~~

<br />


## SOLUTION

The anwser is `templateProvider`.

Both Angular built-in router and the UI Router have a `templateProvider` configuration. `templateProvider` accepts a function that can be injected with run-time dependencies.

~~~js
$stateProvider.state('home', {
  templateProvider: function(abTestService) { // abTestService is injected here
    var result = abTestService.pick('a', 'b'); // Choose version A or B
    return '...'; // Return template content based on the result
  }
});
~~~

`templateProvider` returns template content (not an URL to the template). We can certainly embed HTML markups directly in JavaScript, but for complicate HTML, it's better to externalize the HTML content to separate template files. Here, we created `home-a.html` and `home-b.html`, and `ngInclude` them in the templateProvider function:

~~~html
<!-- Home version A at app/home/home-a.html -->
<div ng-controller="HomeAController">Version A</div>

<!-- Home version B at app/home/home-b.html -->
<div ng-controller="HomeBController">Version B</div>
~~~

~~~js
$stateProvider.state('home', {
  templateProvider: function(abTestService) {
    var result = abTestService.pick('a', 'b');

    // ngInclude template content based on the A/B test result
    return '<div ng-include="\'app/home/home-' + result + '.html\'"></div>';
  }
});
~~~

`templateProvider` can also return a Promise which is resolved to template content.

~~~js
$stateProvider.state('home', {
  templateProvider: function($http, USER_SERVICE_REST_URL) {

    // Here, we return a promise instead of the template content
    return $http.get(USER_SERVICE_REST_URL).then(function(data) {
      var result = (data.type === 'member' ? 'a' : 'b');

      // Return the template content
      return '<div ng-include="\'app/home/home-' + result + '.html\'"></div>';
    });
  }
});
~~~

<br />


## EVEN BETTER SOLUTION

Having `ngInclude` in `templateProvider` function feels still a bit hackish to me. The ideal solution is to specify a template URL, and then let Angular fetch the content. However, sending separate HTTP requests just to fetch templates seems to be unnecessary web traffic. It will be better if the template content can be cached in the $templateCache service; and then, all I need to do is `$templateCache.get('templateUrl')`:

~~~js
$stateProvider.state('home', {
  templateProvider: function(abTestService, $templateCache) {
    var result = abTestService.pick('a', 'b');

    // Retrieve the cached template content from $templateCache service
    return $templateCache.get('app/home/home-' + result + '.html');
  }
});
~~~

To achieve this, we need a Gulp task to convert all HTML files under the app/ directory to JavaScript strings, and save the strings in $templateCache.

~~~js
// Load gulp and its plugins
var gulp = require('gulp');
var minifyHtml = require('gulp-minify-html');
var angularTemplateCache = require('gulp-angular-templatecache');


gulp.task('templates', function() {

  return cacheTemplates('src/app/**/*.html', 'app.template.js');

  function cacheTemplates(input, output) {
    return gulp.src(input) // Get all HTML files
      .pipe(minifyHtml({ // Minify HTML content first
        empty: true,
        spare: true,
        quotes: true
      }))
      .pipe(angularTemplateCache(output, { // Save minified strings to cache
        module: 'myApp' // Setup $templateCache for Angular module 'myApp'
      }))
      .pipe(gulp.dest('.tmp/templates/'));

  } // /function cacheTemplates

});
~~~

Then, import the generated `template.js` in `index.html`:

~~~html
<script src=".tmp/templates/app.template.js"></script>
~~~


<br />


## CONCLUSION

By leveraging the `templateProvider` function that can be injected with dependencies, we are able to resolve template content based on run-time data. This technique is useful for switching among more than one templates for a state, for instance, A/B testing, and swappable content in limited space.

<br />
