---
layout: post
title: "Launch Image for custom URL scheme"
tags: Project
categories: [Tech]
cover: "upload/blog_masonry_03.jpg"
---

<br>
不同的URL scheme进入App展示不同的Launch Image
<br>
<br>
If I want to specify launch images for the custom url scheme "myscheme", I need to use the following naming convention,
 even though I'm already using assets catalog (.xcassets file) for the standard launch image:
<br>

* Default-myscheme~iphone.png --> for no Retina Display 3,5'' iPhones
* Default-myscheme@2x~iphone.png --> for Retina Display 3.5'' iPhones
* Default-myscheme-568h@2x~iphone.png --> for Retina Display 4'' iPhones
* Default-myscheme-Portrait~ipad.png --> for no Retina Display iPads in portrait
* Default-myscheme-Portrait@2x~ipad.png --> for Retina Display iPads in portrait
* Default-myscheme-Landscape~ipad.png --> for no Retina Display iPads in landscape
* Default-myscheme-Landscape@2x~ipad.png --> for Retina Display iPads in landscape

<br>
These files need to be in the app bundle in order to be found for the system when launching the app.
<br>


