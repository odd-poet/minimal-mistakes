---
layout: single
title: "SPDY 테스팅 서버 만들기"
date: 2012-05-22 17:22
comments: true
tags: [SPDY, node.js]

---
<!-- TODO : redirect to http://oddpoet.net/archives/425 -->

현재 SPDY 프로토콜 구현체는 [http://dev.chromium.org/spdy](http://dev.chromium.org/spdy)에 언급된 대로 아파치 모듈이나 주요 언어들의 웹서버 구현 등이 있습니다. 그러나 SPDY가 SSL위에서 작동하는데다가, NPN(Next Protocol Neotiation) 확장을 필요로 하기 때문에 간단하게 SPDY의 작동을 확인할 목적으로 테스팅 환경을 만들 때 부담스러운 면이 많습니다.

<!-- more -->

실제 도입할 목적이 아니라 Lab Testing 목적이라면 node.js의 express를 사용하는 express-spdy를 사용하는게 가장 손쉽게 SPDY환경을 구축하는 방법일 것 같습니다. 이 글에서는 [The SPDY Book][1] 에서 실습환경 부분 참고하여 linux와 mac에서의 SPDY 서버를 만들어 보겠습니다. 클라이언트는 Google Chrome을 사용하면 세션확인까지 가능합니다.

## Edge openssl 설치

SPDY가 SSL 위에서 정상작동하기 위해서는 [NPN (Next Protocol Neogtiation)][2]확장이 필요합니다. NPN은 SSL의 다음 계층에서 HTTP를 쓸지, SPDY를 쓸지를 서버와 클라이언트가 협상할 수 있도록 해주는 것이라고 보시면 됩니다. NPN은 현재 edge openssl에서만 지원하고 있으므로 직접 openssl ftp server에서 소스를 받아서 컴파일 해야 합니다.

우선 [openssl ftp server][3] 에서 **openssl-SNAP-*** 으로 시작하는 파일 중 가장 최신파일의 경로를 확인합니다.

    mkdir -p $HOME/src
    cd $HOME/src
    wget ftp://ftp.openssl.org/snapshot/openssl-SNAP-.tar.gz
    tar xvzf openssl-SNAP-.tar.gz
    cd openssl-SNAP-

이제 아래와 같이 설정을 합니다. (맥OS의 경우 *linux-elf* 대신에 *darwin64-x86_64-cc*를 씁니다. 64비트 linux라면 *linux-x86_64*를 쓰세요.)

    ./Configure shared \
      --prefix=$HOME/local \
      no-idea no-mdc2 no-rc5 zlib \
      enable-tlsext \
      linux-elf

이제 컴파일하고 설치합니다.

    make depend
    make
    make install

## Node.js 설치

기존에 이미 apt-get(ubuntu)이나 brew(osx)로 node.js를 설치했더라도 위에서 컴파일한 edge openssl을 사용하도록 재컴파일해서 별도 설치합니다.

    mkdir -p $HOME/src
    cd $HOME/src/
    wget http://nodejs.org/dist/v0.6.18/node-v0.6.18.tar.gz
    tar xvzf node-v0.6.18.tar.gz
    cd node

앞서 컴파일 설치한 openssl을 사용하도록 아래와 같이 설정하고 컴파일해서 설치합니다. 설치경로는 마찬가지로 `$HOME/local`입니다.

    ./configure \
      --openssl-includes=$HOME/local/include \
      --openssl-libpath=$HOME/local/lib \
      --prefix=$HOME/local/node-v0.6.18
    make
    make install

## Shell 설정하기

$HOME/.bashrc 에 아래 내용을 추가합니다.

    # For locally installed binaries
    export LD_LIBRARY_PATH=$HOME/local/lib
    PATH=$HOME/local/bin:$PATH
    PKG_CONFIG_PATH=$HOME/local/lib/pkgconfig
    CPATH=$HOME/local/include
    export MANPATH=$HOME/local/share/man:/usr/share/man
    # For node.js work. For more info, see:
    # http://blog.nodejs.org/2011/04/04/development-environment/
    for i in $HOME/local/*; do
      [ -d $i/bin ] &amp;&amp; PATH="${i}/bin:${PATH}"
      [ -d $i/sbin ] &amp;&amp; PATH="${i}/sbin:${PATH}"
      [ -d $i/include ] &amp;&amp; CPATH="${i}/include:${CPATH}"
      [ -d $i/lib ] &amp;&amp; LD_LIBRARY_PATH="${i}/lib:${LD_LIBRARY_PATH}"
      [ -d $i/lib/pkgconfig ] &amp;&amp; PKG_CONFIG_PATH="${i}/lib/pkgconfig:${PKG_CONFIG_PATH}"
      [ -d $i/share/man ] &amp;&amp; MANPATH="${i}/share/man:${MANPATH}"
    done

## NPM설치

    curl http://npmjs.org/install.sh | sh

이미 기존 node와 함께 npm이 설치되었다면 이 단계는 건너뛰어도 됩니다.

## express-spdy 설치

    npm install -g express-spdy

위와 같이 express-spdy를 설치합니다. express-spdy는 express에서 spdy를 사용할 수 있게 확장한 모듈입니다.

## 샘플 App 만들기

express-spdy를 설치하였으면 application을 생성해봅니다.

    cd
    mkdir $HOME/projects
    cd $HOME/projects
    express-spdy spdy-sample
    cd spdy-sample
    npm install

spdy-sample이라는 이름의 app이 생성되었습니다. app.js 파일을 열어보면 기본적인 app 골격이 나와있습니다. (생성된 코드를 보면 Ruby 개발자분들은 sinatra, rails의 느낌을 받으실듯)

    var app = module.exports = express.createServer({
      key: fs.readFileSync(__dirname %2B '/keys/spdy-key.pem'),
      cert: fs.readFileSync(__dirname %2B '/keys/spdy-cert.pem'),
      ca: fs.readFileSync(__dirname %2B '/keys/spdy-csr.pem'),
      NPNProtocols: ['spdy/2', 'http/1.1'],
      push: awesome_push
    });

위 코드에서 key, cert, ca 부분은 SSL과 관련된 설정입니다. SSL관련 설정까지 템플릿으로 만들어주니 간단히 만들고 테스트하기에 좋습니다.

그리고 ***NPNProtocols*** 부분이 NPN 설정에 대한 부분입니다. Chrome 처럼 SPDY를 지원하는 브라우저와는 SPDY/2로 통신하고, 그 외에는 HTTP/1.1로 통신을 하게 됩니다. 즉, 여기에서 ‘spdy/2′ 부분을 제거하면 HTTP/1.1만 사용하게 되겠죠? 이렇게 SPDY를 ON/OFF하면서 비교테스트 하시면 되겠습니다. 그리고 ***awesome_push***&nbsp;는 함수 정의부를 보시면 설명이 있을텐데 SPDY의 server push에 대한 부분입니다.

## 서버 실행하고 확인하기

쉘에서 아래와 같이 실행하면 서버가 뜹니다.

    node app.js

자~ 이제 브라우저를 띄워볼까요? [https://localhost:3000/](https://localhost:3000/)로 접근하시면 됩니다. **http가 아니라 https입니다!**

뭐 별거 없는 썰렁한 샘플 페이지 입니다만, 구글 크롬으로 [chrome://net-internals/#spdy](chrome://net-internals/#spdy) 페이지를 열어보시면 그 페이지에 대한 세션이 있음을 확인할 수 있습니다. 게다가 server push된게 1개 있다는 것도 볼 수 있습니다. (awesome_push함수에서 index 페이지에서 사용하는 css파일 하나를 push하도록 되어 있어요.)

## 그리고… 흔적제거

저만 그런지 모르겠지만 잠깐 테스팅하겠다고 구질구질하게 unstable lib들을 컴파일해서 설치하고 나면 나중에 뒷정리가 짜증이 납니다. apt-get, brew와 같은 패키지 관리툴에서 관리해주는게 아니라서 의존성이 꼬일 수도 있구요.

하지만 openssl, node를 모두 `$HOME/local` 밑에 설치했기 때문에, `$HOME/local` 디렉토리만 삭제하면 됩니다. 그리고 .bashrc에 추가했던 부분도 지우면 설치전과 동일한 상태가 됩니다.

## 참고자료

- [The SPDY Book: Making Websites Fly][1]
- [Install express-spdy][4]

[1]: http://pragprog.com/book/csspdy/the-spdy-book
[2]: https://technotes.googlecode.com/git/nextprotoneg.html
[3]: ftp://ftp.openssl.org/snapshot/
[4]: https://github.com/eee-c/express-spdy/blob/master/INSTALL.md
