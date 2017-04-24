---
layout: single
title: "git 명령어 요약"
date: 2012-10-26 18:31
comments: true
tags:
---

[Pro Git 번역판]을 보다가 자주 안써서 맨날 까먹는 명령들만 정리한다.

[Pro Git 번역판]: http://dogfeet.github.com/articles/2012/progit.html

<!-- more -->

## diff ##

* `git diff`
: staged가 아닌 파일들에 대한 diff 결과 출력

* `git diff --cached`
: staged 파일들에 대한 diff 결과 출력

## log ##

* `git log --pretty=format:"%h %s" --graph`
: 아스키 그래프와 함께 히스토리 출력

* `git log --since=2.weeks`
: 2주 내의 커밋만 조회

## undo  ##

* `git commit --amend`
: 직전 커밋을 수정하는 경우
: 변경된 파일이 없을 경우 직전 커밋의 메시지만 수정
: 변경된 파일이 있을 경우 직전 커밋에 변경 내역 추가 (하나의 커밋으로 처리됨)

* `git reset HEAD <file>`
: staging area의 파일을 unstange.

* `git checkout -- <file>`
: 수정된 파일을 원복

* `git clean -f`
: untracked file 삭제

## remote ##

* `git remote -v`
: 리모트 저장소들을 URL과 함께 출력

* `git remote show origin`
: 특정 원격 저장소의 정보를 출력

* `git remote add <name> <url>`
: 주어진 URL의 리모트 저장소를 추가

* `git remote rename <old_name> <new_name>`
: 원격 저장소 이름 변경

* `git remote rm <name>`
: 원격 저장소를 제거

## tag  ##

* `git tag -a v1.4 -m 'my version 1.4'`
: annotated tag 생성 (-a 옵션)
: 일반 태그와는 달리 메시지와 tag 생성자의 정보도 저장된다.

* `git show <tag_name>`
: tag의 정보 및 커밋 정보 확인

* `git push --tags`
: tag를 원격 저장소로 push

## branch ##

* `git checkout -b iss53`
: `iss533`라는 이름의 브랜치를 생성하고 checkout.
: 아래 명령과 동일

```
    git branch iss53
    git checkout iss53
```

* `git branch -d <branch_name>`
: 브랜치 삭제

* `git branch --merged | --no-merged`
: 현재 브랜치를 기준으로 merge된(혹은 merge되지 않은) 브랜치를 확인.

* `git push <remote> <branch>`
: 리모트 저장소에 특정 브랜치를 push(처음인 경우 원격저장소에 브랜치 생성됨)
: 로컬 저장소의 브랜치와 다른 이름으로 push하려면
`git push <remote> <local_branch>:<remote_branch>`를 사용.

* `git push <remote> :<remote_branch>`
: 원격 브랜치를 삭제.

## rebase ##

rebase는 merge와 달리 두 branch를 선형으로 합친다.

**주의** : 공개 저장소에 push 커밋을 rebase하지 말 것.

* `git rebase <master_branch> <topic_branch>`
: `<topic_branch>`를 `<master_branch>`로 rebase한다.

## 배포  ##

* `git describe master`
: 가장 가까운 annotated tag로 부터 빌드 넘버를 생성한다.
: e.g. `v1.6.2-rc1-20-g8c5b85c`

* `git archive master --prefix='project/' | greph > `git describe master`.tar.gz `
: 소스코드의 snapshot 압축 파일 생성 (이메일 혹은 웹사이트 배포용)

* `git shortlog --no-merges master --not v1.0.1`
: 가장 최근 릴리즈(v1.0.1) 이후의 커밋 요약 정보 출력

## stashing  ##

작업 도중 commit 하지 않고, 브랜치 변경해야 할 때 작업내역 임시 저장.
(modified, staged 모두 보관 )

* `git stash `
: 작업 중인 파일 보관

* `git stash list`
: 저장된 stash 목록 확인

* `git stash apply <stash_name>`
: stash 적용. `stash_name`이 없으면 가장 최근 stash 사용.
: `--index` 옵션을 사용하면 staged 파일들을 staged 상태로 만들어 줌.


## blame  ##

* `git blame -L <begin_line>,<end_line> <file>`
: 각 라인을 누가 수정했는지 확인.
: `-C` 옵션을 주면 파일 이름 출력. (파일 이름이 변경된 경우)


## config ##

* `git config --global user.name "John Doe"`
: 사용자 이름 설정
* `git config --global user.email johndoe@example.com`
: 이메일 설정
* `git config --global core.editor vim`
: 기본 편집기 설정
* `git config --global color.ui true`
: 콘솔 출력에서 컬러 사용
* `git config --global core.autocrlf input`
: MS 윈도우에서의 개행문자가 문제되는 경우
: 개행문자를 윈도우에서는 CRLF 사용. mac, linux, 저장소는 LF 사용함
