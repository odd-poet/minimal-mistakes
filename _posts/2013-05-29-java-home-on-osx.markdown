---
layout: single
title: "Mac OSX에서 JAVA_HOME"
date: 2013-05-29 23:30
comments: true
tags: [java_home, osx]
---

OSX에서 JAVA_HOME 경로는 아래와 같은 커맨드로 얻을 수 있다.

``` bash
> /usr/libexec/java_home
/Library/Java/JavaVirtualMachines/jdk1.7.0_21.jdk/Contents/Home
> /usr/libexec/java_home -v1.6
/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
> /usr/libexec/java_home -v1.7
/Library/Java/JavaVirtualMachines/jdk1.7.0_21.jdk/Contents/Home
```

`.bashrc` 파일에 아래처럼 세팅하면 되겠다.

``` bash
export JAVA_HOME=`/usr/libexec/java_home`
```
