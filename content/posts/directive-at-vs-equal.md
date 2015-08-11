---
title: "Directive: @ vs. ="
date: "2015-08-12"
draft: true
categories:
  - "angular"
hero: "img/vs.jpg"
heroalt: "@ vs. ="
---

@ or =? Their difference is very well documented in Angular's official site. However, I often find people asking the same question whenever creating Angular directives. Hope this article will clarify the difference.

<br />


## @: one-way binding

@ is for one-way binding - the bound value is passed into a directive's scope, however, any change made to the value inside the directive will not be passed back from the directive. To help myself remember this syntax, I'd like to think @ as 'point at' - a one-way direction - from the external scope to a directive's internal scope.

For example, we have an awesome-widget directive:

~~~html
<div ng-controller="MainController">
  <awesome-widget name="{{ person.name }}"></awesome-widget>
</div>
~~~

We bind `{{ person.name }}` to the directive through the `name` attribute. The value of `person.name` comes from a person object that is attached to the controller's scope:

~~~js
// A controller
function MainController($scope) {
  $scope.person = { name: 'David' };
}
~~~

Inside the directive, the @ symbol is to indicate that name will be one-way bound to the directive's scope variable (scope.name).

~~~js
// A directive
function awesomeWidget() {
  return {
    ...
    scope: {
      name: '@' // One-way binding
    },
    link: function(scope) {
      // Here we change the name variable on the directive scope
      scope.name = 'Serena';
    }
  }
}
~~~

If we change the name variable inside the directive (e.g., inside the link function), the changed name will not be passed back from the directive's scope to the controller's scope.

~~~html
<div ng-controller="MainController">
  <awesome-widget name="{{ person.name }}"></awesome-widget>

  <!-- In the controller's scope, person.name will still be "David" instead of "Serena" -->
  {{ person.name }}
</div>
~~~

<br />


## =: two-way binding

= is for two-way binding, meaning the change made to the bound value will be "sync'd" both inside and outside of a directive. I'd like to think the "=" symbol as "bi-directional".

Using the above example, we change "@" to "=":

~~~js
// A directive
function awesomeWidget() {
  return {
    ...
    scope: {
      name: '=' // Two-way binding
    },
    link: function(scope) {
      scope.name = 'Serena';
    }
  }
}
~~~

We also need to change `{{ person.name }}` to `person.name` in HTML (I will explain the reason later):

~~~html
<div ng-controller="MainController">
  <awesome-widget name="person.name"></awesome-widget>

  <!-- person.name is changed to "Serena" -->
  {{ person.name }}
</div>
~~~

Now, the name both inside and outside the directive is changed to "Serena".

<br />


## =: bind variables

Have you ever wondered why some Angular directives require using quotes to bind a string, while other directives don't? For example, the following `ngInclude` directive uses single quotes to wrap a path:

~~~html
<ng-include src="'app/content.html'" /> <!-- Why the single quotes? -->
~~~

The single quotes are needed because `src` in ngInclude is bound using the = symbol. Besides two-way binding, "=" introduced another interesting characteristic: "=" symbol is used to bind variables that need to be "eval'd".

Let's say we have a `templateUrl` variable attached to a scope in a controller:

~~~js
function MainController($scope) {
  $scope.templateUrl = 'app/content.html';
}
~~~

Then, we can bind the variable `templateUrl` to ngInclude:

~~~html
<ng-include src="templateUrl" />
~~~

In this case, `src` points to the `templateUrl` variable. When `templateUrl` is eval'd, the resolved value will be "app/content.html".

If we want to bind a string primitive to `src`, we have to wrap the string in single quotes, so when src is eval'd, the eval'd value will be the string value:

~~~html
<ng-include src="'app/content.html'" />
~~~

For numbers and booleans, they can be passed directly to =-bound values:

~~~html
<awesome-widget age="26" single="false"></awesome-widget>
~~~

Angular is smart enough to figure out the type:

~~~js
function awesomeWidget($log) {
  return {
    ...,
    link: function(scope) {
      $log.log(typeof scope.age); // number
      $log.log(typeof scope.single); // boolean
    }
  };
}
~~~

<br />


## @: bind interpolations

"@" symbol, on the contrary, is used to bind interpolations. Interpolations are {{ }} expressions.

In our examples in the "@: one-way binding" and "=: two-way binding" sections, we use interpolation `{{ person.name }}` for the @-bound value:

~~~html
<!-- Use interpolation for @-bound values -->
<awesome-widget name="{{ person.name }}"></awesome-widget>
~~~

... While for =-bound value, we use variable `person.name` instead of interpolation `{{ person.name }}`:

~~~html
<!-- Use variable for =-bound values -->
<awesome-widget name="person.name"></awesome-widget>
~~~

We can also assign primitives directly to the @-bound values:

~~~html
<awesome-widget age="26.5" single="false"></awesome-widget>
~~~

For strings, unlike =-bound values, single quotes are not needed:

~~~html
<awesome-widget name="David"></awesome-widget>
~~~

<br />


## Conclusion

| Type   | Direction       | Value type                    | String value             |
| :----- | :-------------- | :---------------------------- | :----------------------- |
|  @     | One-way binding | Interpolations and primitives | No single quote: "David" |
|  =     | Two-way binding | Variables and primitives      | Single quote: "'David'"  |

<br />
