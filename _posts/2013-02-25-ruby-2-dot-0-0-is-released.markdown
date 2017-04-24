---
layout: single
title: "ruby 2.0.0-p0 릴리즈"
date: 2013-02-25 12:19
comments: true
tags: ["ruby", "ruby 2.0"]
---

오늘 ruby 2.0의 첫 안정화 버전인 `ruby 2.0.0-p0`이 릴리즈 되었다: [공식홈페이지][ruby-2.0-is-released]

그와 발맞춰 일명 '곡괭이책'-[Programming ruby 1.9 & 2.0 (4th ed)][programming-ruby-4th-ed] 역시 업데이트되었다.

<!-- more -->

### `ruby 2.0`의 새로운 기능

keyword arguement

: 기존에 hash로 named argument처럼 쓰는 것과 비슷해보이지만, 정의한 keyword arguement가 넘겨지지 않으면 에러가 발생한다.

Module#prepend

: `Module#include`는 include되는 모듈의 메소드를 클래스의 메소드가
override 하지만, `Module#prepend`는 반대로 클래스의 메소드를
모듈의 메소드가 override하게 된다. 즉, 모듈 메소드를 `call chain` 앞에
추가하는 형태가 된다.

: 아래 예를 보면 `Module#include`와 `Module#prepend`의 차이를 확인할 수 있다.

```ruby
    module A; end
    module B; end
    class C
        include A
        prepend B
    end

    C.ancestors #=> [B, C, A, Object, Kernel, BasicObject]
```

Refinements

: Rails도 그렇고 ruby 커뮤니티는 기본 core lib나 stdlib의 클래스를 열어서 기존 메소드를 변경하거나 확장하는 경우가 종종 있는데(e.g. `3.days.ago`), 이런 류의 변경을 Module scope 내에서만 적용할 수 있도록 `Module#refine` 메소드가 추가됨

```ruby
    module NumberQuery
        refine String do
            def number?
                match(/^[1-9][0-9]+$/) ? true : false
            end
        end
    end

    "123".number?  #=> NoMethodError

    module NumberQuery
        p "123".number?  #=> true
    end

    module MyApp
        using NumberQuery

        p "123".number?  #=> true
        p "foo".number?  #=> false
    end
```


symbol list creation

: 심볼 배열을 생성할 수 있는 새로운 literal 표현(`%i`)이 추가됨

```ruby
    # Ruby 1.9:
    KEYS = [:foo, :bar, :baz]

    # Ruby 2.0:
    KEYS = %i{foo bar baz}
```

\_\_dir__

: 현재 실행 중인 파일의 경로를 `File.dirname(__FILE__)`로 쓸 필요가 없어짐. 이제부터는 `__dir__`을 사용하면 됨. (지금까지 왜 없었는지 의문임)

UTF-8 default encoding

: 이제 파일의 default encoding을 UTF-8로 처리함. 파일 첫라인에 매번 `#encoding: UTF-8`을 써줄 필요가 없음.

### 나머지...

그외의 자세한 변경사항은 [릴리즈 노트][ruby-2.0-is-released]를 볼 것.

아래 아티클에서는 Ruby 2.0의 변경점을 예제 코드로 설명하고 있으니 참고.

* [Preview of the new features in Ruby 2.0.0][new features in ruby 2.0.0]
* [Ruby 2.0.0 by Example][ruby2 by example]


### 1.9와의 호환성

`ruby 1.9`가 `ruby 1.8`과 `ruby 2.0` 사이의 과도기적 버전이다 보니,
1.8에서 1.9로 올라올 때만큼의 호환성 문제를 만들것 같지는 않음.

`ruby 1.9`와의 하위 호환에는 큰문제는 없어보이지만 tricky한 코드를 많이 썼다면 관련문서를 확인해볼 필요가 있음.


[ruby-2.0-is-released]: http://www.ruby-lang.org/en/news/2013/02/24/ruby-2-0-0-p0-is-released/

[programming-ruby-4th-ed]: http://pragprog.com/book/ruby4/programming-ruby-1-9-2-0

[new features in ruby 2.0.0]: http://globaldev.co.uk/2012/11/ruby-2-0-0-preview-features/

[ruby2 by example]:http://blog.marc-andre.ca/2013/02/23/ruby-2-by-example/
