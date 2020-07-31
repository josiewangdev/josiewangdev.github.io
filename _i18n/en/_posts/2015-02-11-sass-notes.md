---
layout: post
title: Sass Notes
tags: Notes
categories: Dev
description: Use Sass to implement blog's css style
---

## Sass Notes

---

#### 1. [Syntax](#syntax)
#### 2. [Varibles](#varibles)
#### 3. [Mixin](#mixin)
#### 4. [Use Sass in Jekyll](#sass-in-jekyll)
#### 5. [Word wrap in pre tag](#)

---

<br>

> #### 1. Syntax {#syntax}

```sass
ul {
    list-style: disc;
    li {
        font-size: $content-font-size;
        line-height: 2rem;
        font-family: $content-font-family;
        a {
            color: $sidebar-item-color;
            &:hover {
                color: $link-hover-color;
            }
        }
    }
}
```

---

<br>

> #### 2. Varibles {#varibles}

Sass variables are simple. 

You assign a value to a name that begins with `$` , and then you can refer to that name instead of the value itself. Variables make it possible to reduce repetition. 

Also you can define a variables file to keep all the values, such as color, font, etc. To use the file, just `@import` it in the other style file.

```sass
//  vars.scss

$bg-color:      #ffffff;
$title-color:   #f8f8f2;
$content-color:	#272822;
$border-color:	#e1e1e1;  
```

```sass
//  main.scss

@import "vars";
h1 {
    color: $title-color;
}
```

---

<br>

> #### 3. Mixin {#mixin}

Mixin can distribute collections of styles in libraries, like a snippet of code, it's easy to avoid using non-semantic classes. 

Also it can be re-used throughout your stylesheet. 

To use it, just use `@mixin` to conbine the style code and `@include` it in other context.

```sass
@mixin border-radius($radius) {
    -webkit-border-radius: $radius;
    -moz-border-radius: $radius;
    -khtml-border-radius: $radius;
    border-radius: $radius; 
}
a {
    @include border-radius(10px);
}
```

---

> #### 4. Use Sass in Jekyll {#sass-in-jekyll}

Jekyll provides built-in support for Sass and can translate Sass file to a Css file. 

But, in order to use them, you must first create a file with the proper extension name (one of `.sass`, `.scss`, or `.coffee`) and start the file with two lines of triple dashes like `===`.

```sass
//  main.scss
---
//  此处为预处理头
---

body {
    ...
}
a {
    ...
}
```

---

<br>

> #### 5. [Word wrap in pre tag](#)

In blog, we usually use `<pre></pre>` to display the code block. But sometimes the code is so long that I use the styles below to wrap it and make it multiple lines.

```scss
pre {
    white-space:pre-wrap; /* css3.0 */
    white-space:-moz-pre-wrap; /* Firefox */
    white-space:-pre-wrap; /* Opera 4-6 */
    white-space:-o-pre-wrap; /* Opera 7 */
    word-wrap:break-word; /* Internet Explorer 5.5+ */
}
```






