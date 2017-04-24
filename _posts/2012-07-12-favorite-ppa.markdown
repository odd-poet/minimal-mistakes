---
layout: single
title: "favorite-ppa"
date: 2012-07-12 17:20
comments: true
tags: [ubuntu]
---

Ubuntu 12.04 LTS 기준이다.

	# install add-apt-repository
	$ sudo apt-get install python-software-properties

	# add ppa for oracle java
	$ sudo add-apt-repository ppa:webupd8team/java

	# add pass for airvideo server
	$ sudo add-apt-repository ppa:rubiojr/airvideo

이제 oracle jdk 및 airvideo server를 설치할 수 있다.

	$ sudo apt-get update
	$ sudo apt-get install oracle-jdk7-installer
	$ sudo apt-get install airvideo-server
