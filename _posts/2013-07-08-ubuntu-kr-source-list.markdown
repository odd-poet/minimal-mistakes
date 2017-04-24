---
layout: single
title: "ubuntu-kr-source-list"
date: 2013-07-08 20:39
comments: true
tags: ["ubuntu", "ubuntu mirror"]
---

ubuntu-kr의 source list가 `kr.archive.ubuntu.com`로  되어 있는데, 이 서버가 느린데다가 가끔 접속이 안되기도 한다. daum에서 제공하는 미러링 서버를 사용하면 조금 더 나은 속도로 package 업데이트를 경험할 수 있다.

``` bash
$ vi /etc/apt/source.list
  # vi에서 :%s/kr.archive.ubuntu.com/ftp.daum.net/g 으로 치환한다.
$ sudo apt-get update
$ sudo apt-get dist-upgrade
```
