---
layout: single
title: "UnitTest에 embeded HSQL 사용하기"
date: 2013-11-29 04:03
comments: true
tags: [test, testing, hsql, spring, mybatis, embeded DB]
---

현재 회사(SKP)의 웃기는 네트웍 정책 덕에 예전보다 unit test 뿐아니라
개발환경에서도 embeded DB를 더 자주 사용하고 있는 편인데,
UnitTest 용으로 embeded DB로 HSQL을 사용할 때 SQL 호환성 이슈를 피하는 방법을 정리한다.
(물론 Spring+MyBatis+MySQL 환경)

<!--more-->

일단 아래와 같이 embeded DB로 HSQL을 설정한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd">

    <jdbc:embedded-database id="dataSource" type="HSQL">
        <jdbc:script location="classpath:database/schema.sql"/>
        <jdbc:script location="classpath:database/init-data.sql"/>
    </jdbc:embedded-database>
</beans>
```
보통 위와 같이 Table 생성하는 스크립트(`schema.sql`)와 초기 데이터를 생성하는 스크립트(`init-data.sql`)을 나누어서 작성한다.

HSQL의 기본 SQL syntax는 다른 Database들과 완전히 호환되지 않는다.
따라서 실제 운영 및 개발환경에서 MySQL을 사용하는 경우 테이블 생성 스크립트 부터 HSQL에서는 정상적으로 실행되지 않는다.

예를 들면, HSQL은 `AUTO_INCREMENT` 키워드 대신 `IDENTITY`라는 키워드를 사용한다.
(`IDENTITY` 키워드는 `PRIMARY KEY`의 의미도 가지기 때문에 `AUTO_INCREMENT`와 완전히 동일하지도 않다)

```sql
-- MySQL
CREATE TABLE sample (
   id INTEGER AUTO_INCREMENT,
   name VARCHAR(100),
   PRIMARY KEY (id)
);

-- HSQL
CREATE TABLE sample (
   id INTEGER IDENTITY,
   name VARCHAR(100)
);
```

다행히도 HSQL은 Oracle, MSSQL, MySQL등의 다른 Database의 syntax를 지원하는 기능이 있다.
그래서 아래처럼 `schema.sql`을 작성하면 HSQL에서 MySQL의 syntax를 그대로 쓸 수 있다.
(HSQL 2.0 이상에서 지원)

```sql
-- for mysql
SET DATABASE SQL SYNTAX MYS TRUE;
-- for oracle
-- SET DATABASE SQL SYNTAX ORA TRUE;

CREATE TABLE sample (
   id INTEGER AUTO_INCREMENT,
   name VARCHAR(100),
   PRIMARY KEY (id)
);
```

하지만 이런 기능에는 빈틈이 있기 마련인데, 이걸로는 `<selectKey>`를 사용할 때 문제가 생긴다.
즉, Insert된 후 auto increment인 column의 값을 가져오는 쿼리에 대한 호환성 문제를
해결해주지는 않는다. 참고로 MySQL에서는 `SELECT LAST_INSERT_ID()`라는 sql을 사용하고,
HSQL에서는 `CALL IDENTITY()`라는 sql을 사용한다.

HSQL을 UnitTest용으로 사용할 때 `selectKey`의 문제는 JDBC 3.0의 `getGeneratedKey()`
기능을 사용하여 피할 수 있다. (ibatis에서는 안되고 mybatis에서만 가능하다는 얘기)

```java
@lombok.Data
public class Sample {
    private Integer id;
    private String name;
}
```

위와 같은 Model 클래스를 Insert하는 SQL map을 아래와 같이 쓸 수 있다.
`<selectKey>` 엘리먼트를 사용하지 않고 `useGeneratedKeys`라는 속성을 사용하면
argument로 넘어왔던 `Sample` 객체의 `id`값이 채워진다.

```xml
<!-- selectKey를 쓰지 않고, JDBC의 getGeneratedKeys를 사용한다. -->
<insert id="insertSample" parameterType="Sample" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO sample
    (name) values #{name}
</insert>
```
그 외에 HSQL의 SQL 호환기능에 대한 유의사항은 아래 링크를 참고한다.

 - [HSQL: Compatibility with Other RDBMS][hsql-compatibility]

[hsql-compatibility]:http://hsqldb.org/doc/2.0/guide/management-chapt.html#mtc_compatibility_mysql
