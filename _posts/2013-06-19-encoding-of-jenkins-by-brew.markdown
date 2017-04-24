---
layout: single
title: "brew로 설치한 Jenkins의 인코딩"
date: 2013-06-19 15:00
comments: true
tags: [osx, jenkins, brew, encoding]
---

OSX에서 [brew]를 이용해서 jenkins를 설치하면 JVM의 기본 인코딩(`file.encoding`) 때문에 한글 처리 등에 문제가 생기는 경우가 있다.

JVM 옵션으로 `-Dfile.encoding=UTF-8`을 주면 될 것 같지만, `~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist` 에 JVM옵션을 추가해도 jenkins의 `file.encoding`은 여전히 US-ASCII 다.

<!--more -->

jenkins가 업데이트 될 때마다 고쳐하는 게 문제이지만, `~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist`파일에 아래처럼 환경변수를 추가해주면 인코딩을 UTF-8로 jenkins를 띄울 수 있다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dt>
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>homebrew.mxcl.jenkins</string>
    <key>ProgramArguments</key>
    <array>
      <string>/usr/bin/java</string>
      <string>-jar</string>
      <string>/usr/local/opt/jenkins/libexec/jenkins.war</string>
      <string>--httpListenAddress=127.0.0.1</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>EnvironmentVariables</key>
    <dict>
      <key>LANG</key>
      <string>ko_KR.UTF-8</string>
    </dict>
  </dict>
</plist>
```

위에서 `EnvironmentVariables` 이하를 추가한다. 다시 말하지만 위쪽의 `ProgramArguments` 부분에 `-Dfile.encoding=UTF-8`을 넣는 방식은 환경변수 때문에 정상적으로 작동하지 않는다.

jenkins를 아래와 같이 다시 시작한다.

```bash
$ launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist
$ launchctl load ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist
```

[brew]:http://mxcl.github.io/homebrew/
