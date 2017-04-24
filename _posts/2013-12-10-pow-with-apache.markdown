---
layout: single
title: "pow와 apache를 함께 실행하기"
date: 2013-12-10 13:23
comments: true
tags: [pow, apache]
---

OSX에서 개발할 때 호스트 설정 없이 로컬에 구동 중인 서버에 접근하기 위해
[pow]를 애용하는데, pow를 설치하고 나면 pow가 `localhost` 접근을
하이재킹하기 때문에 로컬 호스트의 apache에 접근할 수 없게 된다.
이에 대한 해결책은 다음과 같다.

<!--more-->

```bash
$ curl get.pow.cx/uninstall.sh | sh #if you have pow installed
$ echo 'export POW_DST_PORT=88' >> ~/.powconfig
$ sudo curl https://gist.github.com/soupmatt/1058580/raw/zzz_pow.conf -o /private/etc/apache2/other/zzz_pow.conf
$ sudo apachectl restart
$ curl get.pow.cx | sh
```

그리고 `/etc/apache2/httpd.conf`에 아래 설정이 주석해제되었는지 확인한다.

```
Include /private/etc/apache2/extra/httpd-vhosts.conf
Include /private/etc/apache2/other/*.conf
```

위 설정에 대한 자세한 설명은 아래 링크를 참조한다.

* [Running Pow with Apache][Running Pow with Apache]

[pow]:http://pow.cx/
[Running Pow with Apache]:https://github.com/37signals/pow/wiki/Running-Pow-with-Apache
