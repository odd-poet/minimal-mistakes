---
layout: single
title: "ssh ProxyCommand"
date: 2013-01-29 11:09
comments: true
tags: [ssh]
---


ssh로 서버에 접근할 때 많은 회사들이 Gateway를 통해서만 접속이 가능하도록 제한한다.
ssh의 ProxyCommand 설정을 이용하여 Gateway에 접속하는 과정을 줄여보자.

<!-- more -->

ssh key 추가
-----------

로컬 머신의 공개키(e.g. `~/.ssh/id_rsa.pub`)를 Gateway서버와 접근하려는 서버의
`~/.ssh/authorized_keys` 파일에 추가한다. `~/.ssh/authorized_keys` 파일이 없으면
생성해서 추가한다. 단, 파일 퍼미션은 600으로 준다.

ssh config
---------

로컬 머신에 ~/.ssh/config 파일에 아래와 같은 내용을 넣는다.

```
Host target.server
   User irteam
   ProxyCommand ssh oddpoet@gateway.server nc %h %p

Host *.dev.server
   User irteam
   ProxyCommand ssh oddpoet@gateway.server nc %h %p

```

* `Host` : 특정 서버를 지정한다. '*'로 이름패턴을 사용할 수도 있다.
* `User` : 타겟서버에 로그인할 계정이름이다.
* `ProxyCommand` : Gateway 서버 호스트명과 계정명을 사용한다.

다음과 같이 직접 접근하는 것처럼 사용할 수 있다.

``` bash
$ ssh target.server # 타겟서버에 접속 (irteam 계정)
$ ssh irteamsu@target.server # 계정을 별도 지정 (irteamsu 계정)
```

netcat을 쓰지 않는 방법들
----------

### OpenSSH >= 5.4

OpenSSH 5.4 이상을 쓰는 경우 `-W` 옵션을 이용해서 SSH의 netcat mode를 쓸 수 있다.

```
Host target.server
   User irteam
   ProxyCommand ssh oddpoet@gateway.server -W "%h:%p"
```


### netcat이 없을 경우

Gateway 서버에 `netcat`이 설치되어 있지 않은 경우에는 아래처럼 설정할 수 있다.

```
Host target.server
   User irteam
   ProxyCommand ssh oddpoet@gateway.server "/bin/bash -c 'exec 3<>/dev/tcp/%h/%p; cat <&3 & cat >&3;kill $!'"
```
