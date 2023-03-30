---
layout: post  
title: MySQL Index-1  
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
