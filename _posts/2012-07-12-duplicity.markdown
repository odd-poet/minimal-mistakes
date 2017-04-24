---
layout: single
title: "duplicity를 이용한 백업"
date: 2012-07-12 18:03
comments: true
tags: [홈서버, duplicity, ubuntu]
---

증분 백업을 지원하는 간단한 command line 백업 툴인 [duplicity]를 소개한다.

<!-- more -->

## 백업 툴 선택

백업 툴을 선택하는데 고려할 만한 요소는 아래와 같은 것들이 있다.

* **증분 백업(incremental backup)을 지원하는가?** <br/>
	증분 백업은 바뀐 부분에 대한 정보만 저장하므로써 백업 사이즈를 줄여준다.
	보통 daily로 증분 백업, weekly로 full backup을 하게 된다.
	주기적인 백업본이 필요한게 아니라면 rsync만으로도 충분하다.

* **백업 설정 및 복구를 위한 GUI를 제공하는가?** <br/>
	편리한게 좋은거다. 있으면 좋다.

* **백업 수행에 대한 스케쥴을 관리 기능을 제공하는가?** <br/>
	cron을 직접 설정할거냐의 얘기다. 있으면 편하다.

* **다른 시스템의 스토리지에 원격으로 백업할 수 있는가?** <br/>
	백업 파일을 다른 머신에 저장할거냐인데 상황에 따라 중요하다.

duplicity는 incremental backup과 remote backup을 지원하는 command line 툴이다.
GUI와 스케쥴링 혹은 더 많은 기능들이 필요하다면 [백업 유틸리티들][backup tools]의 목록을 보라.

## 설치

	sudo apt-get install duplicity

## 백업

백업은 아래와 같은 command line으로 수행할 수 있다.

	$ duplicity 원본경로 백업경로

백업경로는 url 형식으로 써야한다. 로컬경로의 경우 `file:///path/to/backup` 형태로 쓴다.

예를 들어 `/etc` 디렉토리를 `/backup`로 백업하려면 shell에서 아래와 같이 한다.

	$ export PASSPHRASE=SomeLongGeneratedHardToCrackKey
	$ duplicity /etc file:///backup
	$ unset PASSPHRASE

실행되면 아래와 같은 형태의 결과가 출력될 것이다.

	--------------[ Backup Statistics ]--------------
	StartTime 1342086679.23 (Thu Jul 12 18:51:19 2012)
	EndTime 1342086689.32 (Thu Jul 12 18:51:29 2012)
	ElapsedTime 10.09 (10.09 seconds)
	SourceFiles 2145
	SourceFileSize 3577250 (3.41 MB)
	NewFiles 2145
	NewFileSize 3577250 (3.41 MB)
	DeletedFiles 0
	ChangedFiles 0
	ChangedFileSize 0 (0 bytes)
	ChangedDeltaSize 0 (0 bytes)
	DeltaEntries 2145
	RawDeltaSize 2646680 (2.52 MB)
	TotalDestinationSizeChange 924110 (902 KB)
	Errors 0
	-------------------------------------------------

## 검증

백업과 원본의 비교는 다음과 같이 한다.

	$ export PASSPHRASE=SomeLongGeneratedHardToCrackKey
	$ duplicity verify file:///backup /etc
	$ unset PASSPHRASE

결과는 다음과 같다

	Verify complete: 3503 files compared, 2 differences found.


`verbose` 레벨을 높여서 자세한 결과를 볼 수 도 있다.

	$ duplicity verify -v4 file:///backup /etc
	Difference found: File . has mtime Fri Dec  2 05:58:42 2005, expected Wed Nov 30 11:42:01 2005
	Difference found: File resolv.conf has mtime Fri Dec  2 05:58:42 2005, expected Mon Nov 28 18:58:28 2005
	Verify complete: 3503 files compared, 2 differences found.


## 확인

백업 내용은 아래와 같이 확인할 수 있다.

	$ export PASSPHRASE=SomeLongGeneratedHardToCrackKey
	$ duplicity list-current-files file:///backup
	$ unset PASSPHRASE

파일 목록이 아래처럼 출력된다.

	...
	Sat Nov 12 19:58:23 2005 rcS.d/S70xorg-common
	Sat Nov 12 20:06:57 2005 rcS.d/S75sudo
	Sat Nov 12 20:35:43 2005 readahead
	Mon Nov 28 18:57:23 2005 readahead/readahead
	Mon Nov 28 18:57:23 2005 readahead/readahead.new
	Tue Jan 25 15:03:18 2005 reportbug.conf
	Mon Nov 28 18:58:28 2005 resolv.conf
	Sat Nov 12 19:55:51 2005 resolvconf
	Sat Nov 12 20:06:57 2005 resolvconf/update-libc.d
	Wed Aug 17 21:17:26 2005 resolvconf/update-libc.d/fetchmail
	Mon Jun 27 07:16:38 2005 rmt
	Mon Sep 19 06:41:04 2005 rpc
	Sat Nov 19 13:08:21 2005 samba
	Thu Jul 21 13:31:14 2005 samba/gdbcommands
	Sat Nov 19 13:08:21 2005 samba/smb.conf
	Sat Nov 19 13:05:32 2005 samba/smb.conf~
	Sat Nov 12 20:00:22 2005 sane.d
	Tue Sep 27 09:14:55 2005 sane.d/abaton.conf
	Tue Sep 27 09:14:55 2005 sane.d/agfafocus.conf
	...

## 복구

### 파일 단위 복구

	$ export PASSPHRASE=SomeLongGeneratedHardToCrackKey
	$ duplicity --file-to-restore apt/sources.list file:///backup ~/restore/sources.list
	$ unset PASSPHRASE

### 전체 복구

	$ export PASSPHRASE=SomeLongGeneratedHardToCrackKey
	$ duplicity restore apt/sources.list file:///backup ~/restore/
	$ unset PASSPHRASE

### 특정 시간에 대한 복구

아래는 4일전 파일 내용을 복구한다.

	$ export PASSPHRASE=SomeLongGeneratedHardToCrackKey
	$ duplicity -t4D --file-to-restore apt/sources.list file:///backup ~/restore/sources.list
	$ unset PASSPHRASE

## 기타

백업 기본 기능에 충실한 편이고 설정없이 command line으로 작동하는 툴이라
scripting 및 cron scheduling을 활용하기 용이하다.

## reference

* [Duplicity Backup HOWTO][howto]

[duplicity]: http://www.nongnu.org/duplicity/index.html
[howto]: https://help.ubuntu.com/community/DuplicityBackupHowto
[backup tools]: https://help.ubuntu.com/community/BackupYourSystem
