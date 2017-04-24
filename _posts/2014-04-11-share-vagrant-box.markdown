---
layout: single
title: "vagrant box 패키징과 공유"
date: 2014-04-11 21:14
comments: true
tags: [vagrant]
---

개발환경에서 필요한 인프라적인 요소들(db, hbase 등)을 쉽게 구축하고 공유하기 위해서
[vagrant]를 종종 사용하게 되는데, 자기가 만든 **vagrant box**를 패키징하고
다른 사람들에게 쉽게 공유하는 방법을 정리한다.

<!-- more -->

Packaging
------

현재 자신의 PC에서 세팅을 끝낸 vagrant box를 공유하기 위해서는 vagrant box 파일로
패키징해야 한다. 일반적으로 패키징할 vagrant 프로젝트 디렉토리에서 `vagrant package`만 실행하면 된다.
패키징된 vm은 `package.box`라는 파일로 생성된다.

패키징 옵션은 [Vagrant Packaging][vagrant-package] 문서를
참고한다.

패키징할 때는 다음과 같은 사항을 주의한다.

#### 가급적이면 `Vagrantfile`을 함께 패키징하지 않는다.

`Vagrantfile`을 함께 box로 패키징하면 box와 함께 설치된다.
그리고 실제 생성된 vagrant box가 부팅될때
[Vagrantfile] 문서에 나와있는 순서대로 패키징됐던 `Vagrantfile`이 먼저
로드되고, 그 다음에 `~/.vagrant.d`에 있는 `Vagrantfile`, 프로젝트의 `Vagrantfile` 순으로
로딩된다.

이때 `warning: already initialized constant VAGRANTFILE_API_VERSION` 과 같은
오류메시지를 만나게 된다. ruby에서 동일한 상수에 또 다시 값을 assign할 때 나오는
메시지인데, 여러 개의 `Vagrantfile`이 읽히면서 파일 앞부분에 있는
`VAGRANTFILE_API_VERSION = "2"` 라인이 계속 다시 evaluate 되기 때문에 발생하는 에러다.
기능에는 지장이 없지만 보기에 흐뭇하지 않다.

`Vagrantfile` 파일에는 네트웍설정 등의 vm 외부 인터페이스 관련 설정만 있으므로
굳이 공유할 vagrant box에 `Vagrantfile`을 embeded하지 않도록 하자.

#### Firewall disable

일반적으로 vagrant box 내에 특정 포트의 service를 띄워놓고 host os에서 그 서비스에
접근하게 되므로 `iptable`, `ip6table` 서비스는 disable 시키는게 좋다.

Centos의 경우 아래와 같은 명령으로 부팅시 `iptable` 서비스가 실행되는 걸 막을 수 있다.

```
$ chkconfig iptables off
```

리눅스 배포본 별 방화벽을 끄는 방법은
[HowTo Disable The Iptables Firewall in Linux][turn-off-firewall-in-linux]을 참고한다.

#### 패키징 전에 `/etc/udev/rules.d/70-persistent-net.rules`을 지운다

자기가 packaging했던 vagrant box을 설치하고 부팅해보면
아래와 같은 메시지를 만나게 되는 경우가 있다. 이 오류가 발생하면
guest os의 네트웍에 접근하지 못할 수 있다.

```
[default] Configuring and enabling network interfaces...
The following SSH command responded with a non-zero exit status.
Vagrant assumes that this means the command failed!

/sbin/ifup eth1 2> /dev/null
```
그 이유는 [udev]가 디바이스를 자동으로 감지하고
그 내용을 `/etc/udev/rules.d/70-persistent-net.rules` 파일에 쓰는데,
거기에는 `eth0`, `eth1` 과 같은 network card 디바이스가 포함된다.

그런데 vagrant가 vm을 초기화할 때 network card의 `MAC address`를 생성하기 때문에
패키징했던 vm과 그것을 기반으로 생성된 새로운 vm의 MAC address가 달라져서 발생하는 오류다.

따라서 아래와 같이 packaging 전에 이 파일을 지운다. (지워도 자동생성되는 파일이다)

```
vagrant ssh -c "sudo rm /etc/udev/rules.d/70-persistent-net.rules"
vagrant halt
vagrant package
```

Box file upload
-------

패키징된 vagrant box 파일은 사이즈가 수 백 MByte를 넘기 때문에
일반적인 파일 호스팅 서비스를 이용하기가 쉽지 않다.

아래와 같은 파일 호스팅 서비스를 고려해보자.

* Dropbox
   * 용량이 몇GB 밖에 안되서 넉넉하진 않지만 가장 손쉬운 방법이다.
* github
   * git repository에는 100MB 제한 때문에 올릴 수는 없다.
   * 하지만 github의 release 기능을 이용하면 그 보다 큰 파일을 올릴 수 있다.
   * release에 포함시킬 수 있는 용량제한이 얼마인지는 모르겠지만, 900Mb 정도 되는 파일까지는 잘 올라가더라.

Vagrant box 공유하기
--------

호스팅 서비스에 올린 box 파일의 url을 직접 사용해서 다른 사람들에게 box를 공유할 수도 있다.

이런 경우 사용자는 아래와 같이 box를 받아서 사용하게 된다.

```
vagrant init yourbox http://very.long.url/very/long/path/yourbox.box
vagrant up
```

긴 이름 때문에 공유하기도 쉽지 않을 뿐더러 URL을 직접 알려주지 않으면 힘들게 만들고
패키징한 vm box를 다른 사람들이 알 수도 없다. [vagrantbox.es]에 등록하는 방법도 있지만
더 좋은 방법이 있다. [vagrantcloud.com]에 등록하는 것이다.

#### vagrantcloud

![Vagrantcloud.com](/assets/include/2014-04/vagrantcloud.com.jpg)

[vagrantcloud.com]에 자신의 박스를 등록하면 `username/boxname` 형태로 box의 식별자가
만들어지고, [vagrantcloud.com] 내에서 **검색이 가능**하며 **version 관리**,
**다운로드 통계** 등의 기능도 제공된다.

[vagrantcloud.com]에 등록되면 다른 사람들은 [oddpoet/hbase-0.94]와 같은 box 설명 페이지를
볼 수 있고 내가 만든 box를 아래처럼 사용하게 될 것이다.

```
vagrant init hbase oddpoet/hbase-0.94
vagrant up
```

#### vagrantcloud에 box 등록하기

[vagrantcloud.com]에 계정을 만들고 **Create Box** 메뉴를 선택하면 아래와 같은 화면을
보인다.

![Create Box](/assets/include/2014-04/vagrantcloud-create-box.png)

box에 대한 이름과 설명을 등록하면 box의 식별자가 생성된다. (e.g. `oddpoet/hbase-0.94`)
그 후에는 버전을 생성한다.

![Create new version](/assets/include/2014-04/vagrantcloud-create-version.png)

버전이 생성되면 버전에 provider를 추가 할 수 있게 된다. provider는 *vitualbox*, *vmware* 등의
가상머신 종류를 의미하고, 동일한 버전의 box를 provider 종류별로 등록할 수 있다.

![Created version](/assets/include/2014-04/vagrantcloud-created-version.png)

아래는 provider를 등록하는 화면인데, 여기에 실제 box URL을 등록하면 된다.
아직 자체적인 파일 호스팅은 지원하지 않는다. 그건 베타버전인 탓도 있겠지만, 파일 호스팅을 유료로 지원할 생각인 것 같다.

![Create provider](/assets/include/2014-04/vagrantcloud-create-provider.png)

provider까지 등록하고 나면 release할 수 있게 된다.

![release box](/assets/include/2014-04/vagrantcloud-release.png)

release 후에는 다른 사람들이 [vagrantcloud.com]에서 등록된 box를 검색할 수 있고,
`vagrant box add`, `vagrant init boxname username/boxname` 형태로 등록된 박스를 사용할 수 있게 된다.

[vagrant]:http://www.vagrantup.com/
[vagrant-package]:https://docs.vagrantup.com/v2/cli/package.html
[vagrantfile]:https://docs.vagrantup.com/v2/vagrantfile/index.html
[turn-off-firewall-in-linux]:http://www.cyberciti.biz/faq/turn-on-turn-off-firewall-in-linux/
[udev]:https://wiki.debian.org/udev
[vagrantbox.es]:http://www.vagrantbox.es/
[vagrantcloud.com]:https://vagrantcloud.com/
[oddpoet/hbase-0.94]:https://vagrantcloud.com/oddpoet/hbase-0.94
