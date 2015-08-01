---
title: "Just another sample post"
date: "2014-03-29"
description: "This should be a more useful description"
categories:
    - "hugo"
    - "fun"
    - "test"
---

## First Heading

Lorem ipsum dolor sit amet, consectetur adipisicing elit. Accusamus, consequatur aspernatur omnis iste. Voluptates, id inventore ea natus ab sed amet ipsa ratione sunt dignissimos. Soluta illum aliquid repellendus recusandae.


### Sub

Lorem ipsum dolor sit amet, consectetur adipisicing elit. Optio, perferendis saepe voluptatem a nesciunt architecto voluptas deleniti dolor tempora quasi quidem odit rem fugit magnam minima quam dolores vel id?

~~~js
;(function() {
  'use strict';

  angular
    .module('myApp', [
      'ngRouter'
    ])
    .controller('MainController', MainController)
  ;


  function MainController($log, $scope) {
    $log.log('MainController');
    $scope.name = 'David';
  }
})();
~~~

Lorem ipsum dolor sit amet, consectetur adipisicing elit. Accusamus, consequatur aspernatur omnis iste. Voluptates, id inventore ea natus ab sed amet ipsa ratione sunt dignissimos. Soluta illum aliquid repellendus recusandae.

~~~bash
#!/bin/bash

echo -e "\033[0;32mDeploying updates to Github...\033[0m"

# Build the project.
#rm -rf public
hugo

# Add changes to git.
git add -A

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master
#git subtree push --prefix=public git@github.com:spencerlyon2/hugo_gh_blog.git gh-pages

# http://blog.srackham.com/posts/porting-my-blog-with-hugo/
# https://gist.github.com/cobyism/4730490
git subtree push --prefix public origin gh-pages
~~~

~~~html
<div>
  <ul>
    <li><a href="#">abc</a></li>
  </ul>
</div>
~~~


## Conclusion

Lorem ipsum dolor sit amet, consectetur adipisicing elit. Corporis, numquam ipsa ad! Quasi, deleniti quae sint consequatur error corporis dicta inventore alias soluta dignissimos? Molestias, quia ab deserunt repellat ut.
