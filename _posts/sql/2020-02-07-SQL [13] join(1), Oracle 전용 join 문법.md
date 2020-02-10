---
layout: post
title:  "SQL [13] join(1), Oracle 전용 join 문법"
date:   2020-02-07T14:25:52-05:00
author: woodi
categories: SQL
comments: true
---
# < join >
**join이란, 두 개 이상의 서로 다른 테이블에서 데이터를 가져오는 방법이다.**<br/>
중복되는 테이블을 제거하기 위해 정규화 작업을 하다보니 테이블이 분리되어 저장되어 있다. 그러나 각 요소들을 같이 놓고 비교해야 하는 일이 있어서 join을 사용하게 된다. 

join의 종류는 다음과 같다. 

## 1. cartesian product
첫 번째 테이블의 모든 행이 두 번째 테이블의 모든 행에 조인 되는 경우이다. 보통 아래와 같은 경우 발생한다.
- 조인 조건이 생략된 경우
- 조인 조건을 잘못 만든 경우
```
select employee_id, department_name
from employees, departments;
-- employee_id와 department_name이 서로 다른 테이블에 있기 때문에 from절에서 각 테이블을 명시해주지 않으면 cartesia product가 발생한다. 
```

<br/>**cartesian product가 발생했는지 확인하는 방법**

- 건수검증 : 결과 값의 최대 갯수는 employees테이블의 row개수를 넘어갈 수 없다.
- employees 테이블을 기준으로 보면 M족 집합, departments 테이블은 1족 집합이다. 따라서 M족의 개수보다 출력값의 개수가 크게 나오면 cartesian product가 발생한 것.
---


## 2. equi join
등가 조인, inner join, simple join 이라고도 한다.
**key값이 일치되는 데이터끼리 join한다.**

예시) 두 테이블을 join 할 때, dept_id를 key로 합친다. 
![equi join](https://user-images.githubusercontent.com/55940348/74122429-869e5d00-4c0e-11ea-9e68-dac27ce4bd5e.PNG)

```
select employee_id, department_name
from employees, departments
where department_id = department_id; --조인조건 술어
-- 오류 "column ambiguously defined"
```
```
select employee_id, department_name
from employees e, departments d
where e.department_id = d.department_id;
```
equi join 을 하면 null값은 안 나옴.(조인 키 값이 일치되는 데이터만 뽑아내는 것이기 때문에)
<br/>
만약, 겹치는 컬럼이 없다면, 다른 테이블을 매개체로 이용해서 join할 수 있다.
```
select e.employee_id, l.city
from employees e, departments d, locations l
where e.department_id = d.department_id
and d.location_id = l.location_id;
-- employee테이블과 location테이블을 연결하기 위해 department테이블을 징검다리로 사용
```
```
select e.employee_id, c.country_name
from employees e, locations l, departments d, countries c
where e.department_id = d.department_id
and d.location_id = l.location_id
and l.country_id = c.country_id;
-- employee테이블과 country테이블을 연결하기 위해 department테이블과 location테이블을 징검다리로 사용.
```
join조건 술어를 먼저 쓸지, 비join조건 술어를 먼저 쓸지는 데이터에 따라 다르다.
```
select emlast_name, d.department_id, d.department_name
from employees e, departments d
where e.department_id = d.department_id -- 조인 조건 술어
and e.last_name like 'K%'; -- 비조인 조건 술어
```
예) join조건을 이용해서 20번 부서 사원을 찾을 때
```
--방법 1
select e.last_name, d.department_id, d.department_name
from employees e, departments d
where d.department_id = 20
and e.department_id = 20;
```
```
--방법 2
select e.last_name, d.department_id, d.department_name
from employees e, departments d
where e.department_id = d.department_id
and e.department_id = 20;
```
---
## 3. outer join
equi join에서 key값이 일치되는 사원만 뽑아내는 것의 문제점을 해결 한 것이 outer join이다. 즉, **key값이 일치되지 않는 데이터도 뽑아낼 수 있다**는 것이다.
<br/>
key값이 일치되는 데이터를 다 뽑아내고, **+ 부호가 없는 쪽의 데이터는 key값이 일치되지 않아도 다 출력**하라는 뜻
```
select e.last_name, d.department_id, d.department_name
from employees e, departments d
where e.department_id = d.department_id(+);
-- employee는 있는데 department_id가 없는 부서는 누락됨
```
```
select e.last_name, de.department_id, d.department_name
from employees e, departments d
where e.department_id(+) = d.department_id;
-- 소속 부서가 없는 사원은 누락
```
오라클 전용에서는 양쪽 다 + 부호를 붙일 수 없다.

- - -

## 4. self join
다른 테이블을 참조하는 것이 아니라, 자기 자신의 테이블을 참조해야 하는 상황에서는 self join을 사용한다. 

예)아래의 상태에서 manager_name도 파악하고 싶으면? 
```
select employee_id, last_name, manager_id
from employees;
```
manager_id값은 employee_id에서 가져온 것이기 때문에 manager 테이블이 따로 없는 상태이다. --> manager_id값을 employee_id값과 연결해서 name을 찾을 수 있다!
```
selece w.employee_id, w.last_name, m.employee_id, m.manager_id
from employees w, employees m
wehre w.manager_id = m.employee_id;
```
- 여기서 M족 집합은 manager_id값이다. employee_id는 중복 없는 1족 집합이다.
- 누락된 데이터가 없는지 확인하고 싶다면 M족 집합의 개수와 출력값이 일치하는지 확인하기 --> 1개 부족함 (1개 누락)
- 누락된 사원도 출력하고 싶다면 null값이 있는 쪽 반대쪽에 + 부호를 붙여서 해결하기 (self join과 outer join을 활용해서 해결!)

```
select w.employee_id, w.last_name, m.employee_id, m.manager_id
from employees w, employees m
where w.manager_id = m.employee_id(+)
```
---
## 5.non-equi join
= (같다) 이외에 다른 연산자를 이용해 조인을 수행할 때 사용한다.(크다, 작다 등 범위적으로 찾아야 할 때)
```
select e.employee_id, e.salary, j.grade_level
from employees e, job_grades j
where e.salary between j.lowest_sal and j.highest_sal;
```
