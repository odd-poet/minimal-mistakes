

## upgrade

현재 `gh-pages` branch가 메인. `master`는 upstream용 브랜치.

대략 아래처럼 `minimal-mistakes` 테마를 업그레이드 하자.

1. add remote
  - 이건 한번만 실행하면 될듯
  - `$ git remote add upstream https://github.com/mmistakes/minimal-mistakes.git`
2. pull down updates
  - `master` 브랜치를 update
  - ` git pull upstream master`
3. rebase
  - `gh-pages` branch에서 `git rebase master`
