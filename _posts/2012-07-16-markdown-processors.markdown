---
layout: single
title: "markdown processors"
date: 2012-07-16 03:41
comments: true
tags: [markdown, octopress, jekyll]
---

요즘 [octopress]에 꽂혀서 [markdown]관련 툴들을 이것 저것 보고 있는데 뭔가 많다.
프리뷰 지원하는 전용 에디터와 전용 뷰어 등도 구입해서 써보는 중인데
markdown 처리기가 많다보니 적절한 조합을 찾는 것도 쉽지 않다.
이번 포스트에서는 markdown 처리기들을 정리해본다.

<!--more-->

John Gruber의 [Standard markdown][markdown] 외에도 processor 구현체에 따른 확장 문법들이 존재하는데
그 수도 적지 않고 또한 자잘한 차이점들이 있다.

[Standard Markdown][markdown]
: John Gruber가 만든 오리지널이다. 홈페이지에 보면 markdown 문법에 대한 설명과 perl로 만든 html 변환 툴을 제공하고 있다.
: markdown은 간결한게 장점이라지만 definition list, table 등에 대한 syntax가 없는게 개인적으로 아쉽다.

[PHP Markdown Extra][PME]
: 표준 Markdown에 fenced code block, definition list, table, footnote 등의 문법을 확장했다. 이 후 대부분의 markdown processor들이 이것을 언급할 만큼 markdown 확장의 완전판에 가깝다.
PHP 구현체라는 점 때문에 실제 많이 사용되고 있지는 않은듯.

[Multimarkdown]
: HTML 외에도 LaTex, PDF, OpenDocument, OPML 등 다양한 포맷으로 변환을 제공하는 markdown processor다.
: PHP Markdown extra 와 비슷한 수준의 확장 문법을 제공하는 듯.
: 다양한 포맷 지원 외에도 c구현체의 command line 툴이라 많이 사용되는 듯하다.

[Github Flavored Markdown][GFM]
: Github에서 코드 snippet 들을 embed 하기 편하도록 확장한 문법
: syntax highlighting을 위해서 fenced code block 문법에 언어를 명시할 수 있도록 했음

[Maruku]
: 입력으로는 PHP markdown extra 및 기타 확장을 포함 markdown 뿐만아니라 textile까지 지원하고, 출력은 HTML, LaTex, PDF, textile, markdown 을 지원하겠다는 야심찬 툴.
: 많이 쓰이는지는 모르겠음.

[kramdown]
: 이 녀석도 왠만한 확장 문법은 다 지원하고 게다가 ruby 구현체 중에서 제일 빠르다는 markdown processor.
: markdown 관련 툴들 중에 ruby로 만들어진게 많은 편인데 느낌상으로는 이게 앞으로 대세가 될 듯.
: ruby 구현체로는 [BlueFeather], [rdiscount] 등도 있지만 이게 4~5배 빠르다니까 나머지는 생략.

[jekyll]
: markdown 계의 완소툴!
: 단순히 markdown 파일을 html로 변환해주는게 아니라 markdown으로 작성한 문서들을 처리해서 사이트를 만들 수 있게 해준다.
: 요즘 댓글 등의 기능이 사이트 기능이 아니라 SNS 서비스 연동으로 가는게 대세이다 보니 jekyll을 이용해서 DB없이 static한 파일들로 site를 만들고 운영하는게 가능해졌다.
: 보통 jekyll로 사이트를 생성하고 github page에 올려서 블로그를 운영하는게 요즘 geek들의 대세인듯.
: markdown 처리기로 [rdiscount], redcarpet, kramdown 를 사용한다.

[octopress]
: **"Hacker들을 위한 블로그 프레임웍"**이라는 부제를 달고 있는 툴
: [jekyll] 기반으로 만들어진 툴인데 jekyll이 생성한 파일들을 배포하는 기능을 제공한다. GitHub page, Heroku 로의 배포를 지원하며 rsync 이용하여 호스팅 받는 서버로의 배포도 가능하다.
: 검색해보면 Wordpress에서 Octopress로 갈아탄 사람이 꽤나 있는 것 같다 : [25 Reasons to Switch From WordPress to Octopress](http://codebrief.com/2012/01/25-reasons-to-switch-from-wordpress-to-octopress/)
: 물론 다시 Wordpress로 돌아온 사람도 있다 ^^ : [WordPress to Octopress and Back](http://www.kevindangoor.com/2012/03/wordpress-to-octopress-and-back/)


[markdown]:http://daringfireball.net/projects/markdown/
[PME]: http://michelf.com/projects/php-markdown/extra/ (PHP Markdown Extra)
[GFM]:http://github.github.com/github-flavored-markdown/ (Github Flavored Markdown)
[multimarkdown]: http://fletcherpenney.net/multimarkdown/ (Multimarkdown)
[Maruku]:http://maruku.rubyforge.org/
[octopress]: http://octopress.org (Octopress)
[BlueFeather]:http://ruby.morphball.net/bluefeather/index_en.html (BlueFeather)
[rdiscount]:https://github.com/rtomayko/rdiscount/ (rdiscount)
[kramdown]:http://kramdown.rubyforge.org/ (kramdown)
[jekyll]:https://github.com/mojombo/jekyll/wiki/
