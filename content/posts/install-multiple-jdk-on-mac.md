---
title: "Install Multiple Java Versions on Mac"
date: "2015-11-22"
draft: true
description: "Install multiple Java versions on Mac using Homebrew and JEnv."
categories:
  - "java"
  - "mac"
hero: "/img/ab-testing.jpg"
heroalt: "A/B testing"
---

## References

```
brew install caskroom/cask/brew-cask
brew update && brew cask install java

brew install jenv
jenv add /Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk/Contents/Home/
jenv add /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/

jenv versions
jenv global oracle64-1.7.0.79
cd <my project>
jenv local oracle64-1.8.0.66
```

Add the following line to `~/.bash_profile`:

```
if which jenv > /dev/null; then eval "$(jenv init -)"; fi
```

.java-version file

http://stackoverflow.com/questions/26252591/mac-os-x-and-multiple-java-versions
http://hanxue-it.blogspot.com/2014/05/installing-java-8-managing-multiple.html
http://andrew-jones.com/blog/managing-multiple-versions-of-java-on-os-x/
