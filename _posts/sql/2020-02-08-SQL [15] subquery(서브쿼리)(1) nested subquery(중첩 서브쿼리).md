---
layout: post
title:  "SQL [15] subquery(서브쿼리)(1) 단일행 서브쿼리(nested subquery), 여러행 서브쿼리"
date:   2020-02-08T14:25:52-05:00
author: woodi
categories: SQL
comments: true
---
1. 단일행 서브쿼리
- 중첩 서브쿼리(nested subquery)
2. 여러행 서브쿼리
3. 상호관련 서브쿼리(correlated subquery, 상관서브쿼리)
4. inline view (가상테이블)
5. Exist
6. scalar subquery (스칼라서브쿼리)
- with

# < subquery, 서브쿼리 >
SQL문 안에 있는 select문을 서브쿼리라고 한다. select문장 안의 서브쿼리에는 갯수 제한이 없다.

## 1. nested subquery (중첩 서브쿼리)
- 서브쿼리(inner query) 절이 먼저 수행되고 그 값을 가지고 메인쿼리(outer query) 절이 실행된다.
```
Select
From
where 기준컬럼 비교연산자 (select
							From
							where   );

 --Main query--       --sub query--
```

### (1) 단일행 서브쿼리
- 서브쿼리의 결과로 단일값이 나오는 서브쿼리
- 단일행 비교 연산자 : =, >, <, >=, <=, <>, !, !=, ^=
```
select *
from employees
where salary > (select min(salary)
				from employees
                where last_name = 'King');
-- 'King'이라는 사원이 여러명일 경우 오류가 생기기 때문에 min(salary)를 사용함.
```
```
select *
from employees
where job_id = (select job_id
				from employees
                where employee_id =110);
```
```
select *
from employees
where job_id = (selece job_id
				from employees
                where employee_id = 110)
and salary > (select salary
			  from employees
              where employee_id = 110);
```
having절에서도 서브쿼리를 사용할 수 있다.
```
select avg(salary)
from employees
group by job_id
having avg(salary) = (select min(avg(salary))
					  from employees
                      group by job_id)
```
---
### (2) 여러행 서브쿼리
- 서브쿼리의 결과가 여러 개의 값이 나오는 서브쿼리
- 여러행 비교 연산자 : in, any, all

#### in
```
select *
from employees
where employee_id in (select manager_id
					  from employees);
```
여러행 서브쿼리를 쓰지 않는다면 아래와 같이 모두 나열해 줘야 해서 비효율적이다. 
```
select *
from employees
where employee_id = null
or employee_id = 100
or employee_id = 101
or .....
```
#### any
> any는 or의 범주이다. 따라서, ```wehre salary > any(100,200,300)```은 
> ```where salary > 100 or salary > 200 or salary > 300```과 같다. 즉, 이 경우에는  ```salary > 100```이면 다 포함된다는 뜻.

만약, 결과가 여러개의 값이 나오는 서브쿼리를 단일행 연산자로 비교할 경우 오류가 생긴다.
```
select *
from employees
where salary > (select salary
				from employees
                where job_id='DATA_SCIENTIST');
-- job_id가 data_scientist인 사원은 여러명이기 때문에 단일행 연산자인 > 로 비교하면 오류가 생긴다.
```
대신, 여러행 비교 연산자인 **any**를 사용해서 해결 가능하다.

```
select *
from employees
where salary > any(select salary
				   from employees
                   where job_id = 'DATA_SCIENTIST);
```
이는 아래의 단일행 서브쿼리와 같은 값을 출력한다.
```
select *
from employees
where salary > (select min(salary)
				from employees
                where job_id = 'DATA_SCIENTIST');
```
비교 연산자 부호가 반대로 되는 경우, any는 아래의 단일행 서브쿼리와 같은 값을 출력한다.(연산자 부호와 min, max주의하기)
```
select *
from employees
where salary < any(select salary
				   from employees
                   where job_id = 'DATA_SCIENTIST');
```
```
select *
from employees
where salary < (select max(salary)
				from employees
                where job_id = 'DATA_SCIENTIST');
```

#### all
> all은 and의 개념
> ```where salary > all(100,200,300)``` 은 
> ```where salary > 100 and salary > 200 and salary > 300```과 같은 의미이다. 즉, 이 경우에는 salary > 300이면 다 포함된다는 뜻.

아래의 두 서브쿼리는 같은 값을 출력한다.
```
-- 여러행 서브쿼리
select *
from employees
where salary > all(select salary
					from employees
                    where jon_id = 'DATA_SCIENTIST');
```
```
-- 단일행 서브쿼리
select *
from employees
where salary > (select max(salary)
			    from employees
                where jon_id = 'DATA_SCIENTIST');
```
아래의 두 서브쿼리 역시 같은 값을 출력한다.
```
-- 여러행 서브쿼리
select *
from employees
where salary < all(select salary
					from employees
                    where jon_id = 'DATA_SCIENTIST');
```
```
-- 단일행 서브쿼리
select *
from employees
where salary < (select min(salary)
			    from employees
                where jon_id = 'DATA_SCIENTIST');
```
#### 서브쿼리에 null값이 포함되는 경우
아래의 두 서브쿼리는 같은 값(아무 결과도 없음)을 출력한다.
```
-- 여러행 서브쿼리
select *
from employees
where employee_id not in(select manager_id
						 from employees);
```
```
select *
from employees
where employee_id <> (select manager_id
					  from employees);
```
manager id에는 null값이 있다. 따라서 null값이 아닌 값은 모든 값을 의미하므로 아무 결과도 출력되지 않는 것이다. <br/>
이를 해결하기 위해서는 아래와 같이 null을 제외시키고 수행해야 한다.
```
select *
from employees
where employee_id not in(select manager_id
						 from employees
                         where manager_id is not null);
```
