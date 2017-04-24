---
layout: single
title: "bash-completion"
date: 2012-07-13 01:20
comments: true
tags: [bash, shell script]

---

bash에서 키보드 Tab을 누르면 명령어를 자동완성해주는데
그걸 만드는 방법을 간단히 정리한다.


<!-- more -->

``` bash
	_foo()
	{
	    local cur prev opts
	    COMPREPLY=()
	    cur="${COMP_WORDS[COMP_CWORD]}"
	    prev="${COMP_WORDS[COMP_CWORD-1]}"
	    opts="--help --verbose --version"

	    if [[ ${cur} == -* ]] ; then
	        COMPREPLY=( $(compgen -W "${opts}" -- ${cur}) )
	        return 0
	    fi
	}
	complete -F _foo foo
```

위와 같은 파일을 만든 후에 아래와 같이 실행하자.

``` bash
	$ . sample-completion.sh
```

이제 `foo --[TAB]`하면 `--help`, `--verbose`, `--version`이 자동완성 목록으로 나타난다.

## 작동방식

`_foo(){...}`는 function 정의이고 `complete -F _foo foo`는
`_foo` 함수를 `foo`에 대한 자동완성 처리를 위해 등록하는 명령이다.
즉,  `foo` 이후에 [TAB]을 누르면 _foo 함수가 실행된다.

자동완성 처리 함수에는 입력변수 `COMP_WORD`, `COMP_CWORD`가 전달되고,
자동완성 목록은 `COMPREPLY`로 리턴된다.

COMPREPLY
: shell 함수에 의해서 만들어지는 배열 변수이고
bash가 이 변수를 읽어서 자동완성 목록을 표시한다.

COMP_WORDS
: 현재 명령행에 있는 단어들에 대한 배열 변수이다.

COMP_CWORD
: 현재 커서가 있는 단어의 `${COMP_WORDS}` 상의 인덱스.

함수는 자동완성에 보여줄 목록을 가지고 적당한 조건으로 필터링한 후에 `COMPREPLY` 변수에 담기만 하면 된다.

## 목록 필터링

보통 목록 필터링에는 `compgen` command를 사용한다.

	compgen LIST -- INPUT

compgen은 `LIST` 중에서 `INPUT`으로 시작하는 단어만 돌려주는 command이다.
아래 예를 보라.

	$ compgen -W "apple banana melon apple-iphone" -- app
	apple
	apple-iphone



compgen에 대한 자세한 내용은 [Unix Command: compgen][compgen]을 참고하도록 한다.

## References

* [Bash Completion Part1][bash completion part1]
* [Bash Completion Part2][bash completion part2]


[bash completion part1]: http://www.debian-administration.org/articles/316
[bash completion part2]: http://www.debian-administration.org/article/An_introduction_to_bash_completion_part_2
[compgen]: http://linux.about.com/library/cmd/blcmdl1_compgen.htm
