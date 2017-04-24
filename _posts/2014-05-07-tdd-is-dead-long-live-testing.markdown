---
layout: single
title: "TDD is dead. Long live testing"
date: 2014-05-07 12:52
comments: true
tags: ["TDD", "test-first", "TDD is dead", "DHH"]
---

Rails를 만든 [DHH]가 쓴 "[TDD is dead. Long live testing.][TDD-is-dead]"라는 글이 최근 TDDer들에게 논란이 되고 있다. 이 논쟁에는 엉클밥, 켄트백, 마틴파울러도 참여하면서([RIP-TDD],[unclebob-twitter], [Is TDD DEAD?][is-tdd-dead]) 분위기가 후끈 달아오른 터라 급하게 날림번역을 해본다.

<!-- more -->

대충 DHH의 논지는 TDD의 "test-first"가 단위 테스트에만 초점을 맞추다 보니 전체적인 관점에서 균형이 맞지 않을 뿐더러, 전체적인 시스템 설계에는 도움이 안되더라는 얘기인 듯 하다. 그리고 중간중간 다소 과격한 듯한 표현이 나오기는 하지만 DHH는 "TDD 너네들 틀렸어!"가 아니라 "Rails에서는 다른 대안을 찾겠다"는 의도의 글로 보인다(Rails가 프로젝트 생성시에 unit/funtional/integration test의 stub, fixture 및 환경 구성도 제공하는 framework이다보니 대부분 rails가 제공하는 방식을 따르게 된다). 어쨌든 DHH의 이 글 때문에 심기가 불편한 사람이 많았는지 [Is TDD DEAD?][is-tdd-dead] 같은 토론회(?)도 진행된다니 그 결과가 사뭇 기대된다.

번역 관련해서는 수사적 표현이 많다 보니 번역이 매끄럽지 못한 부분이 많다. 원문도 함께 병기했으니 참고바람.

----------

## TDD is dead. Long live testing.

*By [David Heinemeier Hansson][DHH] on April 23, 2014*

Test-first fundamentalism is like abstinence-only sex ed: An unrealistic, ineffective morality campaign for self-loathing and shaming.

Test-first 원칙주의는 금욕적인 성(性)교육과 유사하다. 자기비하와 자기혐오를 위한 비현실적이며 쓸모없는 도덕적 캠페인 같다.


It didn't start out like that. When I first discovered TDD, it was like a courteous invitation to a better world of writing software. A mind hack to get you going with the practice of testing where no testing had happened before. It opened my eyes to the tranquility of a well-tested code base, and the bliss of confidence it grants those making changes to software.

"test-first"는 그렇게 시작되지 않았다. 내가 처음 TDD에 대해 알게되었을 때, TDD는 더 나은 소프트웨어 개발의 세계를 열어줄 친절한 초대장 같았다. 전에는 테스트가 없던 세계였다면, 테스트를 함께 할 수 있는 마음가짐을 가지게 해주었다. TDD는 잘 테스트된 코드 기반이 주는 평온함에 눈 뜨게 해주었고, 소프트웨어를 변경해야 할 때 (코드에 대한) 신뢰의 기쁨을 누리게 해주었다.

The test-first part was a wonderful set of training wheels that taught me how to think about testing at a deeper level, but also some I quickly left behind.

TDD의 "test-first" 부분은 깊은 수준에서의 테스트를 어떻게 고민해야 할지 가르쳐주고, 뒤로 미뤄야 할 부분을 빠르게 결정할 수 있게 해주는 놀라운 보조바퀴(역주:아동용 자전거에 연습용으로 붙이는 바퀴) 같았다.

Over the years, the test-first rhetoric got louder and angrier, though. More mean-spirited. And at times I got sucked into that fundamentalist vortex, feeling bad about not following the true gospel. Then I'd try test-first for a few weeks, only to drop it again when it started hurting my designs.

몇 년이 지나고, "test-first"에 대한 미사여구는 시끄러워지고 거칠어졌다. 그리고 더욱 천박해졌다. 나는 TDD 원칙주의자들의 소용돌이 속으로 빠져들게 되면서 진짜 복음서를 따르지 않는 것 같다는 나쁜 느낌을 받게 되었다. 그래서 나는 test-first로 몇 주 동안 시도해보고 그것이 내 설계를 망치기 시작할 때 그만 두곤 했다.

It was yoyo cycle of pride, when I was able to adhere to the literal letter of the teachings, and a crash of despair, when I wasn't. It felt like falling off the wagon. Something to keep quiet about. Certainly not something to admit in public. In public, I at best just alluded to not doing test-first all the time, and at worst continued to support the practice as "the right way". I regret that now.

내가 가르침을 문자 그대로 따를 수 있을 때도 있었지만, 그렇지 못할 때는 절망했다. 이것은 자부심의 요요 현상이 있었다. 마치 금주를 그만 두고 다시 술을 마시는 것 같은 느낌이었다. 뭔가 조용한 듯했지만, 확실히 공식적으로 인정하지 않는 뭔가가 있었다. 공식적으로 나는 기껏해야 항상 "test-first"로 개발하지는 않음을 암시적으로 얘기하는 것이었고, 최악은 "올바른 방식"으로써의 TDD 실천법을 계속 지원하는 것이었다. 지금은 내가 그랬던 것을 후회한다.

Maybe it was necessary to use test-first as the counterintuitive ram for breaking down the industry's sorry lack of automated, regression testing. Maybe it was a parable that just wasn't intended to be a literal description of the day-to-day workings of software writing. But whatever it started out as, it was soon since corrupted. Used as a hammer to beat down the nonbelievers, declare them unprofessional and unfit for writing software. A litmus test.

자동화된 회귀 테스트에 대한 산업계의 인식의 부족을 깨기 위해 반직관적 램(역주:기계에서 무엇을 세게 치거나 상하, 수평 이동을 하는 데 쓰이는 부분; 확실치 않음. ram이 뭐야!)으로써 "test-first"를 활용하는 것은 필요한 일일지도 모른다. 또한 소프트웨어 개발의 일상업무를 문자 그대로 표현하려는 의도는 아니었던 단순한 비유였을 수도 있다. 그러나 "test-first"가 어떻게 시작되었던지 간에, "test-first"는 훼손되었다. "test-first"를 비신자(TDD를 따르지 않는 사람들)를 깨부실 햄머로 사용하고 있으며, 비신자들을 소프트웨어 개발에 적합치 않은 비전문가로 규정하고 있다. 리트머스 시험처럼...

Enough. No more. My name is David, and I do not write software test-first. I refuse to apologize for that any more, much less hide it. I'm grateful for what TDD did to open my eyes to automated regression testing, but I've long since moved on from the design dogma.

그걸로 됐다. 내 이름은 David이고, 나는 "test-first"로 소프트웨어를 개발하지 않는다. 나는 더이상 그 사실에 대해서 사과하지 않을 것이며 그 사실을 감추지 않겠다. TDD가 자동화된 회귀 테스트에 눈을 뜨게 해준 것에 감사하지만, 나는 그 설계 신조(dogma)에서는 오래전에 벗어났다.

I suggest you take a hard look at what that approach is doing to the integrity of your system design as well. If you're willing to honestly consider the possibility that it's not an unqualified good, it'll be like taking the red pill. You may not like what you see after that.

"test-first" 접근방법이 당신의 시스템 설계의 무결성에 대해 해줄 수 있는 것이 있는지 다시 한번 면밀히 살펴보길 바란다. 당신이 "test-first"가 완전한 선이 아닐 가능성에 대해서도 솔직하게 고민할 의향이 있다면, 그건 빨간약을 선택하는 것이다(역주:영화 매트릭스에서 네오가 빨간약을 선택한 것에 대한 비유). 당신은 그 선택 후에 보게 될 것을 좋아하지 않을 수도 있다.

### So where do we go from here?

### 우리는 여기에서 어디로 가는가?

Step one is admitting there's a problem. I think we've taken that now. Step two is to rebalance the testing spectrum from unit to system. The current fanatical TDD experience leads to a primary focus on the unit tests, because those are the tests capable of driving the code design (the original justification for test-first).

첫 번째 단계는 문제가 있다는 것을 인정하는 것이다. 나는 이제 우리가 그 단계에 있다고 생각한다. 두 번째 단계는 단위 테스트에서 시스템 테스트까지의 테스트 스펙트럼에 균형을 찾는 것이다. 현재의 광신적인 TDD 경험은 단위 테스트에 주로 초점을 맞추도록 하는데, 단위 테스트가  코드 설계를 유도해내는데(이것이 "test-first"의 정당성에 대한 근거이다) 적합하기 때문이다.

I don't think that's healthy. Test-first units leads to an overly complex web of intermediary objects and indirection in order to avoid doing anything that's "slow". Like hitting the database. Or file IO. Or going through the browser to test the whole system. It's given birth to some truly horrendous monstrosities of architecture. A dense jungle of service objects, command patterns, and worse.

나는 단위 테스트에 초점을 맞추는 것이 건전하다고 생각하지 않는다. "test-first"의 단위 테스트들은 매개 객체들과 "느림"을 피하기 위한 간접적인 행동들로 가득찬 완전히 복잡한 웹을 초래한다. 데이터베이스 접근을 테스트하거나, 파일 IO에 대한 테스트 혹은 전체 시스템 테스트를 위해 브라우저를 사용하는 일을 피하려고 한다. 이것은 정말 끔직한 괴물같은 아키텍쳐를 낳았다. 서비스 객체들, 커맨드 패턴들로 가득찬 복잡한 정글이다.

I rarely unit test in the traditional sense of the word, where all dependencies are mocked out, and thousands of tests can close in seconds. It just hasn't been a useful way of dealing with the testing of Rails applications. I test active record models directly, letting them hit the database, and through the use of fixtures. Then layered on top is currently a set of controller tests, but I'd much rather replace those with even higher level system tests through Capybara or similar.

모든 종속성은 mock으로 처리하고 수 천개의 테스트가 수 초만에 수행되는 전통적인 의미에서의 단위 테스트를 나는 거의 사용하지 않는다. 그것은 Rails 애플리케이션의 테스트를 다루는 유용한 방법이 아니었다. 나는 Active Record(rails의 ORM) 모델을 직접 테스트하고, 그것이 픽스쳐 사용을 통해 직접 데이터베이스를 접근하게 한다. 그 다음의 상위 계층은 controller 테스트 집합인데, 나는 Capybara 같은 것을 사용해서 상위 수준의 시스템 테스트로 그것을 대체하려 한다.

I think that's the direction we're heading. Less emphasis on unit tests, because we're no longer doing test-first as a design practice, and more emphasis on, yes, slow, system tests. (Which btw do not need to be so slow any more, thanks to advances in parallelization and cloud runner infrastructure).

나는 우리가 가고 있는 방향이 그것이라고 생각한다. 설계 실천법으로써의 "test-first"를 우리가 더이상 사용하지 않기 때문에 단위 테스트를 덜 강조할 것이고, 느린 시스템 테스트를 보다 더 강조할 것이다. (병렬수행과 클라우드 실행을 위한 인프라스트럭처의 발전 덕에 더이상 느리게 수행될 필요는 없어지고 있다)

Rails can help with this transition. Today we do nothing to encourage full system tests. There's no default answer in the stack. That's a mistake we're going to fix. But you don't have to wait until that's happening. Give Capybara a spin today, and you'll have a good idea of where we're heading tomorrow.

Rails는 이러한 변화를 도울 수 있다. 요즘 우리는 전체 시스템 테스트를 권장하는 어떤 일도 하지 않는다. 그 스택(시스템 테스트)에서 기본 설정은 없다. 그것은 우리가 고쳐가야할 우리의 실수다. 그러나 그것이 수정되길 기다릴 필요는 없다. 당장 [Capybara]를 돌려보면 우리가 향하고 있는 곳에 대한 좋은 아이디어를 얻을 수 있을 것이다.

But first of all take a deep breath. We're herding some sacred cows to the slaughter right now. That's painful and bloody. TDD has been so successful that it's interwoven in a lot of programmer identities. TDD is not just what they do, it's who they are. We have some serious deprogramming ahead of us as a community to get out from under that, and it's going to take some time.

그러나 먼저 심호흡을 해보자. 우리는 신성한 소(sacred cows; 역주: 지나치게 신성시되어 비판・의심이 허용되지 않는 관습・제도 등)를 지금 도축장으로 데려가고 있다. 이건 고통스러운 일이다. TDD는 성공적이었고 많은 프로그래머의 정체성과 섞여 있다. 그들이 하는 것이 TDD가 아니고, 바로 그들이 TDD가 되었다. 우리는 그곳에서 빠져나와 우리보다 앞서 커뮤니티를 심각하게 재교육을 해야 하며, 이에 시간이 걸릴 것이다.

The worst thing we can do is just rush into another testing religion. I can just imagine the golden calf of "system tests only!" right now. Please don't go there.

우리가 할 수 있는 가장 좋지 않은 선택은 다른 테스트 분파로 들어가는 것이다. "system tests only"의 금송아지(역주: 유대인이 숭배했던 우상) 같은 것이 그런 것이다. 제발 그쪽으로는 가지 말길 바란다.

Yes, test-first is dead to me. But rather than dance on its grave, I'd rather honor its contributions than linger on the travesties. It marked an important phase in our history, yet it's time to move on.

그렇다. 나에게 있어서 "test-first"는 죽었다. 그러나 그 무덤 위에서 춤추기보다는, 그것이 기여했던 바를 존중하여 모사꾼으로 남지는 않겠다. 이것은 우리의 역사에서 중요한 단계이며, 이제 옮겨가야할 시간이다.

Long live testing.

테스트여 영원하라.

[TDD-is-dead]:http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html
[DHH]:http://david.heinemeierhansson.com/
[is-tdd-dead]:https://plus.google.com/app/basic/events/ci2g23mk0lh9too9bgbp3rbut0k
[RIP-TDD]:https://www.facebook.com/notes/kent-beck/rip-tdd/750840194948847
[unclebob-twitter]:https://twitter.com/martinfowler/status/460182896817733632
[Capybara]:https://github.com/jnicklas/capybara
