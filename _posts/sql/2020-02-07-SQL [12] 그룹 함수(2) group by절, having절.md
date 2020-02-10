---
layout: post
title:  "SQL [12] 그룹 함수(2) group by절, having절"
date:   2020-02-07T14:25:52-05:00
author: woodi
categories: SQL
comments: true
---

## group by절
그룹함수에 포함되지 않는 개별 열이 select절에 나열되어 있을 경우 **무조건 group by**를 이용해 묶어줘야 한다. 그렇지 않으면 오류생김.
```
select department_id, sum(salary)
from employees
group by department_id;
```


**※ 주의할 점**
- group by절에서는 열 별칭을 사용할 수 없다는 것 주의하기.
-  절들의 순서를 지켜줘야 한다! <br/> - 적는 순서 : **select > from > where > group by > order by** <br/> - 내부 처리 순서 : **from > where > group by > 집계값(sum) > order by**
```
select department_id, sum(salary)
from employees
where last_name like '%i%'
group by department_id
order by 1 asc;
```

- - -
## having절
그룹함수의 결과를 제한하기 위한 절이다. <br/> ( where절에서 제한하면 오류가 생긴다.)

```
select department_id, sum(salary)------- 4
from employees	---------------------- 1
where last_name like '%i%' ------------- 2
group by department_id ----------------- 3
having sum(salary) > 6000 -------------- 5
order by 1 asc; ------------------------ 6
-- 내부 처리순서
```
group by절 다음에 habing절을 쓰는 것이 일반적이나 두 절은 순서가 바뀌는 것도 가능하다.

- - -

**※ 문법 정리**
- where절은 행을 제한하는 절
- having절은 그룹함수의 결과를 제한하는 절
- 정렬은 무조건 마지막 절