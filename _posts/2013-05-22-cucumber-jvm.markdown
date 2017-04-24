---
layout: single
title: "Cucumber-JVM"
date: 2013-05-22 20:27
comments: true
tags: [ATDD,BDD,cucumber,cucumber-jvm]
---

ruby에서 `rspec`과 함께 대표적인 **BDD(Behavior Driven Development)** 툴로 사용되던,**[cucumber]**가 jvm으로 포팅되어 **[cucumber-jvm]**으로 돌아왔다. **Cucumber**를 만든 Aslak Hellesøy가 예전에 jruby로 java와 연동해서 cucumber를 사용할 수 있게 해주던 [cuke4duke]라는 걸 만들었는데, 실사용에 문제가 많더니만 결국 java로 cucumber를 재개발했나보다. (`cuke4duke`는 현재 더이상 개발되지 않음)

<!-- more -->

`cucumber-jvm`은 `java`외에도 `groovy`, `scala`, `jython` 등 jvm 기반의 언어 다수를 지원한다. 그래서 `cucumber-java`가 아니라 `cucumber-jvm`이라 이름 붙인듯 하다. 이 글에서 `maven`, `junit`, `spring framework`과 같은 대표적인 java 개발환경에서 `cucumber`를 사용하기 위한 방법을 소개한다.

## maven dependency ##

`pom.xml` 파일에 아래와 같이 dependency를 추가한다.

```xml
...
<properties>
  <cucumber.version>1.1.2</cucumber.version>
  <junit.version>4.11</junit.version>
</properties>
<dependencies>
  ...
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>${junit.version}</version>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>info.cukes</groupId>
    <artifactId>cucumber-junit</artifactId>
    <version>${cucumber.version}</version>
  </dependency>
  <dependency>
    <groupId>info.cukes</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>${cucumber.version}</version>
  </dependency>
  <dependency>
    <groupId>info.cukes</groupId>
    <artifactId>cucumber-spring</artifactId>
    <version>${cucumber.version}</version>
  </dependency>
<dependencies>
...
```

주의할 점은 `junit`은 최신버전(현재 4.11)을 사용해야 한다. `4.8`버전에서 문제가 있다. 또, 현재 최신인 `1.1.3`버전의 `cucumber`는 *pretty formatter*에 오류가 있으니, `1.1.2`버전을 사용한다.

> (추가: 2014.04)
> 현재 `1.1.6`버전의 경우 formatter 문제는 없다.

- `cucumber-junit`은 cucumber를 JUnitRunner을 이용하여 실행하기 위해 필요하다. 대부분의 IDE가 junit을 지원하므로 `cucumber-junit`을 사용하는 것이 편하다.
- `cucumber-java`는 java를 이용하여 *step definition*을 작성하기 위해 필요하다.
- `cucumber-spring`은 *Spring Framework*과 연동하여 사용할 때 테스트를 위한 `Configuration Context`를 로딩하기 위해서 필요하다. (Spring을 사용하지 않는다면 뺀다.)

## JUnit 연동 ##

JUnit을 통해서 Cucumber 테스트를 실행하기 위해서 아래와 같은 파일을 `src/test/java/features` 디렉토리 밑에 추가한다.

```java
package features;

import cucumber.api.junit.Cucumber;
import org.junit.runner.RunWith;

@RunWith(Cucumber.class)
@Cucumber.Options(
        features = "classpath:features",
        format = {"pretty", "json:target/cucumber.json", "html:target/cucumber"}
)
public class CucumberTests {
}
```

일반적으로 cucumber 관련 파일들은 `features` 패키지 밑에 둔다.

- **`@RunWith`** : JUnitRunner를 통해서 Cucumber가 실행되도록 한다. `Cucumber.class`가
      실질적으로 feature에 대한 테스트를 수행하게 된다.
- **`@Cucumber.Options`** : Cucumber실행에 대한 옵션.
  - `features` : `gherkin` DSL로 작성된 feature 파일들의 경로를 지정한다.
      보통 `src/test/resources/features/` 밑에 위치하므로 `classpath:features`라고 지정한다.
  - `format` : 출력 포맷을 지정한다. `pretty`는 텍스트 형태의 console 출력이고, json/html 등은 파일명을 지정하게 된다.

- `CucumberTests` 클래스는 JUnit으로 테스트를 실행하기 위한 용도로만 사용하므로 클래스 내부는 비워둔다.

## Spring 연동 ##

Spring Framework 환경에서 JUnit test를 쓸 때 `@org.springframework.test.context.ContextConfiguration` 어노테이션을 사용하여 로딩할 context configuration을 지정하게 된다. Cucumber에서는 기본적으로 `classpath:cucumber.xml` 파일을 Spring context로 로딩한다. 따라서 `cucumber.xml`파일을 `src/test/resources/`에 만든다.

> (추가: 2014.04)
> 버전 `1.1.6`을 기준으로 이제 Cucumber는 `cucumber.xml`을 찾지 않는다.
> 대신 StepDefs 클래스에 `@ContextConfiguration("classpath:some-context.xml")` 어노테이션으로 SpringContext를 로딩한다.


아래는 예시일 뿐이고, 테스트에 필요한 Spring 설정을 추가하거나, 별도로 테스팅용 context configuration 파일이 있다면 import하도록 한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:annotation-config/>
    <context:component-scan base-package="net.oddpoet.cucumber.sample"/>
</beans>
```

## feature 작성  ##

feature 파일은 위의 `CucumberTests.java` 에서 `features` 옵션으로 설정한대로
`src/test/resources/features/` 밑에 만든다. feature가 많아지면 sub-directory를 만들어 구성할 수도 있다.
아래는 feature 파일의 예이다.

``` gherkin main.feature
@interface
Feature: 대화형 팀매칭 시스템

  시스템과 대화형으로 작업하는 사용자는
  입력값을 입력하면 그에 대한 즉각적인 피드백을 받는다.
  모든 입력이 완료되면 시스템은 입력된 선수 정보를 기반으로 2팀을 구성하여 출력한다.

  - 각 선수 정보 포맷 : <선수번호> <싫어하는 선수 수> <싫어하는 선수번호> <싫어하는 선수번호> ...

  Scenario: 기본 예제
    Given 시스템 출력: '총 선수 수를 입력하시오'
    When 사용자 입력: '5'
    Then 시스템 출력: '1번선수 정보를 입력하시오'
    When 사용자 입력: '1 1 2'
  # 1번 선수는 1명을 싫어하고 싫어하는 선수는 2번 선수이다
    Then 시스템 출력: '2번선수 정보를 입력하시오'
    When 사용자 입력: '2 1 3'
  # 2번 선수는 1명을 싫어하고 싫어하는 선수는 3번 선수이다
    Then 시스템 출력: '3번선수 정보를 입력하시오'
    When 사용자 입력: '3 0'
  # 3번 선수는 싫어하는 선수가 없다
    Then 시스템 출력: '4번선수 정보를 입력하시오'
    When 사용자 입력: '4 2 2 3'
  # 4번 선수는 2명을 싫어하고 싫어하는 선수는 2, 3번 선수이다
    Then 시스템 출력: '5번선수 정보를 입력하시오'
    When 사용자 입력: '5 1 4'
  # 5번 선수는 1명을 싫어하고 싫어하는 선수는 4번 선수이다
    Then 팀 구성 결과를 출력한다


  Scenario: 총 선수 수에 오류가 있을 경우
    Given 시스템 출력: '총 선수 수를 입력하시오'
    When 사용자 입력: '-10'
    Then 시스템 출력: '총 선수 수를 다시 입력하시오'
    When 사용자 입력: '3'
    Then 시스템 출력: '1번선수 정보를 입력하시오'

  Scenario: 선수정보에 오류가 있을 경우
    Given 시스템 출력: '총 선수 수를 입력하시오'
    When 사용자 입력: '5'
    Then 시스템 출력: '1번선수 정보를 입력하시오'
    When 사용자 입력: 'A 1 B'
    Then 시스템 출력: '1번선수 정보를 다시 입력하시오'
    When 사용자 입력: '1 1 2'
    Then 시스템 출력: '2번선수 정보를 입력하시오'
```

feature 파일을 만들고 JUnit으로 테스트를 수행하면, Step Definition이 없다는 메시지와 함께
step definition에 대한 code snippet을 console에 출력해준다.

```
You can implement missing steps with the snippets below:

@Given("^시스템 출력: '총 선수 수를 입력하시오'$")
public void 시스템_출력_총_선수_수를_입력하시오() throws Throwable {
    // Express the Regexp above with the code you wish you had
    throw new PendingException();
}

@When("^사용자 입력: '(\\d+)'$")
public void 사용자_입력_(int arg1) throws Throwable {
    // Express the Regexp above with the code you wish you had
    throw new PendingException();
}

@Then("^시스템 출력: '(\\d+)번선수 정보를 입력하시오'$")
public void 시스템_출력_번선수_정보를_입력하시오(int arg1) throws Throwable {
    // Express the Regexp above with the code you wish you had
    throw new PendingException();
}

@When("^사용자 입력: '(\\d+) (\\d+) (\\d+)'$")
public void 사용자_입력_(int arg1, int arg2, int arg3) throws Throwable {
    // Express the Regexp above with the code you wish you had
    throw new PendingException();
}

...(이하 생략)...
```

아직 각 `step`(`Given`, `When`, `Then`)에 대한  `step definition`이 정의되지 않아서
발생하는 경고이다. Cucumber는 이것을 테스트의 **Fail**로 처리하지 않고, **Pending** 처리한다.

## Step Definition 작성  ##

Step definition은 feature에 기술된 각 단계(step)를 어떻게 수행할 것인지를 코딩하는 하는 단계다.
보통 실제 코드가 만들어진 후 작성하여 연결하게 된다.

Step definition을 작성할 때는 cucumber 실행시 출력되는 code snippet을 복사해서 붙일 수도 있지만,
Intellij같은 IDE를 사용한다면 Step Definition을 생성해주는 IDE의 기능을 사용한다.

일반적으로 `src/test/java/features/step_definitions` 밑에 Step Definition 클래스를 정의한다.
Cucumber는 *step*과 *step definition*을 정규식 매칭으로 연결하므로 의미적으로 동일한 step들이
동일한 step definition으로 연결되도록 정규식을 수정한다.

> (추가: 2014.04)
> 버전 `1.1.6` 기준으로 StepDef 클래스에 `@ContextConfiguration` 어노테이션으로 필요한 SpringContext를 지정해야 DI가 정상적으로 이루어진다.


```java
package features.step_definitions;

import java.io.*;

import static org.hamcrest.CoreMatchers.is;
import static org.junit.Assert.*;
import org.springframework.test.context.ContextConfiguration;

@ContextConfiguration("classpath:cucumber.xml")
public class InteractiveStepDef {
    @Autowired
    TeamCreateService teamCreateService;

    @Autowired
    @Qualifier("inputWriter")
    PrintWriter inputWriter;

    @Autowired
    @Qualifier("outputWriter")
    LoggedWriter outputLog;

    @Given("^시스템 출력: '(.+)'$")
    public void 시스템_출력_(String output) throws Throwable {
        String systemOut = outputLog.getLoggedLine();
        assertThat(systemOut, is(output));
    }

    @When("^사용자 입력: '(.+)'$")
    public void 사용자_입력_(String input) throws Throwable {
        inputWriter.println(input);
    }
    .. 후략...
```

- *JUnit*과 유사하게 `@cucumber.api.java.Before`, `@cucumber.api.java.After` 어노테이션으로
    Step Definition 클래스에 대한 `setUp`, `tearDown` 코드를 쓸 수 있다.
- Step Defnition 클래스는 일반적인 Java 구현이므로, Spring DI를 사용하거나, `mockito` 역시 사용할 수 있다.


## 문서로써의 테스트 결과 ##

Cucumber의 feature는 **test automation**으로써의 기능 외에도 **문서**의 기능도 가진다.
따라서 프로젝트 관계자들이 모두 그 결과를 보고 검토할 수 있도록 CI 툴과 연계하는 것이 좋다.

Jenkins를 사용한다면 [cucumber-report] 플러그인을 사용하여 보다 fancy한 결과를 얻을 수 있다.


## 참고할만한 것들 ##
- [Step Definitions 용례][cucumber-step-definitions]
- [Cucumber 다국어 예제][cucumber-examples]
- [Top5 Cucumber Best Practices][top-5-best-practices]
- [자작 Cucumber+Spring Sample][cucumber-spring-sample]

[cucumber-spring-sample]:https://github.com/odd-poet/cucumber-sample
[top-5-best-practices]:http://blog.codeship.io/2013/05/21/Testing-Tuesday-6-Top-5-Cucumber-best-practices.html
[cucumber-step-definitions]:http://cukes.info/step-definitions.html
[cucumber-examples]:https://github.com/cucumber/cucumber/tree/master/examples/i18n
[cucumber]:http://cukes.info/
[cuke4duke]:https://github.com/cucumber/cuke4duke
[cucumber-jvm]:https://github.com/cucumber/cucumber-jvm
[cucumber-report]:https://github.com/masterthought/jenkins-cucumber-jvm-reports-plugin-java
