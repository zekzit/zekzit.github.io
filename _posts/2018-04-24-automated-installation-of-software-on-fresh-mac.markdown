---
layout:	post
title:	"Automated installation of software on fresh Mac"
date:	2018-04-24
tags: [automation, macos]
---

  In this moment of time, I have to swap my MBA with my girlfriend’s MBP. So, I think it would be better if I can use some tools to automatically install some essentials software on a new Mac. Here is a method of mine:

Homebrew is a tool that I’ve chosen, it is a package manager on Mac which can be install some software for me. You can find more information of Homebrew on <http://brew.sh/>. And its extension is Homebrew Cask, more information <https://caskroom.github.io/>.

But, there is a new way better than input each line of brew command separately. It is Brewfile. (<https://github.com/Homebrew/homebrew-bundle>, <https://robots.thoughtbot.com/brewfile-a-gemfile-but-for-homebrew>) I just create Brewfile and run brew bundle command. Everything specify in Brewfile will be install.

Here is my [Brewfile](https://gist.github.com/zekzit/67213156aeb4f92be293697aa31951b7)

Here is a good example for more automated Mac install:

* <https://gist.github.com/millermedeiros/6615994>
* <https://mattstauffer.co/blog/setting-up-a-new-os-x-development-machine-part-2-global-package-managers>
More Mac settings I expected to do:

* Never sleep while on power adapter
* Tap to click
* Three-finger drag
* Put display to sleep on upper left hot-corner
  