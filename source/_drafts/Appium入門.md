---
title: Appium入門
tags:
id:
---


Appium は Selenium WebDriver の一種。  
Node.js 上でサーバーとして動作し、HTTP 経由で WebDriver API を通して操作を受け付けるという仕組み。  
Appium の背後には iOS 用, Android 用, Win 用などのドライバがある。

# 環境設定

```sh
$ brew install node      # get node.js
$ npm install -g appium  # get appium
$ npm install wd         # get appium client
$ appium &               # start appium
$ node your-appium-test.js
```
