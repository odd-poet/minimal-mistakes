---
layout: single
title: "docker device-mapper error"
date: 2014-11-19 13:12
comments: true
tags: ['docker']
---

지난주에 빌드 서버에서 `docker pull`하다가 서버에 hang이 걸리는 문제가 발생했다.

시스템 담당자와 로그 뒤지면서 씨름하던 끝에 devicemapper가 원인이라는 것을 알았다.
시스템 로그에 보면 아래와 같은게 있는데, `dm-1`이라는게 docker의 **devicemapper** 이다.

```
...
Nov 10 16:55:56 my-server-01 kernel: EXT4-fs (dm-1): VFS: Can't find ext4 filesystem
Nov 10 17:06:40 my-server-01 kernel: EXT4-fs (dm-1): VFS: Can't find ext4 filesystem
...
```

<!-- more -->

해결방법
--------

먼저 [thin-provisioning-tools]을 설치한다.

CentOs 계열은 아래와 같이 [thin-provisioning-tools]을 설치할 수 있다.

```
sudo yum install device-mapper-persistent-data -y
```

[thin-provisioning-tools]을 설치했으면 아래와 같이 devicemapper에 문제가 있는 확인해본다.

```
sudo thin_check /var/lib/docker/devicemapper/devicemapper/metadata
```

문제가 있다면 아래와 같은 메시지들이 출력된다.

```
...
10122: was 1, expected 2
10123: was 1, expected 2
10124: was 1, expected 2
10125: was 4, expected 5
10126: was 4, expected 5
10127: was 1, expected 2
12384: was 1, expected 2
12385: was 1, expected 2
...
```

아래와 같이 문제있는 devicemapper의 metadata를 복구할 수 있다. (`thin_dump`의 `-r`이 *repair* 옵션이다.)

```
sudo service docker stop
sudo thin_dump -f xml -r /var/lib/docker/devicemapper/devicemapper/metadata > /tmp/metadata
sudo thin_restore -i /tmp/metadata -o /var/lib/docker/devicemapper/devicemapper/metadata
sudo service docker start
sudo thin_check /var/lib/docker/devicemapper/devicemapper/metadata
```

docker가 실행 중일 때 dump & restore를 하면 `thin_check`시에 *Bad Checksum* 관련 오류 메시지가 뜨니, 반드시 docker 데몬을 중단시키고 작업하자.

이제 `thin_check`를 수행해도 아무런 메시지가 실행되지 않을 것이다.

만약 이렇게 해도 문제가 되면 아예 devicemapper 관련 파일을 몽땅 삭제하는 방법을 쓰자.

```
# 모든 image를 삭제하고 devicemapper의 metadata도 삭제한다.
docker rmi $(docker images -a | awk '{print $3}' | grep -v IMAGE)

sudo service docker stop
sudo rm -rf /var/lib/docker/devicemapper
sudo service docker start
```

다시 `docker pull`을 하면 `/var/lib/docker/devicemapper/devicemapper/metadata`은 다시 생성된다.

참고 : [StackOverflow의 스레드][device-mapper-error]

[thin-provisioning-tools]:https://github.com/jthornber/thin-provisioning-tools
[device-mapper-error]:http://stackoverflow.com/questions/24709741/cant-run-docker-container-due-device-mapper-error
