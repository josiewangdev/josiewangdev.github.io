---
layout: post
title: Shell Notes
tags: Notes
categories: Dev
---

## Shell Notes

---

#### 1. [Show the hidden files](#show-hidden-files)
#### 2. [Search cmd history](#cmd-history)
#### 3. [XCode plugin update](#xcode-plugin)
#### 4. [Command alias](#command-alias)
#### 5. [Usage of Bower](#bower)
#### 6. [Uninstall Nodejs](#uninstall-nodejs)
#### 7. [Git command line](#git)

---

<br>

> #### 1. Show the hidden files {#show-hidden-files}

We can use the following cmd to show the hidden files.

```shell
$ defaults write ~/Library/Preferences/com.apple.finder AppleShowAllFiles -bool true
```

We can use the following cmd to not display hidden files.

```shell
$ defaults write ~/Library/Preferences/com.apple.finder AppleShowAllFiles -bool false
```

---

<br>

> #### 2. Search cmd history {#cmd-history}

```shell
$ ctrl-r
```

---

<br>

> #### 3. XCode plugin update {#xcode-plugin}

All the plugin installed in XCode is kept in `~/Library/Application Support/Developer/Shared/Xcode/Plug-ins/` .

If you update the XCode, it will show you the prompt to ask you whether to reinstall the plugins.

The bad news is that although you choose not to reinstall them, the prompt to load the bundles won’t show again, even if you reinstall **Alcatraz**.

**Solution:**

1). Delete the whitelist / blacklist by running following cmd, and the XCode version should be corresponding to the old one.

```shell
$ defaults delete com.apple.dt.Xcode DVTPlugInManagerNonApplePlugIns-Xcode-6.3.2
``

2). Using cmd below to get the `DVTPlugInCompatibilityUUID` .

```shell
$ defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUID
```

3). Add the current XCode UUID to each plugin by running the cmd below. The UUID is the one you get from the last step. Eg. `9F75337B-21B4-4ADC-B558-F9CADF7073A7`

```shell
$ find ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins -name Info.plist -maxdepth 3 | xargs -I{} defaults write {} DVTPlugInCompatibilityUUIDs -array-add 9F75337B-21B4-4ADC-B558-F9CADF7073A7
```

or input following code directly

```shell
find ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins -name Info.plist -maxdepth 3 | xargs -I{} defaults write {} DVTPlugInCompatibilityUUIDs -array-add `defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUID`
```

---

<br>

> #### 4. Command alias {#command-alias}

For example, you can set a command alias for pods update by using

```shell
$ alias pod_update='pod update --verbose --no-repo-update'
```

When the next time you update the pods, you can use the alias directly.

```shell
$ pod_update
```

---

<br>

> #### 5. Usage of Bower {#bower}

Bower, a package manager of the web, published by Twitter.

It can manage components that contain HTML, CSS, JavaScript, fonts or even image files. 

Bower doesn’t concatenate or minify code or do anything else - it just installs the right versions of the packages you need and their dependencies.

**Install Bower**

Installing Bower requires `node` , `npm` and `git` beforehand.

```shell
$ npm install bower
```

**Install packages**

```shell
$ bower install jquery       //安装jquery库
$ bower install jquery#1.7.2 //安装1.7.2版本jquery库
```

**Update packages**

```shell
$ bower update jquery        //升级jquery库
```

**Uninstall packages**

```shell
$ bower uninstall jquery     //删除jquery库 
```

---

<br>

> #### 6. Uninstall Nodejs {#uninstall-nodejs}

1). Go to `/usr/local/lib` and delete any node and node_modules

```shell
$ cd /usr/local/lib
$ sudo rm -rf node*
```

2). Go to `/usr/local/include` and delete any node and node_modules directory

```shell
$ cd /usr/local/include
$ sudo rm -rf node*
```

3). If you installed with brew install node, then run brew uninstall node in your terminal

```shell
$ brew uninstall node
```

4). Go to `/usr/local/bin` and delete any node executable

```shell
$ cd /usr/local/bin
$ sudo rm -rf /usr/local/bin/npm
$ sudo rm -rf /usr/local/bin/node
$ ls -las
```

5). You may need to do the additional instructions as well:

```shell
$ sudo rm -rf /usr/local/share/man/man1/node.1
$ sudo rm -rf /usr/local/lib/dtrace/node.d
$ sudo rm -rf ~/.npm
```

---

<br>

> #### 7. Git command line {#git}

**1). Hard back to a version**

```shell
$ git reset --hard version-number
```


