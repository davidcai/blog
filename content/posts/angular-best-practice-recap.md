---
title: "Angular Best Practice Recap"
date: "2015-08-01"
description: "Recap on what best practices I learned from the Angular community."
categories:
  - "angular"
hero: "/img/best-practice.png"
heroalt: "Best practice"
---

I attended a great Angular training provided by OasisDigital the last week. Learned so many tips and best practices from Bill Odom and his team. While my memory is still fresh, I'd like to document the stuff I learned. So again, I use my blog as study notes. While I'm on the topic of Angular best practices, I also like to bring in some advices from John Papa, Shai Reznik, and other wisdoms of the community. This article basically is a recap of what I heard and learned.

<br />


## PREFER FACTORY OVER SERVICE

A service is a simplified version of a factory. Services are a constructor functions. However, I've seen some service code that returns objects. The author of the code probably mistook services with factories. To clear the confusion, let's favor factories over services. I personally prefer naming a factory as xyzService instead of xyzFactory.

~~~js
angular
  .module('myApp')
  .factory('fooService', fooService)
;

function fooService() {
  return {
    getFoo: function() { ... },
    getBar: function() { ... }
  };
}
~~~

<br />


## ASSIGN THIRD-PARTY GLOBALS TO VALUES

Although I can use $window to reference globals of third-party libraries, I like to assign globals to values, and inject them. This makes function arguments a clear contract of what dependencies my code requires.

~~~js
angular
  .module('myApp')
  .value('_', _) // lodash
  .value('d3', d3) // D3
;

...

function MainController(_, d3) { // Injection
  _.forEach( ... );
  d3.select( ... );
}
~~~

For low-level libraries (e.g., lodash), I can even delete lodash's global '_' from the window object, so '_' will be available only through injection:

~~~js
angular
  .module('myApp')
  .factory('_', function() {
    var _ = window._;
    delete window._; // Delete _ from window

    return _;
  })
;
~~~

<br />


## STICK WITH $HTTP

I feel $http gives me more flexibilities than $resource. And $http service's promise interface is much nicer. Better to stick with $http than mixing $http and $resource which unnecessarily complicate my code.

<br />


## WRAP REST IN SERVICES

Encapsulate REST requests as methods in services, so users of the service don't have to know the interface of your REST APIs, and the choice of REST libraries ($http, $resource, etc.).

~~~js
angular
  .module('myApp')
  .factory('accountService', accountService)
;

function accountService($http, $log) {
  return {
    getAccounts: function(userId) {
      return $http.get('/api/accounts/' + userId)
        .then(function(response) {
          return response.data;
        })
        .catch(function(error) {
          $log.error(error);
          return error;
        })
      ;
    }
  };
}
~~~

<br />


## UI ROUTER RESOLVE PATTERN

I often run to the scenarios where I need to hold off displaying UI until certain data is ready. The resolve property of uiRouter is designed to tackle this common problem.

Assign a name-to-function map to the resolve property. If the function returns a value, the returned value is treated as dependency, and the value is injected into the controller; if the function returns a promise, the promise is resolved before the controller is instantiated, and the resolved value is injected into the controller.

~~~js
$stateProvider
  .state('home', {
    ... ,

    // Resolve items and users before routing to the 'home' state
    resolve: {
      items: function() {
        // Returns an array value
        return [
          { name: 'The Settlers of Catan', price: 51.46 },
          { name: 'Mansions of Madness', price: 56.52 }
        ];
      },
      users: function(userService) {
        // Returns a promise
        return userService.getUsers();
      }
    },

    // items and users are injected to the controller
    controller: function($scope, $log, items, users) {
      $log.log(items);
      $log.log(users);
    }
  });
~~~

<br />


## ISOLATE DIRECTIVE SCOPE

Think Angular directives as re-usable functions, and its scope as the argument list of a function. In most cases, a directive shouldn't assume the presence of data in the ancestor scopes or elements in the DOM. The scope (and the directive's DOM attributes) should be the only place where a directive retrieves external information.

Pass information to directives via element attributes:

~~~html
<div data-browse-happy-banner
  data-ie-version="8"
  data-on-dismiss="bannerDismissed()"></div>
~~~

Then retrieve these information from the scope:

~~~js
angular
  .module('myApp')
  .directive('browseHappyBanner', browseHappyBanner)
;

function browseHappyBanner() {
  return {
    restrict: 'A',
    scope: { // Isolated scope
      ieVersion: '@', // Pass ieVersion from data-ie-version attribute
      onDismiss: '&' // Pass onDismiss callback from data-on-dismiss attribute
    },
    template: '<div> ... <a data-ng-click="onClose()"></a></div>',
    ... ,
    link: function(scope) {
      // Retrieve ieVersion from scope
      var iev = parseInt(scope.ieVersion, 10);
      ...
      scope.onClose = function() {
        // Invoke onDismiss callback from the scope
        scope.onDismiss();  
      };
    }
  };
}
~~~

<br />


## CLEAN TEMPLATE

If snippets in a template look like program, then you are doing it wrong. Templates need to be declarative, easy to understand for non-programmers. The best practice is to imagine your template is written by a web designer rather than a developer.

Will a web designer understand the following template?

~~~html
<div data-ng-class="{
  'error': ((bFormSubmitted || PersonalInfoForm.DOBmonth.$dirty) && PersonalInfoForm.DOBmonth.$invalid) ||
            ((bFormSubmitted || PersonalInfoForm.DOBday.$dirty) && PersonalInfoForm.DOBday.$invalid) ||
            ((bFormSubmitted || PersonalInfoForm.DOByear.$dirty) && PersonalInfoForm.DOByear.$invalid ) || errorMapResult.birthdate,
  'success': (PersonalInfoForm.DOBmonth.$dirty && !PersonalInfoForm.DOBmonth.$invalid) &&
              (PersonalInfoForm.DOBday.$dirty && !PersonalInfoForm.DOBday.$invalid) &&
              (PersonalInfoForm.DOByear.$dirty && !PersonalInfoForm.DOByear.$invalid )}">
  ...
</div>
~~~

Guess no. Even developer might have hard time to understand this. Move the logic to controller or service, so the template becomes clean:

~~~html
<div data-ng-class="{ 'error': hasError, 'success': isSuccessful }">
  ...
</div>
~~~

<br />


## THIN CONTROLLER

Make controller thin. Move logic to services and directives. This makes controller much easy to test.

<br />


## LIGHT FILTER

Filter tends to be called multiple times. So if you have a filter that does a lot of heavy lifting, it will drag down the performance of the entire app. Try to simplify the logic in filter. Use cache or memoization to improve the performance.

~~~js

angular
  .module("myApp")
  .filter("heavyLifting", heavyLifting) // A filter does heavy lifting stuff
;

function heavyLifting() {

  var doHeavyLifting = function() { ... }; // Complicate process

  // Use lodash's memoize function to cache the returned result for a specific input.
  // Future calls with the same input will return the cached result.
  return _.memoize(function(input) {
    return doHeavyLifting(input);
  });
}
~~~

More details on filter and memoization can be found [here](http://www.terrencewatson.com/2014/06/28/memoize/)

<br />


## NAMED FUNCTIONS

Use named functions whenever you can. When error occurs inside the named function, the function name will appear in the stack trace, which is really helpful for debugging.

~~~js
// Named function showToggle
$timeout(function showToggle() {
  angular.element('.toggle').css('display', 'block');
}, 1000);

// Named function greet
var greet = function greet(name) {
  console.log('Hello ' + name);
};

// Named controller
angular.module('myApp').controller('MyController', MyController);
function MyController() { ... }
~~~

<br />


## SCOPE JS CODE IN IIFE

This best practice is not limited to Angular. IIFE (Immediately-Invoked Function Expression) is often used to create scopes in which the variables and functions will not be leaked to outer scopes.

~~~js
;(function() {
  ...
  var app = angular.module('myApp');
  ...
})();

console.log(app) // not defined
~~~

<br />


## PYRAMID TESTING

Again this advice is not limited to Angular.

Write more unit tests. Treat unit tests as front-line defense. Use e2e protractor tests as high-level tests for verifying critical happy and error paths.

Martin Fowler discussed [Test pyramid](http://martinfowler.com/bliki/TestPyramid.html) in his blog.

![Test Pyramid](/img/pyramid.png)

<br />


## NG-MODEL DOT RULE

Misko Hevery in his presentation on Angular best practices coined the [dot rule](https://www.youtube.com/watch?feature=player_detailpage&v=ZhfUv0spHCY#t=1758s). My over-simplified version is to bind objects instead of primitives to $scope, and always initialize the bound object:

~~~js
function MainController($scope) {
  $scope.something = {}; // Always initialize the bound objects

  $scope.person = {
    name: 'David Cai'
  }; // Bind objects to $scope instead of $scope.name = 'David Cai';
}
~~~

In the HTML template:

~~~html
<div ng-controller="MainController">
  <input ng-model="person.name">
</div>
~~~

<br />


## ONE COMPONENT PER FILE

Limit one Angular component (directive, service, controller, etc.) per JavaScript file. Make it easy to locate components, and keep files shorter.

~~~js
angular
  .module('myApp')
  .directive('myAwesomeWidget', function() { ... })

  // Don't.
  // We already defined a directive in this file.
  // Move the following directive to another file.
  .directive('myOtherAwesomeWidget', function() { ... })
;
~~~

<br />


## GROUP FILES BY FEATURES

There are two schools of thoughts when it comes to organizing Angular files - grouping by types, and grouping by features. I prefer the latter for big projects. Much easier for navigation.

~~~plain
common
  - util.filter.js
  - util.filter.spec.js
action-panel
  - action-panel.directive.js
  - action-panel.directive.spec.js
  - action-panel.html
  - action-panel.scss
~~~

<br />


## INCLUDE INTENTION IN FILE NAMES

.js can mean a lot of things in Angular apps. It makes the intention much clear when the file names contain .controller, .directive, .spec, .service, etc.

Shai Reznik recommends using .ctrl for controllers, .drv for directives, .srv for services., .tpl for templates, and .test for unit tests. I prefer full names, e.g. .controller. I also feel that HTML files don't need .tpl or .template, since in most cases, HTML files are templates. No need to explicitly call them out as templates.

<br />


## LINT JAVASCRIPT

Use jshint, JSCS, or even better - [eslint](http://eslint.org/) to lint javascript code. Integrate lint into the build process. Use editor plugins to bring lint warnings and errors right in front of developers's eyes. I've been considering to create GIT hook to block git pushes if lint reports any warnings or errors.

<br />


## USE NG-ANNOTATE

Angular replies on the names of function arguments for dependency injection. However, JS minification will shorten argument names, which ruined Angular DI. To work around this issue, we either use the DI array syntax:

~~~js
// DI array syntax
app.controller('MainController', ['$scope', '$log', function($scope, $log) {
  ...
}]);
~~~

... or assign dependencies to the $inject property:

~~~js
// $inject syntax
MainController.$inject = ['$scope', '$log'];
function MainController($scope, $log) {
  ...
}
~~~

These workarounds look like patchwork to me. Not to mention that both are error prone where you have to maintain the dependency order in arrays.

Here comes [ngAnnotate](https://github.com/olov/ng-annotate) - an Angular plugin that automatically inserts the DI array syntax in source code:

~~~js
// ngAnnotate will replace function($scope, $log) { ... } with
// ['$scope', '$log', function($scope, $log) { ... }]
app.controller('MainController', function($scope, $log) {
  ...
});
~~~

Although ngAnnotate is quite smart to figure out where to annotate DI syntax, in some cases, ngAnnotate might miss the DI annotation for a function. You can prepend a function with `/*@ngInject*/` to explicitly state that the function should get annotated.

<br />


## LEVERAGE TASK RUNNERS

Lint and ngAnnotate can be integrated in the build process by task runners such as Gulp.

Use Gulp to automate:

* JS lint - [gulp-eslint](https://github.com/adametry/gulp-eslint)
* ngAnnotate - [gulp-ng-annotate](https://github.com/Kagami/gulp-ng-annotate)
* Sort angular files - [gulp-angular-filesort](https://github.com/klei/gulp-angular-filesort)
* Convert HTML templates to JS strings in $templateCache - [gulp-angular-templatecache](https://github.com/miickel/gulp-angular-templatecache)

<br />
