---
layout: post  
title: MySQL Index  
author: Hyeon Uk Cho  
date: 2023-03-30 18:09:00 +09:00  
categories: [MYSQL]  
tags: [index, b-tree]  
math: true  
mermaid: true  
image: /assets/img/pos-header/mysql.png  
path:   
lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA  
alt:
---


<h3 data-toc-skip>한줄 정리</h3>
auto-increment 초기화의 경우, 기존 데이터가 초기화하는 값보다 높은 값이 있다면 적용이 되지 않는다.

<h3 data-toc-skip>인덱스란?</h3>
- 책의 마지막에 있는 '찾아보기'로 비유
- SortedList DB 인덱스의 자료구조, ArrayList는 데이터의 자료구조
- DBMS에서 인덱스는 INSERT, UPDATE, DELETE 성능을 희생하고 그 대신 읽기 속도를 높이는 기능  

<h3 data-toc-skip>B(balanced)-Tree 인덱스</h3>
- 가장 일반적으로 사용되는 인덱싱 알고리즘
- 구조
  - ![index-structure](/assets/img/data/index-structure.png)
- 리프 노드는 항상 실제 데이터 레코드를 찾아가기 위한 주솟값을 가지고 있다.
- 인덱스의 키 값은 모두 정렬되어 있지만 데이터 레코드는 정렬되어 있지 않다.
- 항상 insert 되는 순서대로 저장되지 않는다. (삭제도 일어나기 때문에).
  - innodb 데이블에서 레코드는 클러스터 되어 디스크에 저장되므로 기본적으로 프라이머리 키 순서로 정렬되어 저장된다.
- innodb 스토리지 엔진을 사용하는 테이블에서는 프라이머리 키가 ROWID의 역할을 한다.
  - ![index-structure](/assets/img/data/innodb-btree-index-structure.png)

