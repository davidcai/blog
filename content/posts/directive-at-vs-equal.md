---
title: "Directive Isolated Scope: @ vs. ="
date: "2015-08-13"
description: "Angular Directive Mystery: @ and = in directive isolated scope."
categories:
  - "angular"
hero: "/img/apple-vs-orange.png"
heroalt: "@ vs. ="
---

Shall I use @ or = to bind scope variables? Their difference is very well explained in Angular's documentation. However, this question still often pops up when I create directives. In this post, I listed some of the major differences between these two.
<!--more-->

<br />


## @: Interpolation Binding

"@" symbols can be used to bind interpolations. Interpolations are {{ }} expressions.

Let's say we have a @-bound scope variable -- `name` in a directive:

~~~js
// A directive
function awesomeWidget() {
  return {
    ...
    scope: {
      name: '@' // @-bound variable
    }
  }
}
~~~

We now can assign interpolation `{{ person.name }}` to the @-bound variable `name`:

~~~html
<!-- Use interpolation {{ person.name }} for @-bound variables -->
<awesome-widget name="{{ person.name }}"></awesome-widget>
~~~

However, what will happen if we replace `{{ person.name }}` with `person.name`? Angular will not throw any error, but the value of `scope.name` will be a `'person.name'` string instead of the real value of person.name.

We can also assign primitives (string, number, boolean, and etc.) directly to @-bound values. For example:

~~~html
<awesome-widget name="David" age="26.5" single="false"></awesome-widget>
<!-- Note: "David" here is a string primitive not a variable name -->
~~~

Angular is smart enough to figure out their types:

~~~js
function awesomeWidget($log) {
  return {
    ...
    scope: {
      name: '@', age: '@', single: '@'
    },
    link: function(scope) {
      $log.log(scope.name, typeof scope.name);
      // --> "David", string
      $log.log(scope.age, typeof scope.age);
      // --> 26.5, number
      $log.log(scope.single, typeof scope.single);
      // --> false, boolean
    }
  }
}
~~~

<br />


## =: Variable Name Binding

"=" symbols can be used to bind variable names.

Here, we have a =-bound variable -- `name`:

~~~js
function awesomeWidget() {
  return {
    ...
    scope: {
      name: '=' // =-bound scope variable
    }
  }
}
~~~

Unlike @-bound variables which require interpolations such as `{{ person.name }}`, =-bound variables can directly refer to variable names, e.g. `person.name`.

~~~html
<awesome-widget name="person.name"></awesome-widget>
<!-- Not {{ person.name }} -->
~~~

If we replace `person.name` with `{{ person.name }}`, Angular will throw an error this time:

~~~plain
Error: [$parse:syntax] Syntax Error:
Token '{' invalid key at column 2 of the expression [{{ person.name }}]
starting at [{ person.name }}].
~~~

Primitives can also be assigned to =-bound variables. For instance:

~~~html
<awesome-widget age="26.5" single="false"></awesome-widget>
~~~

However, for strings, single quotes are needed. This is different from @-bound values:

~~~html
<awesome-widget name="'David'"></awesome-widget>
<!-- Note: David is wrapped in single quotes -->
~~~

The single quotes are needed because without the single quotes, Angular will treat "David" as a variable name instead of a string primitive, which might not be what you wanted.

<br />


## @: One-way Binding

@ is for one-way binding - the bound value is passed into a directive's scope, however, any change made to the value inside the directive will not be passed back from the directive. To help myself remember this syntax, I'd like to think @ as 'point at' - a one-way direction - from the external scope to a directive's internal scope.

For example:

~~~html
<div ng-controller="MainController">
  <awesome-widget name="{{ person.name }}"></awesome-widget>
</div>
~~~

We bind `{{ person.name }}` to the directive through the `name` attribute. The value of `person.name` comes from a person object that is attached to the `MainController`'s scope:

~~~js
// A controller
function MainController($scope) {
  $scope.person = { name: 'David' };
}
~~~

In the directive, the @ symbol is to indicate that `name` will be one-way bound to the directive's scope variable `scope.name`.

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

If we change the name variable inside the directive (e.g., in the link function), the changed name will not be passed back from the directive's scope to the controller's scope.

~~~html
<div ng-controller="MainController">
  <awesome-widget name="{{ person.name }}"></awesome-widget>

  <!-- In the controller's scope, person.name will still be "David" instead of "Serena" -->
  {{ person.name }}
</div>
~~~

Inside awesome-widget's `link` function, name is changed from "David" to "Sereana". However, outside of the directive, `person.name` is still "David". This is because the scope variable `name` is one-way bound. The change will not affect the scope outside of its directive.

<br />


## =: Two-way Binding

= is for two-way binding, meaning the change made to the bound value will be "sync'd" both inside and outside of a directive. I'd like to think the "=" symbol as "bi-directional".

Using the above example, we change "@" to "=":

~~~js
// A directive
function awesomeWidget() {
  return {
    ...
    scope: {
      name: '=' // Changed to two-way binding
    },
    link: function(scope) {
      scope.name = 'Serena';
    }
  }
}
~~~

Note that in the HTML template, we also changed `{{ person.name }}` to `person.name`:

~~~html
<div ng-controller="MainController">
  <awesome-widget name="person.name"></awesome-widget> <!-- Removed {{ }} -->

  <!-- Now, person.name is changed to "Serena" -->
  {{ person.name }}
</div>
~~~

The scope variable `name` is changed in directive's `link` function. Because `name` is two-way bound, the change is also reflected in `person.name` on `MainControler`'s scope.

<br />


## Conclusion

| Type   | Direction       | Value type                    | String primitive         |
| :----- | :-------------- | :---------------------------- | :----------------------- |
|  @     | One-way binding | Interpolations and primitives | No single quote: "David" |
|  =     | Two-way binding | Variable names and primitives | Single quotes: "'David'" |

<br />
