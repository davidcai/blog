---
title: "Angular: Copy vs. Extend vs. Merge"
date: "2015-09-09"
description: "Angular Copy vs. Extend vs. Merge"
categories:
  - "angular"
hero: "/img/clones.png"
heroalt: "Clones"
---

Copy, extend, and merge are parts of the utility functions come with Angular. This post explores their differences.
<!--more-->
<br/>

## Copy

`angular.copy` is a *deep* copy.

~~~js
var o1 = { name: 'David', age: 26, skill: {} };
var o2 = angular.copy(o1);

console.log(o2);
// Output: { name: 'David', age: 26, skill: {} }

console.log(o1 === o2);
// Output: false

console.log(o1.skill === o2.skill);
// Output: false
// o2.skill is a copy of o1.skill. They don't point to the same skill object.
~~~
<br/>

## Extend

`angular.extend(dst, src1, src2, ...)` is to *shallow* copy the properties of the source objects from right to left, all the way to the destination object.

~~~js
var src1 = { name: 'David', age: 30 }; // source 1
var src2 = { age: 26, skill: {} }; // source 2
var dst = {}; // destination

console.log(angular.extend(dst, src1, src2));
// Output: { name: 'David', age: 26, skill: {} }

console.log(src2.skill === dst.skill);
// Output: true
// src2 and dst share the same skill object due to shallow copy.
~~~
<br/>

## Merge

`angular.merge` is an Angular 1.4+ API that is to *deep* (recursively) copy the properties of the source objects to the destination object.

Here we use the above example, and replace `angular.extend` with `angular.merge`:

~~~js
var src1 = { name: 'David', age: 30 };
var src2 = { age: 26, skill: {} };
var dst = {};

console.log(angular.merge(dst, src1, src2));
// Output: { name: 'David', age: 26, skill: {} }
// It seems to the same result as the previous example's, however, ...

console.log(src2.skill === dst.skill);
// Output: false
// src2.skill and dst.skill point to different skill objects due to deep copy.
~~~
<br/>

## More Extend vs. Merge

`angular.copy` is simply to clone an object. `angular.extend` and `angular.merge` are more interesting. Here is another set of comparing examples.

`angular.extend` -- shallow copy:

~~~js
var src1 = { skill: { name: 'Java', experience: 20, certified: true } };
var src2 = { skill: { name: 'JavaScript', experience: 10 } };
var dst = {};

console.log(angular.extend(dst, src1, src2));
// Output: { skill: { name: 'JavaScript', experience: 10 } }
~~~

`certified: true` is lost. Because of shallow copy, the skill property is *assigned* from src2 to src1 without *recursively* copying the individual properties inside the skill value.

The same code yields totally different result if we replace `extend` with `merge`:

`angular.merge` -- deep (recursive) copy:

~~~js
var src1 = { skill: { name: 'Java', experience: 20, certified: true } };
var src2 = { skill: { name: 'JavaScript', experience: 10 } };
var dst = {};

console.log(angular.merge(dst, src1, src2));
// Output: { skill: { name: 'JavaScript', experience: 10, certified: true } }
~~~

With `angular.merge`, properties are recursively copied from src2 to src1, so `certified: true` is reserved.

<br/>

## User and Default Options

`angular.extend` or `angular.merge` is often used to achieve user options overriding default options. Here is an example:

~~~js
function orderDrink(options) {

  // Default options
  var defaultOptions = { size: 'medium' };

  // Supplied options override default options
  var opts = angular.extend({}, defaultOptions, options);

  console.log(opts.size);
}

orderDrink({ size: 'large' }); // Output: 'large'
orderDrink(); // Output: 'medium'
~~~

For more complicate case that has nested options, you might want to use `angular.merge` for its deep copy.

<br/>
