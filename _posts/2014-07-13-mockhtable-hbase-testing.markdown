---
layout: single
title: "MockHTable : HBase testing"
date: 2014-07-13 15:56
comments: true
tags: ['HBase', 'testing']
---

HBase 기반 시스템을 개발하면서 제일 아쉬운게 HSQL과 같이 테스트 시에 사용할 수 있는 embeded DB였다. HSQL은 실제 DB와 거의 동일한 기능을 제공하면서 로컬 머신에서 빠르게 동작하기 때문에 Unit Test 작성하고 확인할 때 빠르게 피드백을 받을 수 있게 도와주는 좋은 툴이었다.

하지만 HBase는 unit test 수행후 rollback 한다던가 하는 전략이 불가능하기 때문에 매 테스트마다 테이블을 생성하고 삭제해야 했다. 시간이 지날수록, 로직에 대한 시나리오 테스트가 늘어날 수록 테스트 수행시간은 늘어날 수 밖에 없었다.

그러다가 이 문제의 해결책으로 찾은 것이 바로 [MockHTable]이다.

<!-- more -->

[MockHTable]은 in-memory로 작동하는 HTableInterface 구현이다. HBase 스펙에 맞게 로직을 구현했기 때문에 Get/Put/Delete/Increment 등의 대부분의 operation이 동일하게 작동한다. 또한 서버 구동이나 네트웍 접근을 하지 않기 때문에 간편하게 HBase 연동 테스트를 작성하고 결과를 확인할 수 있다.

하지만 MockHTable가 완벽하게 실제 HBase와 동일하게 작동하는가의 문제가 있다. MockHTable에 버그가 있을 수도 있고, HBase 버전에 따라 세부적인 작동이 달라질 수도 있다. 따라서 Unit Test 수준에서는 MockHTable로 빠르게 검증하고, Integration Test에서는 실제 HBase에 연동하여 테스트하는 것이 좋겠다. (실제로도 agaoglue 의 구현에는 실제 HBase와 다르게 작동하는 부분이 있다. )

하나의 클래스라서 코드가 길어 보이지만, 단순하게 생각하면 HBase는 기본 데이터 모델인 KevValue을 분산저장하는 시스템에 불과하다. MockHTable의 소스를 보면 HBase가 KeyValue 객체를 어떻게 다루는지 대략적으로 파악을 할 수 있으므로 HBase를 이해하는데 도움이 될 수 있다.

아래는 내가 원래 버전인 agaoglu의 MockHTable를 fork해서 몇 가지 문제를 수정한 버전이다.

{% gist odd-poet/bbc2aa93e41de6dd5475 MockHTable.java %}

* filter 로직 수정(by k-mack): https://gist.github.com/k-mack/4600133
	* agaoglue 버전은 filter 구현에 버그가 있다.
* batch 수정
	* result array의 갯수와 actions의 갯수가 일치하지 않는 문제.
	* 누락된 Increment / Append 등의 operation 처리 추가
* exists 메소스 수정
	* 버전에 따른 차이인지 모르겠지만 HBase 0.94 기준으로 exists는 단순히 get 수행후 result.isEmpty()를 체크하는 방식으로 수행된다. [MockHTable]의 구현과 달라서 내가 사용 중인 0.94 버전 기준으로 로직을 변경함.


아래는 실제 Hbase와 동일하게 작동하는지 확인하기 위해 만든 테스트다. MockHTable 사용 중에 다른 점이 나타날 경우 여기에 테스트를 추가하면서 보완할 생각.

{% gist odd-poet/bbc2aa93e41de6dd5475 MockHTableTest.java %}

[MockHTable]:https://gist.github.com/agaoglu/613217
