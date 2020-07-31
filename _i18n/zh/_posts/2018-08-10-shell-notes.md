---
layout: post
title: Shell 随笔
tags: Notes
categories: Dev
---

## Shell 随笔

---

#### 1. [显示隐藏文件](#show-hidden-files)
#### 2. [Mac文件安装提示文件已损坏解决方法](#p2)
#### 3. [XCode插件更新](#xcode-plugin)
#### 4. [命令别名](#command-alias)
#### 5. [Bower的使用](#bower)
#### 6. [卸载Nodejs](#uninstall-nodejs)
#### 7. [Git命令](#git)
#### 8. [搜索命令历史记录](#cmd-history)

---

<br>

> #### 1. 显示隐藏文件 {#show-hidden-files}

用以下命令来显示系统隐藏文件

```shell
$ defaults write ~/Library/Preferences/com.apple.finder AppleShowAllFiles -bool true
```

撤销命令, 不再显示隐藏文件

```shell
$ defaults write ~/Library/Preferences/com.apple.finder AppleShowAllFiles -bool false
```

---

<br>

> #### 2. Mac文件安装提示文件已损坏解决方法 {#p2}

```shell
$ sudo spctl --master-disable
```

---

<br>

> #### 3. XCode插件更新 {#xcode-plugin}

XCode所有的插件都安装在目录 `~/Library/Application Support/Developer/Shared/Xcode/Plug-ins/` 下，如果你更新了Xcode，打开XCode时会出现一个提示询问你是否重新安装插件。

不幸的是如果你跳过了重新安装插件，这个提示则再也不会出现，哪怕你重新安装了 **Alcatraz** 也不行。要想重新安装插件, 则可以手动按照以下步骤去做。

**1).** 先删除上一个安装的XCode版本的白名单, 即XCode版本需要对应更新之前的。例如我之前安装的XCode版本为 6.3.2

```shell
$ defaults delete com.apple.dt.Xcode DVTPlugInManagerNonApplePlugIns-Xcode-6.3.2
```

**2).** 获取当前当前新版本XCode的UUID -- `DVTPlugInCompatibilityUUID` 。

```shell
$ defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUID
```

**3).** 给每一个插件增加当前新版本XCode的UUID。即上一步操作得到的例如这样的字符串 `9F75337B-21B4-4ADC-B558-F9CADF7073A7`。

```shell
$ find ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins -name Info.plist -maxdepth 3 | xargs -I{} defaults write {} DVTPlugInCompatibilityUUIDs -array-add 9F75337B-21B4-4ADC-B558-F9CADF7073A7
```

<br>

也可以将三步合并一个语句来做, 直接输入以下命令行

```shell
find ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins -name Info.plist -maxdepth 3 | xargs -I{} defaults write {} DVTPlugInCompatibilityUUIDs -array-add `defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUID`
```

---

<br>

> #### 4. 命令别名 {#command-alias}

我们可以给某个命令自定义一个名字。

比如，你可以将更新pods的命令行操作设置一个别名为 `pod_update`

```shell
$ alias pod_update='pod update --verbose --no-repo-update'
```

当之后再更新pods时，就可以直接使用 `pod_update` 来执行

```shell
$ pod_update
```

---

<br>

> #### 5. Bower的使用 {#bower}

Bower 是 Twitter 推出的一款web开发包管理工具。 它可以安装你所需要的对应版本的包以及其相关依赖，也可以管理HTML，CSS Javascript，字体甚至图片文件。

**安装Bower:** 

安装Bower前需要预先安装 `node`, `npm` 和 `git`

```shell
$ npm install bower
```

**Bower的使用:** 

以管理jquery库举例

```shell
//  安装jquery库

$ bower install jquery  
$ bower install jquery#1.7.2 //  安装1.7.2版本jquery库
```

```shell
//  升级jquery库

$ bower update jquery
```

```shell
//  卸载jquery库 

$ bower uninstall jquery     
```

---

<br>

> #### 6. 卸载Nodejs {#uninstall-nodejs}

**1).** 前往 `/usr/local/lib` 文件夹下删除node及其模块

```shell
$ cd /usr/local/lib
$ sudo rm -rf node*
```

**2).** 前往 `/usr/local/include` 文件夹下删除node及其模块

```shell
$ cd /usr/local/include
$ sudo rm -rf node*
```

**3).** 如果你安装了brew，可以使用在终端中用brew卸载node

```shell
$ brew uninstall node
```

**4).** 前往 `/usr/local/bin` 文件夹下删除node及其模块

```shell
$ cd /usr/local/bin
$ sudo rm -rf /usr/local/bin/npm
$ sudo rm -rf /usr/local/bin/node
$ ls -las
```

**5).** 你可能还需要执行以下命令：

```shell
$ sudo rm -rf /usr/local/share/man/man1/node.1
$ sudo rm -rf /usr/local/lib/dtrace/node.d
$ sudo rm -rf ~/.npm
```

---

<br>

> #### 7. Git命令行 {#git}

**1). 硬回退到某一版本**

```shell
$ git reset --hard version-number
```

---

<br>

> #### 8. 搜索命令历史记录 {#cmd-history}

用以下命令来搜索历史命令行记录

```shell
$ ctrl-r
```
