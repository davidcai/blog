---
title: "A Simple A/B Testing Service"
date: "2015-08-16"
draft: true
description: "A Simple A/B Testing Service and its example."
categories:
  - "angular"
  - "test"
hero: "/img/ab-testing.jpg"
heroalt: "A/B testing"
---

A very simple A/B testing service implemented in Angular -- abTestService:
<!--more-->

~~~js
;(function() {
  'use strict';

  angular
    .module('abTest')
    .factory('abTestService', abTestService)
  ;


  function abTestService() {
    return {
      pick: pick
    };

    function pick() {
      var len = arguments.length;
      if (len === 0) {
        return null;
      }

      // Random number from [0, 1)
      var res = Math.random();

      for (var i = 0; i < len; i++) {
        if (res < (1 / len) * (i + 1)) {
          return arguments[i];
        }
      }
    } // /function pick
  }

})();
~~~

abTestService has only one method -- pick, which is to evenly pick one from many choices:

~~~js
// a or b. 50% vs 50% chances
abTestService.pick('a', 'b');

// apple or orange. 50% vs 50%
abTestService.pick('apple', 'orange');

// a, b, or c. 1/3 vs 1/3 vs 1/3
abTestService.pick('a', 'b', 'c');

// a or b. 2/3 vs 1/3
abTestService.pick('a', 'a', 'b');
~~~

Here is an example of how to use abTestService in UI Router:

~~~js
;(function() {
  'use strict';

  angular
    .module('myApp', ['abTest'])
    .config(configRoute)
  ;


  function configRoute($stateProvider) {
    $stateProvider.state('home', {
      url: '/',
      templateProvider: function(abTestService) { // Inject abTestService
        var result = abTestService.pick('a', 'b'); // Pick a or b

        // home-a.html or home-b.html
        return '<div ng-include="\'app/home-' + result + '.html\'"></div>';
      }
    });
  }

})();
~~~

To better understand the logic behind `templateProvider`, I created this [post](posts/router-dynamic-templates/) to explain it.

<br />