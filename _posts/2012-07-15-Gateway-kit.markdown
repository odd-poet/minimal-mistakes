---
layout: single
title: "Gateway-kit"
date: 2012-07-15 13:34
comments: true
tags: [bash, shell script, kerberos]

---

회사의 kerberos gateway 서버에서 사용하기 위한 bash script.
<!-- more -->

ruby 사용이 불가능한 환경의 서버라 bash script 로 작업했는데 익숙치 않아서 삽질을 많이함.



Manage host list
: 알고 있는 호스트 목록을 보여주고 커서를 움직여서 선택
: 호스트 이름을 입력하면 목록 필터링됨
: Tab 자동완성 지원
: Bash key map 지원
: 접속에 성공한 호스트는 목록에 자동으로 추가
: 호스트에 대한 설명을 추가할 수 있음.

Automatic kinit
: 비밀번호를 `~/.kinit_passwd` 파일에 써놓으면 실행시
	자동으로 kinit 인증을 통해 ticket을 발급받는다.
: **(주의)** 비밀번호를 plain text로 기록하게 되므로 보안 문제가 있을 수 있다.
: `~/.kinit_passwd` 파일이 없으면 비밀번호를 입력받음.

![](/assets/include/2012-07/gw-kit.png)

스크립트는 gist에 있음. [https://gist.github.com/odd-poet/3115022](https://gist.github.com/odd-poet/3115022)

{% gist odd-poet/3115022 %}
