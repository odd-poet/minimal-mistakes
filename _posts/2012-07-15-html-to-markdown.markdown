---
layout: single
title: "HTML-to-Markdown"
date: 2012-07-15 17:14
comments: true
tags: [markdown, html to markdown]
---

기존 블로그를 [octopress]로 이주할 때 사용할 만한 **HTML to Markdown** 변환 툴들.

<!-- more -->

[heckyesmarkdown]
: 웹에서 URL을 입력하면 변환 및 변환결과에 대한 프리뷰를 볼 수 있다.
블로그의 경우 메뉴/사이드바를 제외한 컨텐츠 부분만 변환해준다.

[html2text]
: 파이썬 스크립트.
: [heckyesmarkdown]와 마찬가지로 웹 변환도 제공하지만,
script를 받아서 로컬에서 수행할 수 있다는게 장점. 옮길 포스트가 많을 때 좋을 듯.
: 메뉴/사이드바를 포함한 페이지 전체를 변환한다는게 아쉽다.

둘 다 original [markdown] 포맷만 지원하는듯 하다.
table 등은 정상적으로 변환하지 못한다. 어차피 변환 후 수작업을 피할 수 없으니 큰 문제는 아니다.


[octorpress]:http://octopress.org/
[markdown]:http://daringfireball.net/projects/markdown/
[heckyesmarkdown]:http://heckyesmarkdown.com/
[html2text]:http://www.aaronsw.com/2002/html2text/
