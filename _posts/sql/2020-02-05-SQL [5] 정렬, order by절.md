---
layout: post
title:  "SQL [5] 정렬, order by절"
date:   2020-02-03T14:25:52-05:00
author: woodi
categories: SQL
comments: true
---
### 정렬
- order by 절을 이용해서 정렬을 수행한다. 
- order by 절은 무조건 마지막에 기술해야 한다.(select문장의 결과를 먼저 만들어 두고 마지막에 정렬하는 것)
- asc(ascending)이 기본값, 오름차순
desc(descending)은 내림차순
- 문자, 숫자, 날짜 타입 모두 가능

```sql
-- salary를 기준으로 내림차순 정렬

select employee_id, last_name, salary
from employees
where department_id=50
order by salary desc;
```
order by절에서는 select절에 나열된 표현식 그대로 써줘야 한다.
```sql
select employee_id, last_name, salary*12
from employees
where department_id=50
order by salary*12 desc;
```
만약 큰따옴표로 묶은 열별칭을 사용할 땐  order by절에서도 큰따옴표를 써줘야 한다.
```sql
select employee_id, last_name, salary*12 as "ann_sal"
from employees
where department_id=50
order by "ann_sal" desc;
```
열 별칭 대신 위치표기법을 이용할 수 있다.
```sql
-- 1:employee_id, 2:last_name, 3: ann_sal

select employee_id, last_name, salary*12 as "ann_sal"
from employees
where department_id=50
order by 3 desc;
```
order by 기준은 여러개로 사용 가능하다.
```sql
select department_id, salary, last_name
from employees
order by 1 asc, 2 desc;
```