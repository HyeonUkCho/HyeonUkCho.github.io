---
layout: post  
title: MySQL Index 1  
author: Hyeon Uk Cho  
date: 2023-03-30 18:09:00 +09:00  
categories: [MYSQL]  
tags: [index, b-tree]  
math: true  
mermaid: true  
image: /assets/img/post-header/mysql.png  
path:   
lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA  
alt:
---

<h3 data-toc-skip>한줄 정리</h3>
아주 중요하다...

<h3 data-toc-skip>인덱스란?</h3>
- 책의 마지막에 있는 '찾아보기'로 비유
- SortedList DB 인덱스의 자료구조, ArrayList는 데이터의 자료구조
- DBMS에서 인덱스는 INSERT, UPDATE, DELETE 성능을 희생하고 그 대신 읽기 속도를 높이는 기능  

<h3 data-toc-skip>B(balanced)-Tree 인덱스</h3>
- 가장 일반적으로 사용되는 인덱싱 알고리즘
- 구조  
  ![index-structure](/assets/img/data/index-structure.png)
- 리프 노드는 항상 실제 데이터 레코드를 찾아가기 위한 주솟값을 가지고 있다.
- 인덱스의 키 값은 모두 정렬되어 있지만 데이터 레코드는 정렬되어 있지 않다.
- 항상 insert 되는 순서대로 저장되지 않는다. (삭제도 일어나기 때문에).
  - innodb 데이블에서 레코드는 클러스터 되어 디스크에 저장되므로 기본적으로 프라이머리 키 순서로 정렬되어 저장된다.
- innodb 스토리지 엔진을 사용하는 테이블에서는 프라이머리 키가 ROWID의 역할을 한다.  
  ![index-structure](/assets/img/data/innodb-btree-index-structure.png)
- innodb 테이블에서 인덱스를 통해 레코드를 읽을 때는 위 그림과 같이 데이터 파일을 바로 찾아가지 못한다.  
인덱스에 저장돼 있는 프라이머리 키 값을 이용해 프라이머리 키 인덱스를 한 번 더 검색한 후, 프라이머리 키 인덱스의 리프 페이지에 저장돼 있는레코드를 읽는다.  
즉, innodb 스토리지 엔진에서는 모든 세컨더리 인덱스(프라이머리 키를 제외한 인덱스, 유니크 인덱스 포함) 검색에서 데이터 레코드를 읽기 위해서는 반드시 프라이머리 키를 저장하고 있는 B-Tree를 다시 검색해야한다.  
__(질문) 그럼 innodb에선 데이터를 세컨더리 인덱스로 조회할 때, 인덱스와 프라이머리 키로 되어 있는 B-Tree를 검색하고 그 다음, 프라이머리 키를 주소로 하는 인덱스 루트 노드를 찾아간다?__

<h3 data-toc-skip>B-Tree 인덱스 키 추가 및 삭제</h3>
- 인덱스 키 추가
  - 새로운 키 값이 B-Tree에 저장될 떄 테이블의 스토리지 엔진에 따라 새로운 키 값이 즉시 인덱스에 저장될 수도 있고 그렇지 않을 수도 있다. B-Tree에 저장될 때는
  저잘됭 키 값을 이용해 B-Tree상의 적절한 위치를 검색해야 한다. 저장할 위치가 결정되면 레코드의 키 값과 대상 레코드의 주소 정보를 B-Tree의 리프 노드에 저장한다.
  이러한 작업 떄문에 쓰기 작업에 비용이 많이 드는 것으로 알려졌다.
  - 인덱스 추가로 인해 드는 작업 비용을 간단하게 계산하는 방법
  레코드를 테이블에 추가하는 작업을 1이라고 할때 해당 테이블의 인덱스에 키를 추가하는 작업 비용을 1.5 정도로 예측하는 것이다.
  인덱스가 3개가 있다면 이때 테이블에 인덱스가 하나도 없는 경우는 작업 비용이 1이고, 3개인 경우에는 5.5 (1.5 * 3 + 1)
  
- 인덱스 키 삭제
  - 해당 키 값이 저장된 B-Tree의 리프 노드를 찾아서 그냥 삭제 마크만 하면 작업이 완료된다.
  이렇게 삭제 마킹된 인덱스 키 공간은 계속 그대로 방치하거나 재활용할 수 있다.

- 인덱스 키 변경
  - 인덱스의 키 값은 그 값에 따라 저장될 리프 노드의 위치가 결정되므로 B-Tree의 키 값이 변경되는 경우에는 단순히 인덱스상의 키 값만 변경하는 것은 불가능하다. 
  B-Tree의 키 값 변경 작업은 먼저 키 값을 삭제한 후, 다시 새로운 키 값을 추가하는 형태로 처리된다.

<h3 data-toc-skip>B-Tree 인덱스 키 검색</h3>
- 인덱스를 검색하는 작업은 B-Tree 루트 노드부터 시작해 브랜치 노드를 거쳐 최종 리프 노드까지 이동하면서 비교 작업을 수행하는데, 이 과정을 '트리 탐색'이라고 한다.
- 인덱스 트리 탐색은 SELECT에서만 사용하는 것이 아니라 UPDATE, DELETE를 처리하기 위해 항상 해당 레코드를 먼저 검색해야 할 경우에도 사용된다.
- 100% 일치, left-most part, 부등호 가능 / 변형이 가해지거나 뒷부분만 검색하는 것은 B-Tree 인덱스 사용 불가
- innodb 인덱스는 레코드 잠금이나 넥스트 키락이 검색을 수행한 인덱스를 잠근 후 테이블의 레코드를 잠그는 방식으로 구현돼 있다.

<h3 data-toc-skip>B-Tree 인덱스 키 값의 크기</h3>
- 인덱스는 페이지 단위로 관리되며 루트, 브랜치, 리프 노드를 구분하는 기준이 페이지 단위. innodb_page_size 시스템 변수를 통해 4KB ~ 64KB 사이의 값을 선택할
수 있고 기본값은 16KB.
- 인덱스 1개가 16B, 자식 노드 12B >> 16*1024 / (16+12) = 585 (1개의 페이지당)
- 커진다면? 키 값의 크기가 32B? > 16*1024 / (32+12) = 372 > 500개를 읽어야한다면 위에 케이스는 페이지를 1번 읽으면 되지만 해당 케이스는 최소 2번이상 디스크로부터 읽어야한다.
- 인덱스의 길이가 길어진다 > 크기가 커진다 > 메모리에 캐시해 둘 수 있는 레코드 수는 줄어든다, 메모리 효율 떨어진다.

<h3 data-toc-skip>B-Tree 깊이</h3>
- B-Tree의 깊이가 3인 경우 위에 케이스를 예로 들면,
키 값이 16바이트면 최대 2억개 (585 * 585 * 585)개 정도의 키 값을 담을 수 있지만, 키 값이 32바이트로 늘어나면 5천만(372 * 372 * 372)개로 줄어든다.
- B-Tree의 깊이는 MySQL에서 값을 검색할 때 몇 번이나 랜덤하게 디스크를 읽어야 하는지와 직결되는 문제이다.
- 결론적으로는 인덱스를 가능하면 작게 만드는 것이 좋다는 것

<h3 data-toc-skip>선택도 (기수성)</h3>
- 인덱스에서 선택도(Selectivity) 또는 기수성(Cardinality)은 거의 같은 의미로 사용되며, 모든 인덱스 키 값 가운데 유니크한 값의 수를 의미한다.
- 전체 인덱스가 100개인데 유니크 수가 10이면 기수성은 10이다. 중복된 값이 많아질수록 기수성은 낮아지고 동시에 선택도 또한 떨어진다. 인덱스는 선택도가 높을수록 검색 대상이 줄어들기 때문에 그만큼 빠르게 처리된다.  
  ![index-structure](/assets/img/data/index-caldinal-example.png)

<h3 data-toc-skip>읽어야 하는 레코드 건수</h3>
- 코스트 모델에 따라 다르지만 인덱스를 통해 레코드를 1건 읽는 것이 테이블에서 직접 레코드를 1건 읽는 것보다 4~5배정도 비용이 더 많이 드는 작업인 것으로 예측한다.
- 즉, 읽어야할 레코드의 건수가 전체 테이블 레코드의 20~25%를 넘어서면 인덱스를 이용하지 않고 테이블을 모두 직접 읽어서 필요한 레코드만 가려내는 방식으로 처리하는 것이 효율적이다.

<h3 data-toc-skip>Index Range Scan</h3>
1. 인덱스에서 조건을 만족하는 값이 저장된 위치를 찾는다. 인덱스 탐색  
2. 1번에서 탐색된 위치부터 필요한 만큼 인덱스를 차례대로 쭉 읽는다. 인덱스 스캔. 인덱스 자체 정렬 특성 때문에 정렬되어 레코드를 가져온다.
3. 2번에서 읽어 들인 인덱스 키와 레코드 주소를 이용해 레코드가 저장된 페이지를 가져오고, 최종 레코드를 읽어온다.
4. 리프노드 -> 데이터 레코드 가져오는데 레코드 한 건 한 건 단위로 랜덤 I/O 발생.
5. 쿼리가 필요로 하는 데이터에 따라 3번 과정은 필요하지 않을 수 있는데 이를 커버링 인덱스라고 한다.

<h3 data-toc-skip>Index Full Scan</h3>
1. 인덱스의 처음부터 끝까지 모두 읽는 방식을 인덱스 풀 스캔이라고 한다.

<h3 data-toc-skip>Loose Index Scan</h3>
1. Oracle과 같은 DBMS의 인덱스 스킵 스캔이라고 하는 기능과 작동 방식 비슷.
2. 일반적으로 GROUP BY 또는 집합 함수 가운데 MAX() 또는 MIN() 함수에 대해 최적화를 하는 경우에 사용.
3. 골라서 읽는 ...

<h3 data-toc-skip>Index Skip Scan</h3>
1. index가 (a,b) 컬럼으로 잡혀 있을 때, a 없이도 b 컬럼만으로도 인덱스 검색이 가능하게 해주는 기능.
2. 커버링 인덱스의 경우에만 적용이 된다.
3. WHERE 조건절에 조건이 없는 인덱스의 선행 컬럼의 유니크한 값의 개수가 적어야 적용 가능한 최적화.

<h3 data-toc-skip>다중 컬럼 인덱스</h3>
1. 첫번째, 두번째, 세번째 각각 선행하는 컬럼에 의존적으로 정렬되어 있다.
2. 인덱스 내에서 각 컬럼의 위치가 중요하다.
   
<h3 data-toc-skip>B-Tree 인덱스의 정렬 및 스캔 방향</h3>
1. 인덱스를 생성할 때 ASC, DESC로 설정했다고 해서 계속 생성에 설정된 것으로 읽지 않고 쿼리에 따라 옵티마이저가 실시간으로 만들어내는 실행 계획에 따라 달라진다.
2. 인덱스 생성 시점에 오름차순, 내림차순으로 정렬이 결정되지만 쿼리가 그 인덱스를 사용하는 시점에 인덱스를 읽는 방향에 따라 오름차순 또는 내림차순 정렬 효과를 얻을 수 있다.
3. 오름차순으로 생성된 인덱스를 정순으로 읽으면 출력되는 결과 레코드는 자동으로 오름차순으로 정렬된 결과가 되고, 역순으로 읽으면 그 결과는 내림차순으로 정렬된 상태가 되는 것이다.
```sql
select * from employees where first_name >= 'anneke'
order by first_name asc limit 4;
-- anneke 찾아서 정렬순으로 해당 인덱스를 읽으면서 4개의 레코드 조회
select * from employees
order by first_name desc limit 5;
-- 인덱스 역순으로 읽으면서 처음 다섯 개의 레코드만 가져오면 된다.
```
4. 역순 정렬이 비교적 느리다.
   - 페이지 잠금이 인덱스 정순 스캔에 적합한 구조
   - 페이지 내에서 인덱스 레코드가 단방향으로만 연결된 구조
5. 실제로 레코드들이 물리적으로 저장이 순서대로 배치되지는 않는다. (246 페이지)

<h3 data-toc-skip>B-Tree 인덱스의 가용성과 효율성</h3>
```sql
select * from dept_emp where dept_no = 'd002' and emp_no >= 10114;

-- case A : index (dept_no, emp_no)
-- case B : index (emp_no, dept_no)
```
case A 인덱스는 dept_no=d002 and emp_no > 10144 인 레코드를 찾고, 그 이후에 dept_no가 d002가 아닐 때까지 인덱스를 쭉 읽기만 하면 된다. 꼭 필요한 작업만 진행  
case B 인덱스는 우선 emp_no > 10144 and dept_no = d002 인 레코드를 찾고, 그 이후 모든 레코드에 대해 dept_no가 d002인지 비교하는 과정을 거쳐야 한다.  

가용성은 249 페이지 참고

<h3 data-toc-skip>R-Tree Index</h3>
 - Skip

<h3 data-toc-skip>전문 검색 인덱스 Full Text Index</h3>
1. 어근 분석 알고리즘
   - 불용어 처리(가치 없는 단거 모두 필터링)와 어근 분석(단어의 뿌리인 원형을 찾는 작업)의 과정을 거친다.
   - MeCab 오픈소스 이용, 한글이나 일본어는 형태소 분석이 중요
   - 한글은 거의 사용 X
2. n-gram 알고리즘  
   - 어근 분석 알고리즘의 단점을 보완하기 위한 방법으로 도입.
   - 본문을 잘라서 무조건 몇 글자씩 잘라서 인덱싱하는 방법.
   - 불용어는 기본적으로 영어 단어. 한글 사용할때는 괜찮다.
```sql
-- 불용어 조회
select * from information_schema.innodb_ft_default_stopword;

-- ngram index create
create table tb_test (
                       doc_id int,
                       doc_body text,
                       primary key (doc_id),
                       fulltext key fx_docbody (doc_body) with parser ngram
) engine=innoDB;
-- select 
select * from tb_test where match(doc_body) against ('애플' in boolean mode);
```
