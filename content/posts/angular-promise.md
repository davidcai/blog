---
title: "Tips on Promise and $http Service"
date: "2015-09-14"
description: "Simplified promise resolution and rejection. Interplay between Angular Promise and $http Service."
categories:
  - "angular"
hero: "/img/promise.png"
heroalt: "Promise"
---

I've been reviewing a lot of Angular code lately. One common problem I found is the unnecessary use of deferred object to resolve and reject promises. In this post, I will illustrate a simplified approach. Another problem is a mix of then, catch, two callbacks in then, success, and error. I will go through each, and propose adopting a single style that can also be applied when using other Promise libraries.
<!--more-->
<br/>

## Resolve Promise

The following snippet uses $http service to make a REST call, and resolve the returned data from the response. This is a common pattern I often find in Angular code, however, the code is not optimal (see below).

```js
/////////////////////////////////////////////////
// Not preferred. Unnecessary usage of deferred
/////////////////////////////////////////////////
function getUser($q, $http) {
  var deferred = $q.defer();

  $http.get('...').then(function(response) {
    deferred.resolve(response.data);
  });

  return $q.promise;
}
```

The above code can be simplified to this:


```js
//////////////
// Preferred
//////////////
function getUser($http) {
  return $http.get('...').then(function(response) {
    return response.data;
  });
}
```

First, the `then` function accepts three callbacks: successCallback, errorCallback, and notifyCallback. If you return a value in the successCallback or errorCallback, the returned value will be used to *resolve* the promise. You don't have to explicitly call `deferred.resolve(response.data)` to resolve the promise. Instead, simply `return response.data` in `then`'s callback.

Second, the `then` function returns a new promise, so no need to use `$q` to create a deferred object and return its associated promise. Just return the promise created by `then`.

<br/>


## Reject Promise

What about error handling? Using the not preferred example, we can call `deferred.reject` in the error callback to reject the promise:

```js
/////////////////////////////////////////////////
// Not preferred. Unnecessary usage of deferred
/////////////////////////////////////////////////
function getUser($q, $http) {
  var deferred = $q.defer();

  $http.get('...').then(
    function(response) { // Success callback
      deferred.resolve(response.data); // Resolve
    },
    function(response) { // Error callback
      deferred.reject(response); // Reject
    }
  );

  return $q.promise;
}
```

However, a simplified solution is again to use *return* instead of *reject*. But, what shall we return? You probably think to return the response in the error callback like this:

```js
function getUser($http) {
  return $http.get('...').then(
    function(response) {
      return response.data;
    },
    function(response) { // Error callback
      return response; // <<< Wrong
      // Or return false; // <<< Still wrong
    }
  );
}
```

As I mentioned above, values returned from the successCallback or the errorCallback are considered to be resolved values for the promise. So the `return response` in the error callback will resolve instead reject the promise. Even when the returned value is falsy, the falsy value will be considered as a resolved value. The correct way is to return a rejected promise using `$q.reject(response)`:

```js
//////////////
// Preferred
//////////////
function getUser($q, $http) {
  return $http.get('...').then(
    function(response) {
      return response.data;
    },
    function(response) { // Error callback
      return $q.reject(response); // Use $q.reject
    }
  );
}
```

Besides the errorCallback in `then` function, Promise has another method -- `catch` to take an error callback. I personally prefer the `then` and `catch` pair over the two callbacks in `then` for its readability.

```js
///////////////////////
// Catch is preferred
///////////////////////
function getUser($q, $http) {
  return $http.get('...').then(
    function(response) {
      return response.data;
    }
  ).catch( // Catch
    function(response) {
      return $q.reject(response);
    }
  );
}
```

In the above code, `then` is used only for success callback. A `catch` is basically a `then()` without success callback -- `then(angular.noop, errorCallback)`.

The difference from `then(successCallback, errorCallback)` is that `catch` is chained of the promise returned by `then`. However, the errorCallback in `then` is chained of the original promise returned by `$http.get()`.

Please be advised that in IE8 `catch` has special meaning, so instead of the dot syntax, write `['catch']`. Yes, in IE8, `catch` is the catch :)

```js
/////////////////
// Catch in IE8
/////////////////
function getUser($q, $http) {
  return $http.get('...').then(
    function(response) {
      return response.data;
    }
  )['catch']( // Catch
    function(response) {
      return $q.reject(response);
    }
  );
}
```

If you don't need to handle the error in the `getUser` function shown above, you can drop the entire error callback, and let `getUser`'s consuming code to handle the error:

```js
//////////////
// Preferred
//////////////
function getUser($q, $http) {
  return $http.get('...').then(function(response) {
    return response.data;
  });
}

// getUser's consuming code
getUser().then(
  function(data) { // Success callback
    // Do something about data
  },
  function(response) { // Error callback
    // Handle error state
  }
);
```

<br/>


## Success and Error in $http Service

In old versions (1.3 and less) of Angular, $http service has two additional methods -- success() and error(), which are similar to then(), however, the subtle differences are important:

```js
$http.get('')
  .success(function(data) {
    ...
  })
  .error(function(error) {
    ...
  });
```

These two methods are deprecated in Angular 1.4 partially for the confusion they introduced. The first argument passed into `success` and `error` callbacks is not the response object itself, instead, it is the data unwrapped from the response object. However, in `then` callback, you call `response.data` to retrieve the data. Here is a comparative example:

```js
// Success method
$http.get('').success(function(data) {
  $log.log(data);
});

// Then method
$http.get('').then(function(response) {
  $log.log(response.data); // <<< See the difference?
});
```

The second difference is that unlike `then`, neither `success` nor `error` will create and return new promises. The promise they return is the same HTTP promise from `$http.get()`.

Overall, I'd recommend not using `success` and `error`, but adopting `then` and `catch`. The latter can be found in other Promise libraries, and is already evolved to be a convention.

<br/>


## Conclusion

The usage of a deferred object to resolve and reject promises can be replaced by a simplified style:

* Returning a value in success or error callbacks resolves the promise with the returned value.

* Returning a rejected promise using `return $q.reject(something)` rejects the promise.

Prefer using catch to provide error callbacks. Change `.catch` to `['catch']` in IE8.

Stop using `success` and `error`.

<br/>
