---
layout: single
title: "python으로 daemon 만들기"
date: 2013-09-24 23:56
comments: true
tags: ["python", "daemon", "python-daemon"]
---

python에 익숙치 않은데 요즘 python으로 unix-sytle daemon을 만들 일이 있어서 관련 내용을 정리해본다.

<!--more-->
대충 unix daemon은 background에서 수행되고, 중복 실행되지 않으며, pid파일을 만들면 되지
않을까 생각했는데, python에는 'python의 daemon library는 이런 모습이어야 해'하는
"Standard daemon process library"([PEP-3143]) 표준 문서를 제공하고 있다.

[PEP-3143]에서 *Richard Stevens*의 *Unix Network Programming*를 인용하여
unix daemon의 조건을 다음과 같이 언급하고 있다.

- 열려있는 모든 File descriptor를 닫는다.
- 현재 작업 디렉토리를 변경한다
- 파일 생성시 마스크를 재설정한다
- 백그라운드로 실행된다
- 프로세스 그룹에서 분리한다
- 터미널의 I/O 시그널을 무시한다
- 제어 터미널과 분리한다
- 다음과 같은 상황을 올바르게 다룰수 있다
    - System V *init* 프로세스에 의해 시작된다.
    - `SIGTERM` 시그널에 의해 종료된다
    - 자식 프로세스는 `SIGCLD` 시그널을 발생시킨다

하지만 [PEP-3143]에서는 daemon과 service는 다르기 때문에 service의 관점보다는
background process 측면에만 초점을 맞춘다. 즉, initd, inetd 등에 의해 시작되야 한다는
조건은 무시한다는 얘기인 듯 하다.

어쨌든 [PEP-3143] 문서에 대한 reference 구현이 [python-daemon] 패키지다.
위에서 얘기한 daemon process의 기능을 대부분 제공하며, pid 파일관련 기능도 제공한다.

간단한 사용예는 아래와 같다.

```python
#!/usr/bin/env python

import daemon
from spam import main_program

with daemon.DaemonContext():
    main_program()
```

위의 코드는 `main_program()`을 background process로 수행시켜준다.
그런데 이 프로그램은 중복실행이 가능하다. 즉, 여러번 수행하면 수행한 만큼 프로세스가 떠있게 된다는 얘기다.

다음 코드를 보자

```python
#!/usr/bin/env python

import daemon
from spam import main_program
from daemon.pidfile import PIDLockFile

pidLockfile = PIDLockFile('.pid')
with daemon.DaemonContext(pidfile=pidLockfile):
    main_program()
```

위 코드는 중복실행을 하더라도 첫번째 프로세스만 실행되고, 그 이후 프로세스는 lock파일 때문에
블럭되어 있게 된다. 즉, pid lockfile은 daemon context가 lock을 획득할 때까지는 블럭시키는 역할을 한다.
어쨌거나 중복실행 횟수만큼 프로세스가 떠있다.

아래처럼 하면 중복실행을 막을 수 있다.

```python
#!/usr/bin/env python

import daemon
from spam import main_program
from daemon.pidfile import PIDLockFile

pidLockfile = PIDLockFile('.pid')
if pidLockfile.is_locked():
    print "running already (pid: %d)" % pidLockfile.read_pid()
    exit(1)

with daemon.DaemonContext(pidfile=pidLockfile):
    main_program()
```

`daemon.DaemonContext`의 다른 속성들은 [PEP-3143] 문서의 내용을 참고할 것.

------
(추가)

daemonize될 때 그 전에 열려있는 파일들은 모두 닫히게 되므로,
logger를 쓸 때 파일이 닫히지 않도록 아래처럼 `DaemonContext`에
`files_preserve` 속성을 세팅해서 파일이 닫히지 않도록 주의해야 한다.
(handler 의 `steam` 속성에서 `fileno()`을 호출하여 file descriptor를 얻을 수 있다.)

```python
import daemon
from daemon.pidfile import PIDLockFile
import logging
from logging import handlers

_logger = logging.getLogger("mylogger")
_logger.setLevel(logging.INFO)

file_handler = handlers.RotatingFileHandler(
    "log/daemon.log",
    maxBytes= (1024 * 1024 * 512), # 512GB
    backupCount=3
)
_logger.addHandler(file_handler)

# ... 중략 ..

context = daemonDaemonContext(pidfile=pidLockfile)
# 아래처럼 file descriptor을 얻어서 files_preserve를 설정한다.
logfile_fileno = file_handler.stream.fileno()
context.files_preserve = [logfile_fileno]
with context:
   # files_preserve로 지정되지 않은 파일은 daemonize되면서 모두 닫힌다.
   main_program()
```

위와 같이 처리하지 않으면 daemonize된 후에 log가 파일에 씌여지지 않는다.

[PEP-3143]:http://www.python.org/dev/peps/pep-3143/
[python-daemon]:https://pypi.python.org/pypi/python-daemon/
