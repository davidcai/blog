---
draft: true
title: "Angular Factory Pattern"
date: "2015-08-05"
description: "Write better factories."
categories:
  - "angular"
hero: "/img/factory.jpg"
heroalt: "Factory"
---

## Suffix Factory with 'Service'

The terminology "Factory" used in Angular is somewhat different from the one in the Java community. From a Java developer's perspective, a factory is a singleton to create something by the specification which the factory caller supplies. There should be only one public method on a factory - getInstance(). Well, maybe some variations of the getInstance method. But that should be it. Factories simply create stuff.

However, in Angular, unlike `angular.controller` which creates a controller, what `angular.factory` creates is not a factory but a service - a place to host public APIs and encapsulate logics. It certainly could be used to create things, however, the most common usage is to wrap shared code and expose them to consumer code.

That's the reason why I would name a factory abcService in Angular, just to eliminate the confusion.

~~~js
angular
  .module('myApp')
  .factory('userService', userService) // userService created by the factory function
;
~~~

<br />


## Decouple Interface from Implementation

A common factory looks like this:

~~~js
//
// Not so good
//
angular
  .module('myApp')
  .factory('userService', userService)
;

function userService($http, $log) {

  // Private variables
  var REST_URL = '/api/users';

  // Return the service instance
  return {

    // Public methods

    getUsers: function() { ... },
    queryUsers: function(by) { ... }
  };
}
~~~

Nothing is really wrong with the above example. However, there is room for improvement - the returned service instance is a congealment of both interface and implementation. `getUsers` and `queryUsers` are interface, while their associated anonymous functions are implementation details. If the functions grow in lines of code, it would be "a bit" difficult to tell the service interface from implementations. A common practice is to move the implementations out:

~~~js
//
// Better
//
function userService($http, $log) {

  ...

  // Implementation
  var getUsers = function() { ... };
  var queryUsers = function(by) { ... };

  // Interface
  return {
    getUsers: getUsers
    queryUsers: queryUsers
  };
}
~~~

<br />


## Move Declarations to Top

To push good readability further, it will be nice to have interface declarations precede implementations, just like how we define angular components:

~~~js
//
// Good example
//
angular
  .module('myApp')
  // Component declarations. We have two controllers
  .controller('MainController', MainController)
  .controller('SideBarController', SideBarController)
;

// Component implementations
function MainController() { ... }
function SideBarController() { ... }
~~~

This way, we have a manifest of what this Angular script will create right in front. Then, component implementations follow behind. Apply the same principle to factories:

~~~js
//
// Improved
//
function userService($http) {

  // Declarations
  return {
    getUsers: getUsers,
    queryUsers: queryUsers
  };

  // Implementations
  // Use function declaration syntax instead of
  // var getUsers = function() { ... };
  function getUsers() { ... }
  function queryUsers(by) { ... }
}
~~~

Here we use function declaration syntax to define the implementation functions so they will be hoisted and accessible even before they are defined.

It's also recommended to have variables defined at top of the function body:

~~~js
//
// Improved
//
function userService($http) {

  // Declarations
  var var1 = 1;

  return {
    getUsers: getUsers,
    queryUsers: queryUsers
  };

  // Implementations
  function getUsers() { ... }
  function queryUsers(by) { ... }
}
~~~

<br />


## Conclusion

Suffix factories with 'Service'. Leverage function hoisting to pull declarations above implementations.

~~~js
;(function() {
  'use strict';

  angular
    .module('myApp')
    .factory('userService', userService) // Declaration
  ;


  function userService($http, $log) { // Implementation

    // Declaration

    var REST_URL = '/api/users';

    return {
      getUsers: getUsers,
      queryUsers: queryUsers
    };

    // Implementation

    function getUsers() { ... }

    function queryUsers(by) { ... }
  }

})();
~~~

Readability again is one of the keys to having maintainable code :)

<br />
