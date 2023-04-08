---
layout: post  
title: MySQL Index 2  
author: Hyeon Uk Cho  
date: 2023-04-09 05:09:00 +09:00  
categories: [MYSQL]  
tags: [clustering index, unique index]  
math: true  
mermaid: true  
image: /assets/img/post-header/mysql.png  
path:   
lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA  
alt:
---

<h3 data-toc-skip>한줄 정리</h3>
결국 경험이 중요하다. 인덱스 관련 챕터를 읽고 나서 모든 케이스를 고려하면서 테이블을 생성, 인덱스를 추가 하기에는 시간, 실력 모두 부족하다.
경험이 중요..

<h3 data-toc-skip>함수 기반 인덱스</h3>
- 컬럼의 값을 변형해서 만들어진 값에 대해 인덱스를 구축해야 할 때 사용.
- 가상 컬럼을 이용한 인덱스, 함수를 이용한 인덱스
- 인덱싱할 값을 계산하는 과정의 차이만 있을 뿐, 실제 인덱스의 내부적인 구조 및 유지관리 방법은 B-Tree 인덱스와 동일하다.  

```sql
-- 가상 컬럼을 이용한 인덱스
create table user (
    user_id bigint,
    first_name varchar(10),
    last_name varchar(10),
    primary key (user_id)
);

alter table user 
    add full_name varchar(30) as (concat(first_name, ' ', last_name)) virtual,
    add index ix_fullname (fullname);

explain select * from user where full_name='Matt Lee';
```

```sql
-- 함수를 이용한 인덱스
create table user (
    user_id bigint,
    first_name varchar(10),
    last_name varchar(10),
    primary key (user_id),
    index ix_fullname (concat(first_name, ' ', last_name))
);

explain select * from user where concat(first_name, ' ', last_name)='Matt Lee';
-- 만약 ix_fullname을 안탄다면?,
-- collation_connection, collation_database, collataion_server 값을 셋팅하고 해보자.
```

<h3 data-toc-skip>멀티 밸류 인덱스</h3>
```sql
create table user (
    user_id bigint,
    first_name varchar(10),
    last_name varchar(10),
    credit_info json,
    primary key (user_id),
    index mx_creditscores ( (CAST(credit_info -> '$.credit_scores' as unsigned array)) )
);

-- member of(), json_contain(), json_overlaps() 사용해야 인덱스 사용 가능
explain select * from user where 360 member of(credit -> '$.credit_scores');
```

<h3 data-toc-skip>클러스터링 인덱스</h3>
- 프라이머리 키 값이 비슷한 레코드끼리 묶어서 저장하는 것을 클러스터링 인덱스라고 표현.
- 테이블의 프라이머리 키에 대해서만 적용되는 내용.
- 프라이머리 키 값에 의해 저장 위치가 결정된다는 것.
- 프라이머리 키 값이 없다면?
  1. 프라이머리 키가 있으면 기본적으로 프라이머리 키를 클러스터링 키로 선택.
  2. NOT NULL 옵션의 유니크 인덱스(UNIQUE INDEX) 중에서 첫번째 인덱스를 클러스터링 키로 선택.
  3. 자동으로 유니크한 값을 가지도록 증가되는 컬럼을 내부적으로 추가한 후, 클러스터링 키로 선택
- 세컨더리 인덱스는 프라이머리 키 값을 저장하도록 되어 있어 프라이머리 키 값을 기준으로 움직인다.
- 장점
  - 프라어머리 키(클러스터링 키)로 검색할 떄 처리 성능이 매우 빠름(특히, 프라이머리 키를 범위로 검색하는 경우 매우 빠름 
  - 테이블의 모든 세컨더리 인덱스가 프라이머리 키를 가지고 있기 때문에 인덱스만드로 처리될 수 있는 경우가 많음.(인덱스 커버링)
- 단점
  - 테이블의 모든 세컨더리 인덱스가 클러스터링 키를 갖기 때문에 클러스터링 키 값의 크기가 클 경우 전체적으로 인덱스의 크기가 커짐.
  - 세컨더리 인덱스를 통해 검색할 때 프라이머리 키로 다시 한번 검색해야 하므로 처리 성능이 느림.
  - INSERT 할 떄, 프라이머리 키에 의해 레코드의 저장 위치가 결정되기 떄문에 처리 성능이 느림.
  - 프라이머리 키를 변경할 때 레코드를 DELETE하고 INSERT하는 작업이 필요하기 때문에 처리 성능이 느림.
- 주의 사항
  1. 클러스터링 인덱스 키의 크기, 프라이머리 키는 신중하게
  2. 프라이머리 키는 auto-increment보다는 업무적인 컬럼으로 생성(가능한 경우), 설령 그 컬럼의 크기가 크더라도 업무적으로 해당 레코드를 대표할 수 있다면 그 컬럼을 프라이머리 키로 설정하는 것이 좋다.
  3. 프라이머리 키는 꼭 명시
  4. auto-increment 컬럼을 인조 식별자로 사용할 경우, 프라이머리 키가 여러 복합 키로 만들어지는 경우 세컨더리 인덱스가 필요하지 않다면 그대로 이용하지만 세컨더리 인덱스도 필요한 상황이라면 auto-increment로 사용.

<h3 data-toc-skip>유니크 인덱스</h3>
- 유니크 인덱스와 일반 세컨더리 인덱스의 비교
  - 읽기 : 사실 유니크 인덱스가 더 빠른 것이 아니다. 단지, 유니크하지 않은 인덱스는 중복된 값이 허용되어 여러 값을 읽어야하기 때문에 느린 것.
  - 쓰기 : 유니크 인덱스가 중복된 값 체크를 한번 더 해야하기 때문에 더 느리다. 
  - 결론) 필요하다면 사용하자

<h3 data-toc-skip>외래키</h3>
- innodb에서만 사용 가능하며, 외래키가 설정되면 인덱스까지 설정된다.
- 테이블의 변경(쓰기 잠금)이 발생하는 경우에만 잠금 경합이 발생.
- 외래키와 연관되지 않은 컬럼의 변경은 최대한 잠금 경합을 발생시키지 않는다.
- 되도록 설정하지 말자.


