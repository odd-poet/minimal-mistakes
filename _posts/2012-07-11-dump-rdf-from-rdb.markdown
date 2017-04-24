---
layout: single
title: "DB를 RDF로 덤프하기"
date: 2012-07-11 14:19
comments: true
tags: [LOD, rdf]
---

[D2RQ]는 RDB 데이터베이스 위에서 Linked Data 서비스를 제공하는 시스템인데,
이 시스템에 포함된 mapping 툴들을 사용하여 RDB의 데이터를 RDF로 dump 해보자.

<!-- more -->

## 데이터 모델 매핑

RDB의 관계형 모델을 RDF 트리플로 매핑하기 위해 W3C의 [R2RML]이라는
언어를 사용할 수 있다. 기본적으로 테이블의 각 Row를 하나의 Subject로 각 컬럼명과 컬럼값을
predicate와 object로 매핑하게 되는데 R2RML은 DB의 스키마를 어떻게 RDF 트리플로 매핑할지 정의하는
언어라고 할 수 있다.

[D2RQ]에는 [R2RML]과 유사한 D2RQ Mapping Language가 있다.
개념적인 부분은 R2RML과 유사하므로 여기서는 D2RQ의 툴을 사용하여
데이터베이스를 RDF로 덤프해본다.


## D2RQ 설치

1. [D2RQ]를 다운로드 받아서 적당한 곳에 압축을 푼다.
2. Java 기반의 툴이므로 JAVA 환경을 준비한다.

## mapping rule 생성

* 참고 : [D2RQ Generate mapping][generate mapping]

D2RQ를 이용하여 DB 스키마에서 매핑룰을 생성하는데
DB 접속정보와 적합한 JDBC 드라이버를 준비하면 된다.

	generate-mapping -o outfile.ttl -u user -p pw -d DRIVER_CLASS_NAME JDBC_URL

위와 같이 수행하면 outfile.ttl 에 아래와 같은 DB-RDF 매핑 정보가 생성된다.
테이블 별 s-p-o 매핑 정보 외에도 DB 접속정보도 포함되어 있음을 확인할 수 있다.

	@prefix map: <#> .
	@prefix db: <> .
	@prefix vocab: <vocab/> .
	@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
	@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
	@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
	@prefix d2rq: <http://www.wiwiss.fu-berlin.de/suhl/bizer/D2RQ/0.1#> .
	@prefix jdbc: <http://d2rq.org/terms/jdbc/> .

	map:database a d2rq:Database;
		d2rq:jdbcDriver "com.mysql.jdbc.Driver";
		d2rq:jdbcDSN "jdbc:mysql://125.209.205.37/tahiti";
		d2rq:username "root";
		d2rq:password "lod";
		jdbc:autoReconnect "true";
		jdbc:zeroDateTimeBehavior "convertToNull";
		.

	# Table nsp_user
	map:nsp_user a d2rq:ClassMap;
		d2rq:dataStorage map:database;
		d2rq:uriPattern "nsp_user/@@nsp_user.user_id|urlify@@";
		d2rq:class vocab:nsp_user;
		d2rq:classDefinitionLabel "nsp_user";
		.
	(이후 생략)


**주의** : mysql은 기본 지원하지만 [cubrid]는 문제가 좀 있다.

## mapping rule 수정

위에서 생성한 mapping rule은 컬럼이름이 predicate으로 그대로 사용되므로
실제 적용시에는 이 파일을 수정하여 원하는 vocabulary를 사용하도록 변경하는 것이 필요하다.

이 매핑 파일은 [D2RQ mapping language][D2RQ lang]으로 기술되어 있지만,
개념적으로는 R2RML과 유사하고 자동생성되는 파일이므로
추후 R2RML로 매핑이 변경되더라도 RDF 덤프 관련 작업에 큰 변경은 없을 것으로 보인다.

([D2RQ language][D2RQ lang] 역시 RDF 이다)

데이터베이스를 RDF로 덤프할 때 대략 아래와 같은 매핑 정보 수정이 예상된다.

1. vocab을 적절한 이름으로 수정
2. 외부키 등의 relation에 대한 매핑 확인 및 수정

## dump!

mapping rule을 수정했으면 dump만 남았다.

mapping rule에 DB접속정보가 포함되어 있으므로
outfile.ttl 파일을 [dump rdf] 툴을 사용하여 DB를 rdf로 덤프해보자

	dump-rdf -b baseURI -o rdf_output mapping-file.ttl

__mapping-file.ttl__에 위에서 만든 mapping rule파일명을 쓰면 되는데,
format을 지정하지 않으면 기본 값이 "N-TRIPLE" 포맷으로 출력된다.



[R2RML]: http://www.w3.org/TR/r2rml
[D2RQ]: http://d2rq.org/
[D2RQ lang]: http://d2rq.org/d2rq-language "D2RQ Mapping Language"
[generate mapping]: http://d2rq.org/generate-mapping
[dump rdf]:  http://d2rq.org/dump-rdf
[cubrid]: http://www.cubrid.com/
[LOD2]: http://lod2.eu
