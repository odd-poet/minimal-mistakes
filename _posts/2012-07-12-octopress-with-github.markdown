---
layout: single
title: "octopress-github-pages-tips"
date: 2012-07-12 00:54
comments: true
tags: [blog, octopress, github]
---

[octopress]로 github에 사이트를 퍼블리싱하는 것과 관련된 몇가지 팁들.

1. static site는 `gh-pages`라는 branch에 push된다.
2. `gh-pages` branch는 github에서 프로젝트 페이지가 있다고 가정하는 브랜치이기도 하다.
3. octopress 및 markdown 문서는 `source` branch로 push하는게 관례인듯. <br/>
	관리편의를 위해서 프로젝트 소스와 동일 repository에
	`gh-pages`, `source` branch를 만들어서 문서 작성 및 퍼블리싱을 함.
4. 메타 정보에 `published: false`를 넣으면 generate 때 페이지가 생성되지 않음.
	(draft 작업시 사용)
5. github 프로젝트가 `https://github.com/USER/PROJECT`라면 `gh-pages` 브랜치에 있는 페이지들은
	`http://USER.github.com/PROJECT`에서 호스팅된다.
6. github repository 이름을 `USERNAME.github.com`으로 만들고 그 repository에 site를 publish하면
	`http://USERNAME.github.com` 으로 호스팅됨. <br/>
	즉, 프로젝트에 대한 사이트가 아니라 사용자(조직)에 대한 사이트가 됨.

[octopress]: http://octopress.com
