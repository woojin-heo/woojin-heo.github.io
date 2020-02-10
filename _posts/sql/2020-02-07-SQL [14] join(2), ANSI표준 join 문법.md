---
layout: post
title:  "SQL [14] join(2), ANSI표준 join 문법"
date:   2020-02-07T14:25:52-05:00
author: woodi
categories: SQL
comments: true
---
# < join >
## ANSI 표준 join 문법
## 1. natural join
join조건 수렁가 없고 알아서 동일한 이름의 컬럼으로 join조건 술어를 만들어 낸다. 
- 동일한 이름의 컬럼이 하나만 있을 경우 문제가 없지만, 여러개일 경우에 그 동일한 이름의 모든 컬럼을 다 join하기 때문에 주의하자.
- 양쪽 테이블에 동일한 이름의 컬럼이 하나만 있을 때 사용하는 것이 좋다.
- 컬럼 이름은 동일한데 데이터 타입이 다르면 오류

```
select department_name, city
from departments natural join locations;
```

- - -

## 2. join using절
using절을 이용해 join을 수행할 기준 컬럼을 명시해 준다.
- using절에 사용된 컬럼은 양쪽 테이블의 기준 컬럼이기 때문에 어느 테이블이라고 접두어로 지정해서는 안된다.(그 컬럼에 테이블 별칭을 붙이면 오류)

```
selece e.employee_id, department_id, d.department_name
from employees e join departments d
using(department_id) -- join 수행 할 기준컬럼
where department_id in(10,20,30);
```
여러개의 테이블을 join하는 경우, **join > using > join > using**의 순서로 작성해야 한다.
```
select e.employee_id, department_id, d.department_name, l.city
from employees e join departments d
using(department_id)
join locations l
using(locaion_id)
where department_id in10,20,30
```
---

## 3. join on절
ANSI표준 join문법중 가장 편한 방법이다. join절에 join할 테이블을 명시하고 on절에 기준 컬럼들을 명시한다.

```
select e.employee_id, d.department_name
from employees e join departments d
on e.department_id = d.department)id
where e.department_id in(20,30);
```
여러개의 테이블을 join하는 경우, **join > on >join > on**의 순서로 작성해야 한다.
```
select e.employee_id, d.department_name, l.city
from employees e join departments d
on e.department_id = d.department_id
join locaions l
on d.location_id = l.locaion_id
where e.department_id in(20,30);
```
---

## 4. outer join
join 키 값이 일치되지 않는 데이터도 합치기 위한 join 방법이다.
#### (1) left outer join
**오라클 전용** 문법에서 left outer join <br/>: key값이 일치되지 않아도) + 부호가 붙지 않은 쪽의 데이터 모두 출력
```
select e.employee_id, d.department_name
from employoees e, departments d
where e.department_id = d.department_id(+);
```
**ANSI 표준** 문법에서 left outer join <br/>: (key값이 일치되지 않아도) 왼쪽에 있는 데이터 모두 출력
```
select e.employee_id, d.department_name
from employoees e left outer join departments d
on e.department_id = d.department_id;
```
<br/>
#### (2) right outer join
**오라클 전용** 문법에서 right outer join <br/>: key값이 일치되지 않아도) + 부호가 붙지 않은 쪽의 데이터 모두 출력
```
select e.employee_id, d.department_name
from employoees e, departments d
where e.department_id(+) = d.department_id;
```
**ANSI 표준** 문법에서 right outer join <br/>: (key값이 일치되지 않아도) 오른쪽에 있는 데이터 모두 출력
```
select e.employee_id, d.department_name
from employoees e right outer join departments d
on e.department_id = d.department_id;
```
<br/>
#### (3) full outer join
**오라클 전용** 문법에서 full outer join <br/>: key값이 일치되지 않아도) + 부호가 붙지 않은 쪽의 데이터 모두 출력
```
select e.employee_id, d.department_name
from employoees e, departments d
where e.department_id = d.department_id(+)
union
select e.employee_id, d.department_name
from employoees e, departments d
where e.department_id(+) = d.department_id;
```
하지만 이렇게 하는 경우, 동일한 테이블을 두번 반복하게 한다. 따라서 중복 검사를 위해 sort 또는 hash알고리즘이 돌아가고, 그러면 메모리 사용량과 CPU사용량이 커지는 성능상의 문제가 생긴다. **full outer join의 경우 ANSI 표준 문법을 사용하는 것이 더 좋다.**

**ANSI 표준** 문법에서 full outer join <br/>: (key값이 일치되지 않아도) 오른쪽에 있는 데이터 모두 출력
```
select e.employee_id, d.department_name
from employoees e full outer join departments d
on e.department_id = d.department_id;
```