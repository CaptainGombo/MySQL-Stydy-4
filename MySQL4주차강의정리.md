# MySQL-Stydy-4

*Subquery : 쿼리 안의 쿼리 (더 편하고 간단하게 원하는 데이터를 얻기 위해 사용되는 파워풀한 기능)

1)Where에 들어가는 Subquery
- Where은 조건문이죠? Subquery의 결과를 조건에 활용하는 방식으로 유용하게 사용합니다.
where 필드명 in (subquery) 이런 방식
*쿼리가 실행되는 순서
(1) from 실행: users 데이터를 가져와줌
(2) Subquery 실행: 해당되는 user_id의 명단을 뽑아줌
(3) where .. in 절에서 subquery의 결과에 해당되는 'user_id의 명단' 조건으로 필터링 해줌
(4) 조건에 맞는 결과 출력

2)Select 에 들어가는 Subquery
- Select는 결과를 출력해주는 부분이죠? 기존 테이블에 함께 보고싶은 통계 데이터를 손쉽게 붙이는 것에 사용합니다.
select 필드명, 필드명, (subquery) from .. 이렇게 씀
*쿼리가 실행되는 순서
(1) 밖의 select * from 문에서 데이터를 한줄한줄 출력하는 과정에서
(2) select 안의 subquery가 매 데이터 한줄마다 실행되는데
(3) 그 데이터 한 줄의 user_id를 갖는 데이터의 평균 좋아요 값을 subquery에서 계산해서
(4) 함께 출력해준다!

3)From 에 들어가는 Subquery (가장 많이 사용되는 유형!)
- From은 언제 사용하면 좋을까요? 내가 만든 Select와 이미 있는 테이블을 Join하고 싶을 때 사용하면 딱이겠죠!
ex)select pu.user_id, a.avg_like, pu.point from point_users pu
inner join (
	select user_id, round(avg(likes),1) as avg_like from checkins
	group by user_id
) a on pu.user_id = a.user_id
*쿼리가 실행되는 순서
(1) 먼저 서브쿼리의 select가 실행되고,
(2) 이것을 테이블처럼 여기고 밖의 select가 실행!

*자주 쓰이는 Subquery 유형 

ex) kakaopay로 결제한 유저들의 정보 보기

1)users 와 orders 의 inner join으로 구하기
select u.user_id, u.name, u.email from users u
inner join orders o on u.user_id = o.user_id
where o.payment_method = 'kakaopay'

2)
- 우선 kakaopay로 결제한 user_id를 모두 구해보기 → K 라고 하기
select user_id from orders
where payment_method = 'kakaopay'

- 그 후에, user_id가 K 에 있는 유저들만 골라보기
->이것이 서브쿼리
select u.user_id, u.name, u.email from users u
where u.user_id in (
	select user_id from orders
	where payment_method = 'kakaopay'
)

Quiz1. 전체 유저의 포인트의 평균보다 큰 유저들의 데이터 추출하기
(포인트가 평균보다 많은 사람들의 데이터를 추출해보자!)
답: select * from point_users pu 
where pu.point > (
select avg(pu2.point) from point_users pu2
)

Quiz2. 이씨 성을 가진 유저의 포인트의 평균보다 큰 유저들의 데이터 추출하기
(이씨 성을 가진 유저들의 평균 포인트보다 더 많은 포인트를 가지고 있는 데이터를 추출해보자!)
답:select * from point_users pu 
	where pu.point > 	(
	select avg(pu2.point) from point_users pu2
	inner join users u 
	on pu2.user_id = u.user_id 
	where u.name = "이**"
	)

*With 절로 깔끔하게 쿼리문 정리하기
ex)
select c.title,
       a.cnt_checkins,
       b.cnt_total,
       (a.cnt_checkins/b.cnt_total) as ratio
from
(
	select course_id, count(distinct(user_id)) as cnt_checkins from checkins
	group by course_id
) a
inner join
(
	select course_id, count(*) as cnt_total from orders
	group by course_id 
) b on a.course_id = b.course_id
inner join courses c on a.course_id = c.course_id

이런 쿼리를 좀 더 보기 간편하게 정리하는 방법이 with절 이용

-> with table1 as (
	select course_id, count(distinct(user_id)) as cnt_checkins from checkins
	group by course_id
), table2 as (
	select course_id, count(*) as cnt_total from orders
	group by course_id
)
select c.title,
       a.cnt_checkins,
       b.cnt_total,
       (a.cnt_checkins/b.cnt_total) as ratio
from table1 a inner join table2 b 
on a.course_id = b.course_id
inner join courses c on a.course_id = c.course_id

*실전에서 유용한 SQL 문법
1.문자열

-SUBSTRING_INDEX : 문자열 쪼개기
ex)
select user_id, email, SUBSTRING_INDEX(email, '@', 1) from users
:@를 기준으로 텍스트를 쪼개고, 그 중 첫 번째 조각을 가져오라는 뜻!

ex)select user_id, email, SUBSTRING_INDEX(email, '@', -1) from users
:@를 기준으로 텍스트를 쪼개고, 그 중 마지막 조각을 가져오라는 뜻!

-SUBSTRING: 문자열 일부만 출력하기
ex)
select order_no, created_at, substring(created_at,1,10) as date from orders
:SUBSTRING(문자열, 출력을 하고싶은 첫 글자의 위치, 몇개의 글자를 출력하고 싶은지)

ex)
select substring(created_at,1,10) as date, count(*) as cnt_date from orders
group by date
:일별로 몇 개씩 주문이 일어났는지 살펴보기

2.CASE: 경우에 따라 원하는 값을 새 필드에 출력

ex)포인트 보유액에 따라 다르게 표시해주기
select pu.point_user_id, pu.point,
case 
when pu.point > 10000 then '잘 하고 있어요!'
else '조금 더 달려주세요!'
END as '구분'
from point_users pu
