---
layout: post
title: SQL vs NoSQL
date: 2022-01-09 05:38:00 +0900
description: SQL 과 NoSQL 기반 데이터베이스들을 알아봤어요\ # Add post description (optional)
img: database.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Database, SQL, NoSQL]
---

# SQL vs NoSQL 비교

[SQL vs. NoSQL: Database Differences Explained | Blog | Fivetran](https://www.fivetran.com/blog/sql-vs-nosql-database-differences-explained)

위 링크를 참고한 글 입니다.

## SQL 데이터베이스

관계형 데이터베이스 라고도 불린다. Structured Query Language(구조화된 질의 언어) 라고 불리고, 1970년대 부터 개발되어 아직까지 많은 기업에서 사용하고 있다. 

OLTP (Online Transaction Processing) 서비스에서 활용하면 좋다.

모든 관계형 데이터베이스는 테이블에 있는 데이터를 변경(mutate)하기 위한 작업(트랜잭션)이 이루어진다. 현대 관계형 데이터베이스는 이들 트랜잭션이 동시적으로 일어나는 것을 지원한다. 

이들 트랜잭션은 4가지 특징에 중점을 두고 있다. 

- Atomicity(원자화)
    - 모든 작업이 반영되는 특성
- Consistency(일관성)
    - 미리 정의된 규칙에서만 데이터가 수정되는 특성
- Isolation(고립성)
    - 동시에 일어나는 트랜잭션에 대해서 상대 트랜잭션의 접근 정도
- Durability(지속성)
    - 한번 커밋된 트랜잭션의 내용은 영구 적용되는 특성

ACID 라고 불리는 이 특성을 중점으로 관계형 데이터베이스가 설계되었다. 

### 고립화 수준의 문제 정의

- Dirty Read
    - 트랜잭션이 아직 진행 중인(즉 Commit 명령이 발생하지 않은) 데이터에 대해서 다른 트랜잭션이 읽을 수 있는 문제
- Non-Repeatable Read(=Inconsistent Analysis)
    - 한 트랜잭션을 수행할 때, 그 사이 다른 트랜잭션이 값을 수정한 것을 모르고  두 쿼리의 결과가 비일관적이게 나오는 문제
- Phantom Read
    - 테이블의 레코드를 읽을 때 첫 쿼리에서 없던 레코드가 두 번째 쿼리에서 다른 트랜잭션에 의해 생성되어 나타나는 문제
- Serialization Anomaly
    - 동시에 처리되는 두 개의 트랜잭션에서 하나의 트랜잭션이 처리한 내용이 다른 트랜잭션에 의해 누락되어 버리는 문제
        
        
    
    위 문제들에 대한 예시는 여기를 참고하면 좋다. 
    
    [Transaction Isolation in PostgreSQL](https://pgdash.io/blog/postgres-transactions.html)
    

데이터의 안전성을 고립화 수준 4레벨로 나누어 지원한다. 

- 레벨 0 (Read Uncommitted)
    - 아직 Commit 되지 않은 데이터를 다른 트랜잭션이 읽는 것을 허용하는 수준이다.
    - 데이터의 고립화가 아에 되지 않는 고립화 수준이다.
    - 따라서 Dirty Read, Non-Repeatable Read, Phantom Read 모두 발생한다.
- 레벨 1 (Read Committed)
    - 트랙잭션이 Commit 되어 확정된 데이터만 읽는 것을 허용하는 수준이다.
    - 고립화 수준의 문제에서 Dirty Read 를 방지한다.
    - 대부분의 관계형 데이터베이스들이 이 수준을 기본 값으로 설정해 놓았다.
    - Lock을 통한 구현(SQL Server, DB2)과 Undo를 활용한 구현(Oracle)방식이 있다.
- 레벨 2 (Repeatable Read)
    - 선행 트랜잭션이 처리 중인 데이터에 대해서는 후행 트랜잭션이 갱신 및 삭제 등을 불허하면서 같은 데이터를 두 번 쿼리했을 때 일관성있는 결과를 반환한다.
    - Non-Repeatable Read까지 방지해 준다.
    - 데이터에 걸린 공유 Lock을 Commit할 때까지 유지하는 방식으로 구현
- 레벨 3 (Serializable Read)
    - 선행 트랜잭션이 읽은 데이터를 후행 트랜잭션이 갱신/삭제/삽입하는 것을 막는다.
    - 완벽한 읽기 일관성을 제공한다.
    - Phantom Read 까지 방지한다.

## NoSQL 데이터베이스

NoSQL 데이터베이스는 기존 관계형 데이터베이스가 지원하는 SQL을 활용하지 않는 데이터베이스들을 통합해서 부르는 것이다. 

네트워크 컴퓨팅의 발전으로 지역별로 관리되는 데이터베이스들이 많아졌는데, 이들을 연동하는 과정에서 기존 시스템에서는 100% 유효성을 검증하기 힘들어졌다. 

이들을 관리하기 위해서는 일관성, 유효성, 그리고 네트워크 및 하드웨어 오류 발생 시에도 관대한 데이터베이스 시스템이 필요했다. 

따라서 Horizontal Scaling을 위해 기존 ACID 대체하는 BASE(Basically Available, Soft State, Eventually Consistent - 기본으로 유효한, 유연한, 결국 일관적인) 를 기반으로 한 NoSQL 또는 비관계형 데이터베이스를 생각하게 되었다. 

이들 NoSQL 기반 데이터베이스들은 Event-Driven 아키텍쳐로 여러 네트워크 상에 분산되어 있는 데이터베이스들의 트랜잭션을 스케일링하기 위해서 사용된다.

Document, Key-Value, Graph, Wide-Column 등의 데이터베이스 종류가 있다.

- Document stores
    - MongoDB, Firebase, Couchbase
    - 각 문서(Document)가 고유하며 시간에 따라 변하는 카탈로그, 사용자 정보, 컨텐트 관리 시스템 등에 유용하다.
    - JSON, XML 등의 형태로 문서를 관리한다.
    - 스케일 가능한 구조로 짜여있기 때문에 스타트업 등에 유용하다
    - 데이터의 고유성 등 ACID 적용이 필요한 데이터에는 적절하지 않을 수 있다.
        - 최근 MongoDB는 ACID 트랜잭션을 지원한다
- Key-Value stores
    - Redis, Memcached, Amazon DynamoDB
    - 기존 정의된 스키마에 의존하는 것이 아닌, 키-값 관계를 정의하여 데이터를 저장한다.
    - Horizontal Scaling 에 유용하다.
- Graph stores
    - Graphbase, Dgraph, InfoGrid
    - 각 노드(사람, 장소와 같은 데이터)가 고유 식별자가 있고, 다른 노드 사이의 엣지(노드 사이의 관계relationship으로 표현)로 이어져 있다.
- Wide-Column stores
    - Apache Cassandra, Apache Hbase, MS Azure Table
    - Extensive record stores라고도 불린다.
    - 관계형 데이터베이스와 유사하게 Row & Column 으로 이뤄져있다.
    - 관계형 데이터베이스와 다르게 데이터가 Column 기준으로 저장된다.
    - 따라서 대용량의 데이터를 다루는 OLAP(Online Analytical Processing-빅데이터 관련 작업 등)에 용이하다
    - 스키마가 다른 NoSQL 기반 데이터베이스와 같이 필요 없다.

## 추가 자료

[DB-Engines Ranking](https://db-engines.com/en/ranking)

데이터 베이스 랭킹