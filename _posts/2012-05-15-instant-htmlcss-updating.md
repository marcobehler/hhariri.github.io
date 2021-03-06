---
layout: post
title: Instant HTML/CSS Updating
categories:
- JetBrains
tags:
- HTML
- JetBrains
- WebStorm
status: publish
type: post
published: true
meta:
  reddit: a:2:{s:5:"count";i:0;s:4:"time";i:1368855287;}
  _wpas_skip_twitter: '1'
  _wpas_skip_fb: '1'
  _elasticsearch_indexed_on: '2012-05-15 04:40:51'
comments: true
---
Want instant updating to HTML / CSS pages you're editing?

<a href="http://www.screenr.com/L3K8">See it in action</a>

You can now have it. I've tried it on both IntelliJ IDEA and WebStorm successfully. It should most likely work on our other IDE's too (PhpStorm, PyCharm).

To install:

1. <a href="http://plugins.intellij.net/plugin/?&amp;id=7006">Download Web Browser</a> Connector plugin for JetBrains plugin repo. This is needed for the actual Instant HTML editing plugin.

2. <a href="http://plugins.intellij.net/plugin/?&amp;id=7007">Download Instant HTML editing</a> plugin.

3. Open up WebStorm (or IDEA) and click on Preferences. Search for 'plugin'

<img title="plugin" src="{{ site.images }}/html-1.png" alt="" width="393" height="222" />

4. Install the plugins previously downloaded from disk. First install the Web Browser Connector*

<img class="alignnone size-full wp-image-2555" title="pluginfile" src="{{ site.images }}/html-2.png" alt="" width="503" height="75" />

5. Restart WebStorm (or IntelliJ)

6. Open up your Web Project. Under the Run menu select Instant HTML editing

<img class="alignnone size-full wp-image-2553" title="menu" src="{{ site.images }}/html-3.png" alt="" width="259" height="509" />

7. Launch Chrome**. If all has gone well you should now see this below the URL bar

<img class="alignnone size-full wp-image-2556" title="sample" src="{{ site.images }}/html-4.png" alt="" width="529" height="251" />

As you make changes (to both HTML and CSS) you should now be able to see them updating live in Chrome.

It's still early days of the plugin so there might be quirks with it, but I'm sure Vladmir will appreciate any and all feedback.

* You could in principle just install this directly from WebStorm by clicking on Browse Repositories without having to download. However for some reason, Web Browser Connector is not listed.

** Currently only works in Chrome.
