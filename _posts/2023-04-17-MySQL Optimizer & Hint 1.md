---
layout: post  
title: MySQL Optimizer & Hint 1  
author: Hyeon Uk Cho  
date: 2023-04-17 05:09:00 +09:00  
categories: [MYSQL]  
tags: [optimizer, hint]  
math: true  
mermaid: true  
image: /assets/img/post-header/mysql.png  
path:   
lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA  
alt:
---

<h3 data-toc-skip>한줄 정리</h3>
인덱스를 적절히, 무거운 쿼리는 최대한 지양

<h3 data-toc-skip>쿼리 실행 절차</h3>
- 사용자로부터 요청된 SQL 문장을 잘게 쪼개서 MySQL 서버가 이해할 수 있는 수준으로 분리한다.
- SQL 파싱 정보를 확인하면서 어떤 테이블부터 읽고 어떤 인덱스를 이용해 테이블을 읽을지 선택한다. (옵티마이저가 처리)
- 두 번째 단계에서 결정된 테이블의 읽기 순서나 선택된 인덱스를 이용해 스토리지 엔진으로부터 데이터를 가져온다.  

<h3 data-toc-skip>옵티마이저 종류</h3>
- 규칙 기반 최적화
- 비용 기반 최적화

<h3 data-toc-skip>기본 데이터 처리</h3>
- 풀 테이블 스캔은 주로 아래와 같은 조건에서 주로 선택한다.
  1. 테이블의 레코드 건수가 너무 작아서 인덱스를 통해 읽는 것보다 풀 테이블 스캔을 하는 편이 더 빠른 경우
  2. where 절이나 on 절에 적절한 조건이 없는 경우
  3. 인덱스 레인지 스캔을 사용할 수 있는 쿼리라고 하더라도 옵티마이저가 판단한 조건 일치 레코드 건수가 너무 많은 경우(통계정보기준)
- innodb에서는 특정 테이블의 연속된 데이터 페이지가 읽히면 백그라운드 스레드에 의해 리드 어헤드 작업이 자동으로 시작된다. (리드 어헤드란 어떤 영역의 데이터가 앞으로 필요해지리라는 것을 예측해서 요청이 오기 전에 미리 디스크에서 읽어 innodb의 버퍼 풀에 가져다 두는 것을 의미한다.)  
즉, 풀 테이블 스캔이 실행되면 처음 몇 개의 데이터 페이지는 포그라운드 스레드가 페이지 읽기를 실행하지만 특정 시점부터는 읽기 작업을 백그라운드로 넘긴다. 최대 64 페이지까지 읽어서 버퍼 풀에 저장해 둔다.
- innodb_read_ahead_threshold 시스템 변수를 통해 설정 가능.

<h3 data-toc-skip>병렬 처리</h3>
- 테이블의 전체 건수를 가져오는 쿼리만 병렬로 처리할 수 있다.
- innodb_parallel_read_threads 변수로 하나의 쿼리를 최대 몇개의 스레드를 이용해서 처리할지를 변경할 수 있다.

<h3 data-toc-skip>ORDER BY 처리</h3>
1. 인덱스를 이용하는 방법
2. Filesort를 이용하는 방법(explain으로 Extra 컬럼에 'Using filesort'라고 표시)  
- 인덱스 이용 
  - 장점 : INSERT, UPDATE, DELETE 쿼리가 실행될 때 이미 인덱스가 정렬되어 있어서 순서대로 읽기만 하면 되므로 매우 빠르다.
  - 단점 : INSERT, UPDATE, DELETE 작업 시 부가적인 인덱스 추가 작업이 필요하므로 느리다. 인덱스 때문에 디스크 공간이 더 많이 필요하다. 인덱스의 개수가 늘어날수록 innodb의 버퍼 풀을 위한 메모리가 많이 필요하다
- Filesort 이용
  - 장점 : 인덱스를 생성하지 않아도 되므로 인덱스를 이용할 때의 단점이 장점으로 바뀐다.  정렬해야 할 레코드가 많지 않으면 메모리에서 Filesort가 처리되므로 충분히 빠르다 
  - 단점 : 정렬 작업이 쿼리 실행 시 처리되므로 레코드 대상 건수가 많아질수록 쿼리의 응답 속도가 느리다.

<h3 data-toc-skip>소트 버퍼</h3>
- 정렬을 수행하기 위해 별도의 메모리 공간을 할당받아서 사용.
- sort_buffer_size로 설정 가능
- 쿼리의 실행이 완료되면 즉시 시스템으로 반납
- 정렬해야하는 레코드의 수가 소트 버퍼보다 크다면? 임시로 디스크에 기록해두고 각 버퍼의 크기만큼 머지하여 정렬을 수행 (Sort_merge_passes 상태 변수에 누적되어 집계됨)
- sort_buffer_size 크기가 무조건 클수록 처리가 빨라지는 것은 아니다.
- 10MB 이상으로 설정하면 대량의 레코드를 정렬하는 여러 커넥션에서 동시에 실행되면서 운영체제는 메모리 부족 현상을 겪을 수 있다. 이 경우 운영체제의 OOM-Killer가 여유 메모리를 확보하기 위해 프로세스를 강제로 종료한다. 메모리를 가장 많이 사용하는 프로세스를 종료하기 때문에 일반적으로 MySQL 서버가 강제 종료 1순위가 된다.

<h3 data-toc-skip>정렬 알고리즘</h3>
- 투패스
  - <sort_key, rowid> 정렬 키와 레코드의 로우 아이디만 가져와서 정렬 수행.
  - 다시 프라이머리 키로 레코드를 가져오는 정렬 방식.
- 싱글패스
  - <sort_key, additional_fields> 정렬 키와 레코드 전체를 가져와서 정렬하는 방식으로, 레코드의 컬럼들은 고정 사이즈로 메모리 저장
  - <sort_key, pecked_additional_fields> 정렬 키와 레코드 전체를 가져와서 정렬하는 방식으로, 레코드의 컬럼들은 가변 사이즈로 메모리 저장
  - 소트버퍼에 정렬 기준 컬럼을 포함에 SELECT 대상이 되는 컬럼 전부를 담아서 정렬을 수행하는 정렬 방식
- 투패스가 2번 읽어야해서 불합리적이지만, 싱글패스는 더 많은 소트 버퍼가 필요하다.
- 투패스를 사용할 경우
  - 레코드의 크기가 max_length_for_sort_data 시스템 변수에 설정된 값보다 클 떄
  - BLOB이나 TEXT 타입의 컬럼이 SELECT 대상에 포함될 때

<h3 data-toc-skip>정렬 처리 방법</h3>

| 정렬 처리 방법                    | 실행 계획의 Extra 컬럼 내용              |
|-----------------------------|---------------------------------|
| 인덱스를 사용하는 정렬                | 별도 표기 없음                        |
| 조인에서 드라이빙 테이블만 정렬           | Using filesort                  |
| 조인에서 조인 결과를 임시 테이블로 저장 후 정렬 | Using temporary; Using filesort |

- 인덱스를 사용하는 정렬
  - ORDER BY에 명시된 컬럼이 제일 먼저 읽는 테이블에 속하고, ORDER BY의 순서대로 생성된 인덱스가 있어야 한다.
  - WHERE 절에 첫 번째로 읽는 테이블의 컬럼에 대한 조건이 있다면 그 조건과 ORDER BY는 같은 인덱스를 사용할 수 있어야 한다.
  - 해시 인덱스나 전문 검색 인덱스 등에서는 인덱스를 이용한 정렬을 사용할 수 없다.

- 조인에서 드라이빙 테이블만 정렬
  - 조인을 실행하기 전에 첫 번째 테이블의 레코드를 먼저 정렬한 다음 조인을 실행하는 것
  - 드라이빙 테이블의 컬럼만으로 ORDER BY 필요.

- 임시 테이블을 이용한 정렬
  - 임시 테이블에 저장하고, 그 결과를 다시 정렬하는 과정

<h3 data-toc-skip>정렬 처리되는 과정</h3>
- 스트리밍 방식 : 조건에 일치하는 레코드가 검색될 때마다 바로바로 클라이언트로 전송해주는 방식
- 버퍼링 방식 : ORDER BY, GROUP BY 같은 처리는 쿼리의 결과가 스트리밍 되는 것을 불가능하게 한다. 먼저 결과를 모아서 서버에서 일괄 가공한다.
- JDBC는 버퍼링 방식
- 인덱스를 이용한 정렬 방식은 스트리밍, 나머지는 버퍼링 방식

<h3 data-toc-skip>정렬 관련 상태 변수</h3>
```sql
FLUSH STATUS;
SHOW STATUS LIKE 'Sort%';
```

<h3 data-toc-skip>GROUP BY 처리</h3>
- 인덱스 스캔을 이용하는 GROUP BY(타이트 인덱스 스캔)
  - 조인의 드라이빙 테이블에 속한 컬럼만 이용해 그루핑할 때 GROUP BY 컬럼으로 이미 인덱스가 있다면 그 인덱스를 차례로 읽어오면서 그루핑 작업을 수행하고 그 결과로 조인을 처리
  - GROUP BY가 인덱스를 사용해서 처리된다 하더라도 그룹 함수 등의 그룹값을 처리해야 해서 임시 테이블이 필요할 때도 있다.
  - GROUP BY가 인덱스를 통해 처리 되는 쿼리는 이미 정렬된 인덱스를 읽는 것이므로 추가적인 정렬이 필요하지 않다.
  - Using index for group-by, Using temporary, Using filesort 표시되지 않음
- 루스 인덱스 스캔을 이용하는 GROUP BY
  - Using index for group-by 표시
  ```sql
  -- loose index scan 사용 못하는 쿼리
  select col1, sum(col2) from tb_test group by col1;
  select col1, col2 from tb_test group by col2, col3;
  select col1, col3 from tb_test group by col1, col2;
  ```
- 임시 테이블 사용하는 GROUP BY
  - Using temporary
  - GROUP BY, ORDER BY 명시하자

<h3 data-toc-skip>DISTINCT 처리</h3>
- 아래의 두 쿼리는 동일하게 내부적으로 처리됨.
```sql
select distinct emp_no from salaries;
select emp_no from salaries group by emp_no;
```
- 레코드를 유니크하게 조회하는 것이지 특정 컬럼만 유니크하게 조회하는 것이 아니다.
- 아래의 쿼리를 괄호가 제거되고 실행된다.
```sql
select distinct (first_name), last_name from salaries;
```
- 집합 함수와 함께 사용된 DISTINCT
  - count(), min(), max()와 같은 집합 함수 내에서 DISTINCT 키워드가 사용 가능
  - 집합 함수가 없는 select 쿼리의 distinct는 조회하는 모든 컬럼이 유니크한 것들만 가져온다.
  - 하지만 집합 함수 내에서 사용된 distinct는 그 집합 함수 인자로 전달된 컬럼값만 유니크한 것들을 가져온다.
  - 인덱스 컬럼에 대해 distinct 처리를 수행할 때는 인덱스를 풀 스캔하거나 레인지 스캔하면서 임시 테이블 없이 최적화된 쿼리를 수행할 수 있다.

<h3 data-toc-skip>내부 임시 테이블 활용</h3>
- 처음에는 메모리에 생성됐다가 테이블의 크기가 커지면 디스크로 옮겨진다.
- 메모리 임시 테이블 & 디스크 임시테이블 > page 314
