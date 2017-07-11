---
layout: single
title: "Graceful Shutdown Server in Docker"
date: 2017-07-11T16:15:22+09:00
tags: ["docker"]
---

대부분의 서버 프로그램들은 `SIGTERM`을 받으면 더이상의 연결은 받지 않지만, 현재 연결에 의한 작업은 마치고 종료된다.

그런데 docker로 이미지를 만들어 server를 구동하는 경우 `docker stop` 명령으로
server를 graceful하게 shutdown할 수 없는 경우가 있다.

<!-- more -->

이런 문제의 발생여부는 해당 docker image의 `ENTRYPOINT`(`Dockerfile`에서 정의된)가 무엇이냐 달려있다.

예를 들어 아래와 같이 `ENTRYPOINT`를 정의한 경우 `docker stop`을 실행하면 서버는 graceful shutdown된다.
```
ENTRYPOINT ["java", "-jar", "server.war"]
```

그러나 `ENTRYPOINT`가 shell script인 경우에는 `TERM` 시그널을 shell script가 받기 때문에
서버 프로세스는 signal을 받지 못하고 최종적으로 docker container는 일정 시간 후 강제종료(`KILL`)된다.

따라서 아래와 같이 `ENTRYPOINT`를 shell script로 설정했다면 추가적인 작업을 해야한다.

```
ENTRYPOINT ["start.sh"]
```

```bash
#!/usr/bin/env bash
# file: start.sh
...(전략)...

trap `exitHandler` EXIT
exitHandler() {
    echo "Server will be shutdown" 1>&2
    kill -TERM $(jps | grep war | awk '{print $1}')
}

java $JAVA_OPTS -jar server.war
```

`ENTRYPOINT` script를 위와 같이 작성하면,
1번 프로세스이자 메인 프로세스인 `start.sh`이 docker container에 대한 SIGNAL을 받고,
`trap`에 의해 `exitHandler`에서 종료 시그널에 대한 작업을 할 수 있게 된다.

따라서 최종적으로 `java` 프로세스는 정상적으로 `TERM` 시그널을 받아 gracefully 종료된다.
