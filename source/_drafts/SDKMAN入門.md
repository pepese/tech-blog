---
title: SDKMAN入門
tags:
id:
---

Java 系の anyenv 、 [SDKMAN](http://sdkman.io/index.html) をさわってみる。

<!-- more -->

# 概要

SDKMAN を導入することで、以下のツールのバージョン管理が可能となる。

- Ant
- AsciidoctorJ
- Ceylon
- CRaSH
- Gaiden
- Glide
- Gradle
- Grails
- Griffon
- Groovy
- GroovyServ
- Java
- JBake
- Kobalt
- Kotlin
- kscript
- Lazybones
- Leiningen
- Maven
- sbt
- Scala
- Spring Boot
- Sshoogr
- Vert.x

ここでは Java と Maven のみ記載する。

# 環境設定

## SDKMAN

```sh
$ curl -s "https://get.sdkman.io" | bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

## Java

```sh
$ sdk install java
$ sdk list java
==========================
Available Java Versions
==========================
     9.0.1-zulu
     9.0.1-oracle
     9.0.0-zulu
 > * 8u152-zulu
     8u151-oracle
     8u144-zulu
     8u131-zulu
     7u141-zulu
     6u65-apple
==========================
+ - local version
* - installed
> - currently in use
==========================
$ sdk install java 8u151-oracle
$ sdk default java 8u151-oracle
```

`-zulu` は OpenJDK 、 `-oracle` は Oracle Java を指す。  
`/usr/libexec/java_home` もちゃんと有効。

## Maven

```sh
$ sdk install maven 3.5.2
$ sdk list maven
```
