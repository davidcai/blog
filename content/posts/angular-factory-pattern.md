---
title: "Angular Factory Pattern"
date: "2015-08-05"
description: "Write better factories."
categories:
  - "angular"
hero: "/img/factory.jpg"
heroalt: "Factory"
---

## Suffix Factory with 'Service'

The terminology "Factory" in Angular is different from the one used in the Java community. From a Java developer's perspective, a factory is to create something by a specification supplied by the factory caller. There should be only one public method on a factory - getInstance(). Well, maybe some variations of the getInstance method. But that should be it. Factories simply create stuff.

However, in Angular, unlike `angular.controller` which creates a controller, what `angular.factory` creates is not a factory but a service - a place to host public APIs and encapsulate logics, not a factory that creates stuff. Services certainly could be used to create things, however, the most common usage is to wrap shared code and expose them to consumer code.

`angular.factory` returns a service singleton. That's why I would like to suffix the factory function with "Service" not "Factory".

~~~js
angular
  .module('myApp')
  // userService instead of userFactory
  .factory('userService', userService)
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

Nothing is really wrong with the above example. However, there is room for improvement - the returned service instance is a congealment of both interface and implementation. `getUsers` and `queryUsers` are interfaces, while their associated anonymous functions are implementation details. If the anonymous functions grow in lines of code, it would be "a bit" difficult to tell the service interface from implementations. To improve the readability, a common practice is to move the implementations out:

~~~js
//
// Better
//
function userService($http, $log) {

  ...

  // Move implementation here
  var getUsers = function() { ... };
  var queryUsers = function(by) { ... };

  // Keep interface simple
  return {
    getUsers: getUsers
    queryUsers: queryUsers
  };
}
~~~

<br />


## Move Declarations to Top

To push good readability further, I will move declarations above implementations, just like how we define Angular components. Here is an example of Angular controller definition:

~~~js
//
// Good example
//
angular
  .module('myApp')
  // Controller declarations. We have two controllers
  .controller('MainController', MainController)
  .controller('SideBarController', SideBarController)
;

// Controller implementations
function MainController() { ... }
function SideBarController() { ... }
~~~

By moving declarations above Implementations, we have a clear manifest of what this Angular script will do. Apply the same principle to factories:

~~~js
//
// Improved
//
function userService($http) {

  // Declarations, moved up.
  return {
    getUsers: getUsers,
    queryUsers: queryUsers
  };

  // Implementations, moved down.
  // Use function declaration syntax instead of
  // var getUsers = function() { ... };
  function getUsers() { ... }
  function queryUsers(by) { ... }
}
~~~

Here we use function declaration syntax to define the implementation functions, so they will be hoisted and accessible even before they are defined.

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

The same style can be applied to other Angular components, such as directives and filters. Here is a directive example:

~~~js
;(function() {
  'use strict';

  angular
    .module('myApp')
    .directive('myAwesomeWidget', myAwesomeWidget)
  ;


  function myAwesomeWidget($log) {

    // Directive declarations
    return {
      restrict: 'AE',
      scope: {
        options: '='
      },
      templateUrl: 'app/common/myAwesomeWidget.html',
      link: linkMyAwesomeWidget
    };

    // Link function implementation
    function linkMyAwesomeWidget(scope, elem, attrs) {
      ...
    }
  }
})();
~~~

<br />
