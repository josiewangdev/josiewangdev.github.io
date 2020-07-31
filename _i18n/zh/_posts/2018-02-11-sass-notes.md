---
layout: post
title: Sass 随笔
tags: Notes
categories: Dev
description: Use Sass to implement blog's css style
---

## Sass 随笔

---

#### 1. [语法](#syntax)
#### 2. [变量](#varibles)
#### 3. [混合宏](#mixin)
#### 4. [Jekyll项目内使用Sass](#sass-in-jekyll)
#### 5. [pre标签内代码段换行样式](#)

---

<br>

> #### 1. 语法 {#syntax}

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

> #### 2. 变量 {#varibles}

Sass中使用变量非常简单。

你可以以 `$` 开头命名一个值，这个值则可以使用它的名称在其他地方被引用。这样极大的减少了代码的重复性。

同样你也可以定义一个变量文件用来存储公共值，例如颜色与字体等。然后在其他样式文件中通过 `@import` 进行引用。

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

> #### 3. 混合宏 {#mixin}

混合宏可以组合代码片段成为一个宏，类似局部文件，它可以让我们避免使用非语义的类。

同时也可以在其他项目样式文件的任何地方来复用。

通过使用 `@mixin` 来组合代码片段并用 `@include` 在其他地方引用。

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

<br>

> #### 4. Jekyll项目内使用Sass {#sass-in-jekyll}

Jekyll已经内置了对Sass的支持，可以直接将Sass文件预编译为Css文件。

但是，要使用Sass，你在创建Sass样式文件时一定要使用正确的扩展名 （`.sass`, `.scss`, 或者 `.coffee`）并且在文件内容开始时加上预处理头 `===` ，才能生效。


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

> #### 5. [pre标签内代码段换行样式](#)

在博客书写中，我们通常会使用 `<pre></pre>` 来展示代码片段。但是有的时候单行代码过长并且没有折行，所以我用了下面的样式代码使其变成多行展示。

```scss
pre {
    white-space:pre-wrap; /* css3.0 */
    white-space:-moz-pre-wrap; /* Firefox */
    white-space:-pre-wrap; /* Opera 4-6 */
    white-space:-o-pre-wrap; /* Opera 7 */
    word-wrap:break-word; /* Internet Explorer 5.5+ */
}
```






