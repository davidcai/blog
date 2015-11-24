---
title: "Atom Essential Packages"
date: "2015-11-19"
description: "Atom essential packages for web development"
categories:
  - tool
hero: "/img/atom.jpg"
heroalt: "Atom"
---

Atom -- the hackable editor coming from Github has been my favorite for web development. Like many other lightweight editors, Atom has a community-driven packages (plugins) marketplace that greatly enhanced the editor. I found the following Atom packages are essential to my web development workflow.
<!--more-->
<br/>


## Project-manager

This package is a lifesaver for people like me who often need to work with more than one projects. Project-manager saves your project in a registry. Switching to other saved projects is just one key shortcut away - `cmd + ctrl + p`.

![Project Manager](/img/project-manager.png)

<br/>


## Pigments

When it comes to styling, colors might be the most important factor. It will be nice to see the actual color defined by color values right in the editor. The Pigments package gives you just that. All color values will be decorated with a block of their correspondent colors in the same file. You have options to select different decoration styles. For me, "sqaure-dot" worked the best.

<div class="img-og">
  <img src="/img/pigments.png">
</div>

<br/>


## Highlight-selected

In Atom, I couldn't find a way to highlight all occurrences that matches my current selection. For example, I selected "export" and would like to immediately see all "reports" are highlighted in the current file. I consider this basic functionality should be included in Atom but sadly it isn't. Highlight-selected fixed that. You can choose to highlight the border or the background. The package even comes with two highlight colors - one for dark background, and the other for light.

<div class="img-og">
  <img src="/img/highlight-selected.png">
</div>

<br/>


## EditorConfig

Have you suffered from other developers' bad habits of using a mix of tabs, spaces, charset, trailing white spaces and lines? EditorConfig.org created a standard to promote a consistent coding styles between different editors and IDEs. Their website maintained [a list of plugins](http://editorconfig.org/#download) that support this standard. All you need to do is to define a `.editorconfig` file at the root of your project, and download the plugin. Then, your editor will recognize `.editorconfig` and start to apply the styles.

Here is an example of .editorconfig file:

```
# http://editorconfig.org
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false

```
<br/>


## Tabs-to-spaces

I found editorconfig itself is not enough to convert all tab indentations to spaces. I had to use this tabs-to-spaces package for that. Like the name suggested, this package transforms all tabs indentations to spaces, or vice versa. In the package's setting UI, you have the option to choose "none", "tabify", and "untabify". "none" does nothing; "tabify" changes indentations in spaces to tabs; and "untabify" does the opposite - replacing tabs with spaces.

<br/>


## Linter and linter-eslint

For hobby projects, you might don't care about linting. However, for me, even hobby projects could use some code static analysis. linter and linter-eslint shout out loud whenever (most of the time) I make syntax mistakes. They saved me tons of time that would have been wasted in JavaScript debugging.

![Eslint](/img/eslint.png)

<br/>


## Themes

No matter how bad our art taste are, we developers do care about our color themes in editor, don't we? Since this is a very subjective area, I will leave the theme picking to yourself. However, here I just want to shamelessly introduce to you my own syntax theme majorly emphasizing on green, brown, white, and blue, designed for web app coding - the theme color for Druids in World of Warcraft, hence the name.

![Emerald Dream Syntax](/img/emerald-dream-syntax-screenshot.png)

This theme is not published as an Atom package yet, however, you can still get it from Github:
https://github.com/davidcai/emerald-dream-syntax. Git clone to ~/.atom/packages/ directory, reload Atom, and then select Emerald Dream from the syntax themes dropdown list. This syntax theme works best with Atom Material UI theme.

Well, Emerald Dream or not, choose your own theme. The point is to make your editor more pleasant and effective.

<br/>
