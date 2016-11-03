---
layout: post
title:  "Installing PECL on Mac OS X with Homebrew"
date:   2016-08-14 23:53:58 +0000
categories: php programming
---

Recently, I needed to install the inotify extension for PHP in my Homebrew installation on my Mac; to my horror I learnt that Pecl wasn't included. Fortunately I was able to re-install PHP using the ````--with-pear```` extension which also includes Pecl.

{% highlight bash %}
brew remove php71
brew install php71 --with-pear
pecl install inotify
{% endhighlight %}
