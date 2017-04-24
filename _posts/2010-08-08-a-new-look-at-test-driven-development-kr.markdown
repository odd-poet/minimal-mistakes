---
layout: single
title: "TDD에 대한 조금 다른 생각"
date: 2010-08-02 02:20
comments: true
tags: [TDD, ATDD, BDD]
---
<!-- TODO : redirect to http://oddpoet.net/archives/242 -->
Dave Astels의 [A New Look at Test Driven Development]라는 Article을 번역한 글입니다.
BDD(Behavior Driven Development)의 시작점이라 할 만한 글이지요.
2005년도에 씌여진 아티클이지만 개발조직에서 TDD의 수행지표로 code coverage를 사용하고 있는 작금의 현실에,
TDD의 의미를 다시금 새겨보는데 도움이 될 듯 합니다.

참고로 BDD(Behaviour Driven Development)를 일반적으로 ‘행위주도개발’이라고 번역하는 듯하지만,
문맥상 Behavior를 ‘행위’로 번역할 경우 의미전달이 힘든 부분이 있어서 ‘기대행동(Behavior)’라고 번역/표기했습니다.
사실 길지 않은 글이니 원문을 읽어보시는 걸 권장합니다. (발번역이라 죄송~)

<!-- more -->
## A New Look at Test Driven Development

Test Driven Development (TDD)는 요즘 전성기를 맞고 있다.
큰 회사들은 개발자들이 TDD를 교육하는데 많은 돈을 쓰고 있으며, 컨퍼런스에도 TDD는 단골주제다.
내 책([Test-Driven Development: A Practical Guide])은 Jolt award를 수상했다.
모든게 잘 되어 가는 것 같다.
TDD를 적용한 모두가 TDD를 완전히 이해하고 있고, 그로 인한 혜택을 누리고 있다. 정말 그런가?

### 우울한 현실 (Fat Chance!)

내가 얘기해본 10% 정도의 사람들만 TDD가 무엇인지 이해하고 있다.
사실 5% 정도밖에 안될지도… 어디서 잘못된걸까?
사람들은 TDD를 **test**에 대한 것이라고 생각한다. **case**라고 생각하지 않는다는 거다.

조금 비슷하기는 하지만, 일반적으로 사람들은 TDD를 low level의 회귀테스트 정도로 본다.
그런데 그건 TDD의 부수적인 이익 정도에 불과하다.
왜 이런 일들이 비일비재하게 일어나는 걸까? 잠깐 과거로 돌아가보자.
만약 [my old Coad Letter] 의 예전 이슈들을 봤다면, 그곳에서 TDD의 배경이 되는 것들을 찾을 수 있다.

	원래 XP는 깨질 수 있는 모든 것들을 테스트한다는 원칙을 가지고 있었다.
	그리고 XP의 테스팅 관습은 Test Driven Development로 진화했다.
	XP originally had the rule to test everything that could possibly break.
	Now, however, the practice of testing in XP has evolved into Test-Driven Development.

그들이 테스트를 작성하는 것에 대해 얘기하던 그 시절을 생각해보면,
왜 TDD가 테스트 중심의 어휘들을 가지게 되었는지를 상상할 수 있다.
그리고 이게 사람들이 TDD가 테스트가 목적이라고 생각하게된 이유이다.
그들이 TestCases, TestSuites, Tests 등에 대해 얘기했고,
메소드 이름을 “test”로 시작하게 했기 때문이다.
jUnit의 경우 그렇다. nUnit은 test로 메소드 이름이 시작해야 한다는 제약은 없다.
(역주: JUnit도 annotation을 활용하게 되면서 이제 method 이름에 대한 명명제약은 없어졌지요.)

결국 TDD로 진화하면서 우리는 전혀 다른 종의 동물을 만들어 냈다.
원래 XP 관습은 깨질 수 있는 모든 것들에 대한 테스트를 작성하는 것이었다.
그래서 테스트를 먼저 작성하는 것에서 시작했고, 결국 TDD가 되었다.
그러나 TDD는 사람들이 생각하는 것처럼 최종점이 아니다.
TDD는 단지 다음 단계를 위한 발판일 뿐이다.

사실 **unit**이라는 단어가 가장 큰 문제다.
그 이유는 첫번째로 **unit**은 막연한 용어라는 점이고,
두번째로는 그 단어가 코드의 구조적 분할을 의미한다는 점이다.
그래서 사람들은 method나 class를 테스트해야 한다고 생각한다.
우리는 *unit*에 대해서 생각해야 하는게 아니고, **기대행동(behavior)**의 관점에서 생각해야 한다.


unit test에 대한 이런 생각들은 우리들이 코드의 구조에 따라 테스트를 나누도록 한다.
예를 들면, product class들과 test class들이 1:1 관계를 만드는 것이다.

그건 우리가 원하는 게 아니다. 실제로는 기대행동(behaviour) 단위로 나누는 것이 필요하다.
또한 일반적인 unit test보다 더 작은 단위의 레벨에서 작업하는 것이 필요하다.
우리는 작은 단위의 기대행동들에 초점을 맞추어, 매우 작은 단위로 작업해야 한다.
예를 들면,
“리스트가 비어 있을 때**[Given]**,
객체에 대해 add() 메소드가 호출되면**[When]**,
리스트에는 하나만 들어있어야 한다**[Then]**.”에서
메소드는 매우 특정 상황(context)에서 특정 매개변수와 함께 호출되고,
매우 특정한 결과가 만들어지는… 이렇게 하나의 메소드에 대해 더 작은 단위로 작업해야 한다.

이것이 바로 “테스트”라는 관점에 의해서 가려져버린 고대의 아이디어이다.
왜 이렇게 되었을까? 사람들이 “Test”에 대해 어떻게 생각하는지 보자.

개발자들은 보통 이렇게 생각한다:
“난 모든 테스트를 작성할 생각은 없어”,
“이건 매우 단순한 코드라서 테스트할 필요가 없거든”,
“테스트는 시간 낭비야”,
“난 이미 많이 해봤어”.

프로젝트 매니져들은 이렇게 생각한다:
“테스트는 코드가 다 만들어 진 후 하는거야”,
“우리는 테스팅할 사람들이 따로 있어”,
“당장 거기에 쏟을 시간은 없어”….

사람들은 테스팅에 대한 생각은 이렇게도 부정적이고, 하지 않을 이유를 쉽게도 찾아낸다.
특히나 시간이 없고, 압박이 심한 경우는 더욱….

### TDD가 테스트가 아니라면, 뭔데? (So if it’s not about testing, what’s it about?)

TDD는 실제 코딩을 하기 전에 무엇을 만들 것인지 생각해보는 작업이다.
즉, 간략하고, 모호하지 않으며, 실행가능한 형태으로 기대행동(behavior)들을
보다 작은 단위로 명세화(specification)하는 과정이다.
이 과정은 테스트를 작성하는 것일까? 아니다.
이것은 코드가 무엇을 해야하는지에 대한, 명세(specification)을 작성하는 것이다.
그래서 이것은 코드를 작성하기 전에 작성하는 것이 좋다.
더 많은 명세들이 해야할 일을 더 명확히 해주기 때문이다.
TDD의 기본 방식대로 조금씩 점진적으로 작업한다.
기대행동(behavior)들을 작은 단위로 명세화하고, 그리고 그것을 구현한다.

이제 TDD가 테스트를 작성하는 것이 아니라,
기대행동(behavior)에 대한 명세에 대한 것이라는 걸 알았다면 관점이 바뀔 것이다.
각 product class마다 test class를 만든다거나,
어떤 method에 대한 test method를 만들어서 테스트한다던가 하는 일들이
얼마나 어이없는 일인지 느껴질 것이다.

### 그럼 무엇을 할까?(So What to do?)
test라는 관점에서 생각하지 않으려고 해도 JUnit은 그걸 어렵게 만든다.
그럼 이런 일에 걸맞는 새로운 프레임웍이 있을까?
ThoughtWorks의 Dan North는 jBehave 라는 프로젝트를 시작했다.

기대행동(behavior) 중심의 어휘와 개념들을 사용하는 행동 중심의 프레임웍을 사용하면, 기대행동(behavior)에 대한 명세로써 생각하는데 도움이 된다.

### 기대행동 명세 프레임웍(A Behaviour Specification Framework)
기대행동(behavior)에 대한 명세는 어떤 모습이어야 할까?
다음과 같은 부분에서는 jUnit과 같은 것들과 비슷해야 할 것이다.

- 충분히 잘 동작한다.
- 모든 사람들에게 익숙한 형태이다.

가장 큰 차이는 어휘(vocabulary)이다.
*TestCase*를 상속받는 것이 아니라, *Context*를 상속받는다.
그리고 method를 쓸때 *test*가 아니라, *should*로 시작하는 이름을 사용한다.
검증에는 `assertions`(e.g. assertEquals(exp, act)) 대신,
`shouldBeEqual`(act, exp) 같은 어휘들을 사용한다.

Smalltalk와 Ruby의 경우, class library 안으로 이런 framework을 내장시키는데 좀더 용이하다.
예를 들면, `actual shouldEqual: expected` 나, `result shouldBeNull`, `[2/0] shouldThrow: DivideByZeroException` 과 같은 형태로 쓰는 것이 가능하다.

### 현재 진행 상황은…(What Now?)
앞에서 얘기한 대로 Dan North는 기대행동(behavior) 명세를 위한 junit 대체물로
jBehave 프로젝트를 시작했다.
나도 조금 다른 방법으로 그 프로젝트에 참여할 것이다.
나는 Smalltalk와 Ruby에서 이러한 아이디어들을 적용해볼 계획을 가지고 있고,
행위기반의 jUnit 버전을 위한 작업을 진행중이다.

### 요약
TDD에 대해 알고 있는 모든 것이 무용지물이 된다는 얘기인가? 아니다.
TDD의 모든 기술을은 BDD에도 적용가능하다.
행위를 기술한다는 접근에서 이것은 TDD와 같다.
여전히 mock을 사용할 수도 있다.
바뀐 것은 어휘와 관점이다.

관점은 매우 중요하다.
프로세스의 가치가 어떻게 바뀌었는가?
다수의 **should**를 사용하며, should method의 이름들은 보다 명세를 명확하게 해줄 것이다.
예를 들면…

* `shouldAllowValidUser`
* `shouldCheckIsValidBeforeUpdate`
* `shouldUpdateRecordSet`
* `shouldWriteToUpdateLog`

요약하면…

1. TDD에 대한 문제는 우리들이 서로 다른 방향을 바라보고 있었다는 것이다. 틀린 방향을…
2. 우리는 검증 테스트(verification test)가 아니라, 기대행동 명세(behavior specifications)를
	생각하는 것에서 시작할 필요가 있다.
3. 클래스나 메소드에 대한 테스트라는 관점보다, 각각의 기대행동(behavior)의 관점이 더 명확해지므로써,
	실행가능한 문서를 가질 수 있게 된다는 잇점이 있다.
4. TDD가 생겨난 이후 어느 누구도 그 이름의 의미를 바꾸거나, 우리가 기대하는 바를 바꾸지 않았다.
	그래서 우리는 이 새로운 작업 방식에 **BDD:"Behaviour Driven Development"** 라는
	새로운 이름을 사용하려고 한다.

[A New Look at Test Driven Development]: http://techblog.daveastels.com/2005/07/05/a-new-look-at-test-driven-development/
[Test-Driven Development: A Practical Guide]: http://www.amazon.com/Test-Driven-Development-Practical-David-Astels/dp/0131016490
[my old Coad Letter]: http://bdn.borland.com/article/0,1410,29689,00.html
