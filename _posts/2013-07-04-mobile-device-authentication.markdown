---
layout: single
title: "모바일 디바이스 인증"
date: 2013-07-04 16:24
comments: true
tags: ["mobile", "device id", "device authentication"]
---

모바일앱들은 사용자를 유치할 때 가입 허들을 낮추기 위해서 디바이스 식별자(device id)를 통해 사용자를 식별하고 인증하는 방식을 취해서 가입절차를 간소화하고 있다. 모바일 단말에 있는 다양한 식별자들을 살펴보고 모바일앱에서 사용자 식별에 적절한 것은 어떤 것인지 살펴본다.

(난 이동통신 관련해서는 문외한이고 구글링한 것들로 정리한 거라 잘못된 정보가 있을 수 있다.)

<!-- more -->

### 모바일 디바이스 식별자에 대한 용어 정리

#### 제조사 부여 식별자

* **ESN (Electronic Serial Number)**
    1. 단말제조업체가 단말기에 부여하는 고유번호.
    2. 단말기를 켜거나 전화를 걸때 기지국을 통해서 사업자에게 전달되며, 사업자는 이를 사용하여 사용자 식별, 통화, 인증 및 과금 등에 사용한다.
    3. 국제 로밍을 고려할 경우 세계적으로 고유한 번호이어야 하므로 미국의 TIA(Telecommunication Industry Association)에 그 관리와 배포를 위임하고 있다.
    4. 총 32bit으로 구성되며 MC(manufacturer code)가 8bit인 것과 14bit인 경우의 두 가지가 있다. TIA가 보유하는 ESN 자원이 고갈되어감에 따라 MEID(Mobile Equipment Identifier)로 전환되었다.

    ![ESN 구성](/assets/include/2013-07/ESN.png)

* **MEID(Mobile Equipment Identifier)**
    1. CDMA의 ESN 고갈에 대한 대안으로 만들어짐
    2. 아래와 같이 56bit로 구성됨 (Hexadecimal이므로 4 * 14 = 56bit, 마지막 CD는 check sum으로 식별자 아님)
    3. 맨 앞의 RR은 Regional code로 GHA(Global Hexadecimal MEID Administrator)에 의해 관리되는데 `00-99`는 3GPP에서 사용되는 IMEI(International Mobile Equipment Identity)를 위해 예약, `A0-FF`는 cdma2000을 위해 reserve 됨

    ![MEID 구성](/assets/include/2013-07/MEID.png)

* **IMEI (International Mobile Equipment Identity)**
    1. 3GPP(GSM/WCDMA)에서 사용하는 코드14자 + 체크용 1자 = 총 15자의 Hex값
    2. MEID와 동일 포맷이지만, CD(check sum)이 포함되며 RR코드의 영역이 다름

#### 통신사 관련 식별자

* **IMSI(Internaltional Mobile Subscriber Identity)**
    1. GSM 서비스 가입시 이동 단말에 할당되는 고유 식별번호(15자리)
    2. MCC, MNC, MSIN 으로 구성
        * *MCC(Mobile Country Code)* : 이동국가코드
        * *MNC(Mobile Network Code)* : 이동 네트워크 코드
        * *MSIN(Mobile Subscriber Identifier Number)* : 이동 가입자 식별번호
    3. MCC+MNC는 가입자의 Home Network를 전세계 어떤 망에서든 유일하게 식별. 즉 로밍서비스에서 첫 6자리를 분석하여 Home Network를 조회할 수 있다.
    4. MSIN은 MCC와 MNC가 주어진 경우 이동전화 단말기를 유일하게 식별한다.

    ![](/assets/include/2013-07/IMSI.gif)

* **TMSI(Temporary Mobile Subscriber Identity)**
    1. 임시 이동 가입자 식별번호
    2. 이동 통신 시스템에서 이동전화단말을 식별하는 임시 식별 번호.
    3. 홈 위치 레지스터(HLR)의 인증센터(AC/Auc)에 의해 부여 된다.
    4. 모바일 단말과 이동전화교환국(MSC) 사이에서 보안삭 IMSI 대신 사용된다.
    5. 이동전화단말 통신권 내에서 유효하고, 통신권 외에서는 추가적인 위치 영역 식별(LAI:Location Area Identification)이 필요하다.

* **MIN(Mobile Identification Number)**
    1. 이동전화단말 식별번호
    2. 10자리 전화번호를 디지털로 표시하는 34비트 숫자로 MIN1과 MIN2로 구성
    3. MIN1은 단말기에 할당된 7개의 digit 전화번호로 24개 비트로 구성
    4. MIN2는 3개의 digit으로 10개 비트로 구성된다.
    5. `011-YYY-XXXX`에서 MIN1은 `YYY-XXXX`이고, MIN2는 `011`이다

* **MSISDN(Mobile Station International ISDN Number)**
    1. 이동전화단말 국제 ISDN 번호
    2. WCDMA IMT-2000에서는 가입자에게 2개 번호를 부여하는데, USIM 카드의 IMSI와 단말기의 MSISDN가 있다. MSISDN에는 국가코드가 포함되어 있다.
    3. GSM 네트웍에서 전화번호란 MSISDN(Mobile Station Integrated System Digital Network)을 뜻하며 국가코드, 네트웍코드, 디렉토리번호로 구성되어 있다.
    4. 반면 IS-41C 네트웍에서의 전화번호란 휴대폰의 MIN(Mobile Identification Number)를 뜻하며 지역번호(Area Code)와 전화번호로 구성되어 있고, Nxx-xxxx 형태를 갖고 있다. MDN(Mobile Directory Number)와 동일한 것으로, 가입자 전화번호를 의미한다.


### 디바이스 식별자가 인증(Authentication)에 문제가 되는 경우들

요즘 휴대폰 외에도 타블렛과 같은 모바일 디바이스를 함께 사용하는 사람들이 많아졌다. 이런 사용자들의 패턴을 봤을 때 위에서 설명한 식별자들을 인증을 위한 ID로 사용하면 다음과 같은 문제가 발생할 수 있다.

* **제조사가 생성한 식별자를 사용하는 경우**
    * 전화기가 아닌 단말은 ESN, IMEI, MEID 등을 가지고 있지 않다.
    * Wifi의 MAC address는 wifi가 꺼져있을 경우 올바른 주소값을 얻을 수 없는 기기가 있다.
    * 디바이스에 영구적으로 할당된 유일한 식별자이므로 완전히 공장 초기화가 되더라도 값이 바뀌지 않는다. 즉, 사용자 간의 기기 양도 및 거래가 될 경우 인증을 위한 ID로 서버에 보관하고 있을 경우 문제가 된다.

* **이동통신망 관련 식별자를 사용하는 경우**
    * 타블렛 PC 등 전화기가 아닌 모바일 디바이스들을 식별할 수 없다

* **iOS에서 UDID(Unique Device ID)를 사용하는 경우**
    * iOS에는 UDID라는 단말기 식별자(40자리)가 존재하는데, 개인정보 논란을 비롯해서 다른 광고플랫폼에서 트래픽 분석에 사용되는 등 문제가 됨에 따라 애플은 iOS5 부터 UDID를 deprecated 시켰다. 게다가 2013.06 부터 UDID를 사용하는 앱은 심사에서 reject되고 있다.
    * 또한 UDID 역시 제조사(애플)의 단말기 식별자이므로 인증을 위한 사용자 식별자로는 부적절하다.

* **Android에서 `Settings.Secure.ANDROID_ID`를 사용하는 경우**
    * `ANDROID_ID`는 64bit 크기의 고유값인데, 디바이스 초기화할 때 리셋되므로 일반적인 제조사 식별자가 가지는 문제(e.g. 기기 양도)는 없다.
    * 하지만 안드로이드2.2(프로요) 이전의 단말의 경우 값을 신뢰할 수 없고, 메이저 제조사에서 제작한 인기 모델의 모든 단말이 같은 `ANDROID_ID` 값을 가지는 버그가 있어서 사용에 위험이 따른다.

### 그럼 디바이스 인증에는 어떤 식별자를 사용해야 하는가?

모바일앱의 디바이스 인증에 사용할 ID값은 일반적으로 다음과 조건을 만족시켜야 한다.

1. Global Uniqueness를 보장할 수 있는 값이어야 한다.
2. 개인정보로 취급되지 않는 식별자를 사용해야 한다. (MAC address의 경우 관점에 따라 개인정보로 취급되는 경우가 있다 )
3. 제조사의 Serial Number와 같이 영속적인 값은 피해야 한다. (사용자간의 기기 양도시의 문제)
4. 기기정보 기반의 값(e.g.통신사의 식별자)은 지원할 수 있는 기기의 제한을 만들 수 있으므로 피한다.

정신을 혼미하게 하는 이상한 용어들을 많이 늘어놨지만, 결론은 단순하다. 디바이스 인증을 위한 식별자로써 UUID(GUID)를 사용하는 것이다. 즉, 앱 설치 후 최초 실행시에 UUID(Unique Device ID)를 생성해서 저장하고 그 값을 디바이스 식별자로 사용한다.

애플 역시 iOS5에서 UDID(Unique Device ID)를 depricate 시키면서 UUID(Universally Unique Identifier)을 사용을 권고했고, Android의 커뮤니티 역시 UUID를 사용하는 것을 일반적으로 권장한다. 자세한건 [여기][ios-udid-policy] 참고.

인증을 위한 디바이스 식별자에 UUID를 사용하면 개인정보와 관련된 이슈를 피할 수 있을 뿐아니라, 다양한 시나리오를 개발자가 직접 컨트롤 할 수 있다.

#### One User - Multi Device 를 고려할 때 주의할 점들

이번에 개발할 통합인증 시스템 관련해서 생각나는 대로 정리한 이슈들...

* 하나의 디바이스로 두 명이상의 사용자의 로그인을 지원한다면(가정에서 iPad를 사용할 경우)?
    * 혹은 두 사람이 하나의 디바이스에서 로그인/로그아웃하여 공용으로 사용한다면?
    * user-device 매핑이 1:N이 뿐만아니라 N:1이 될 수도 있음.
* MIN/MDN을 사용할 때와는 달리 UUID 만으로는 실제 사용자 정보를 알아낼 수 없음
    * MIN/MDN 등의 정보는 별도로 통신하여 기존 사용자 정보와 매핑해야 함.
* UUID의 라이프사이클은 어디서 결정할까?
    * 앱에서 생성하고 폐기? 아니면 서버에서?
    * MIN/MDN 같은 식별자를 받아서 필요한 사용자 매핑 등을 처리하고 UUID는 서버에서 생성할까? 그렇다면 전화기가 아닌 단말에서는 별도의 API를 제공해야 하나?

### References

* [단말기 ESN번호 고갈과 이에 대한 대책][ENS-MEID]
* [이동통신 각종 식별번호 개념잡기(IMSI, MSISDN, MIN등)][IMSI-MSISDN-MIN]
* [(번역) 안드로이드 개별 디바이스를 구분하는 방법][android-identifying-app-installations]
* [iOS5의 UDID정책 변경에 대한 퀵 리뷰][ios5-udid-policy]
* [iOS CFUUID Reference][cfuuid-ref]
* [앱 스토어 2013년 5월 부터 UDID 사용 제한 정책을 강화했다][ios-udid-policy]

[ios-udid-policy]:http://10apps.tistory.com/archive/20130619

[cfuuid-ref]:http://developer.apple.com/library/ios/#DOCUMENTATION/CoreFoundation/Reference/CFUUIDRef/Reference/reference.html

[ios5-udid-policy]:http://dev.kthcorp.com/2011/09/07/ios5-uuid-udid/

[ENS-MEID]:http://committee.tta.or.kr/data/weekly_view.jsp?news_id=943

[IMSI-MSISDN-MIN]:http://blog.naver.com/PostView.nhn?blogId=cache798&logNo=130011558584&parentCategoryNo=47&viewDate=&currentPage=1&listtype=0

[android-identifying-app-installations]:http://huewu.blog.me/110107222113
