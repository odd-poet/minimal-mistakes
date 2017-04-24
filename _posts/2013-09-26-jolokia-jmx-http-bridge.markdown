---
layout: single
title: "Jolokia: JMX-HTTP bridge"
date: 2013-09-26 23:17
comments: true
tags: ['java', 'monitoring', 'jmx', 'jolokia']
---

요즘 python으로 host monitoring 시스템을 만드는 중인데,
java가 아니다보니 JVM의 JMX 연결을 할 수 없는 문제가 발생했다.
그래서 구글링 중에 [Jolokia]라는 녀석을 발견했는데, JMX-HTTP bridge를 제공해준다.

*Jolokia*는 HTTP를 통해서 MBean의 속성에 접근할 수 있게 해주며, `JSON` 형식으로 결과를 돌려준다.
따라서 *Jolokia agent*를 띄워놓으면 Java가 아닌 언어를 쓰더라도 `HTTP/JSON`으로 JVM MBean에
접근할 수 있다.

<!-- more -->

Agent Mode
------

*Jolokia*는 다양한 상황에서 agent로 작동할 수 있게 binary를 제공하는데, 제공하는 agent들은
다음과 같다.

WAR Agent
: JEE서버에 web application으로 작동하는 Agent

OSGi Agent
: OSGi 컨테이너 내에서 작동하는 Agent

JVM Agent
: Oracle/Sun 기반의 JVM(version 6 이상)에 작동하는 Agent

Mule Agent
: Mule ESB 내에서 작동하는 Agent

![Jolokia as Agent](/assets/include/2013-09/jolokia-agent-mode.png)

Proxy Mode
------

Proxy 모드는 *jolokia agent*를 target platform에 배포할 수 없는 경우를 위한 방법이다.
*jolokia*가 대상 JVM에 JMX로 연결하고 MBean에 접근하고, *Jolokia client*에 `HTTP/JSON`
인터페이스를 제공해주게 된다.

![Jolokia as Proxy](/assets/include/2013-09/jolokia-proxy-mode.png)

나의 경우 모니터링 대상 호스트의 Tomcat에 jolokia를 직접 배포할 수 없는 상황이므로,
모니터링을 위해 jolokia를 proxy 모드로 실행하는 방식을 선택했다.

### 다운로드

[Jolokia Download][Jolokia-download] 페이지에서 JVM-Agent를 다운로드한다.
파일은 `jolokia-jvm-VERSION-agent.jar` 형식의 이름이다.

### 실행방법  ###

아래처럼 아무 arguments없이 실행하면 실행중인 JVM process 목록을 pid와 함께 보여준다.

```bash
$ java -jar jolokia-jvm-1.1.3-agent.jar
79182   org.apache.catalina.startup.Bootstrap start
83727   jolokia-jvm-1.1.3-agent.jar
```

`start` command와 *pid* 혹은 이름에 대한 *정규식*을 argument로 주면
*Jolokia*는 타겟 JVM에 attatch되어 시작된다.

```bash
$ java -jar jolokia-jvm-1.1.3-agent.jar start catalina
Started Jolokia for process matching "catalina" (PID: 79182)
http://localhost:8778/jolokia/
```

세부 실행 옵션들은 `--help` 로 확인할 것

### MBean에 접근하기  ###

*Jolokia*가 실행되면 아래처럼 `/jolokia/read/MBean_OBJECT_NAME` 형식의 URL로
MBean 속성을 읽을 수 있다. 이 부분은 *Jolokia*가 Agent 모드로 구동될 때도 마찬가지이다.

```bash
$ curl http://localhost:8778/jolokia/read/java.lang:type=Memory
{"timestamp":1380207665,"status":200,"request":{"mbean":"java.lang:type=Memory","type":"read"},"value":{"Verbose":false,"ObjectPendingFinalizationCount":0,"NonHeapMemoryUsage":{"max":136314880,"committed":24313856,"init":24313856,"used":19809328},"HeapMemoryUsage":{"max":1043922944,"committed":88735744,"init":73310080,"used":33035216},"ObjectName":{"objectName":"java.lang:type=Memory"}}}
```

[Jolokia]:http://www.jolokia.org/
[Jolokia-download]:http://www.jolokia.org/download.html
