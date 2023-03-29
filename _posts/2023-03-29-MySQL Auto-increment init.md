---
layout: post  
title: MySQL Auto-increment init  
author: Hyeon Uk Cho  
date: 2023-03-29 18:09:00 +09:00  
categories: [MYSQL]  
tags: [auto-increment, reset, initialize]  
math: true  
mermaid: true  
image: /assets/img/pos-header/mysql.png  
path:   
lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA  
alt:
---


<h3 data-toc-skip>한줄 정리</h3>
auto-increment 초기화의 경우, 기존 데이터가 초기화하는 값보다 높은 값이 있다면 적용이 되지 않는다.

<h3 data-toc-skip>발생</h3>
1. 누군가 테스트를 위해 shop_seq를 999000대로 변경을 함.  
2. shop_seq가 999001, 999002 채번이 되기 시작함.
3. 아래와 같이 초기화하였으나 적용이 되지 않음
```sql
ALTER TABLE shop_table AUTO_INCREMENT = 1050;
```
<h3 data-toc-skip>원인</h3>
기존 데이터가 초기화하는 값보다 높은 값이 있다면 적용이 되지 않는다.

<h3 data-toc-skip>분석</h3>
N/A

<h3 data-toc-skip>해결</h3>
```sql
select * from shop_table order by shop_seq desc;
update shop_table set shop_seq = shop_seq - 999000  where shop_seq > 999000 ;
ALTER TABLE shop_table AUTO_INCREMENT = 1050;
```

