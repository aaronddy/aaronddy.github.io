---
title:  "[Review] 그로스 인사이트 인턴 SQL 쿼리 테스트 리뷰"
excerpt: "그로스 인사이트 인턴 직무를 지원, SQL로 테스트한 내용을 정리했습니다."

categories:
  - Review
tags:
  - [Review, SQL, PostgreSQL]

toc: true
toc_sticky: false
toc_label: "어떻게 진행할 것이냐"
toc_icon: "cookie-bite"

date: 2021-04-04
last_modified_at: 2021-04-05
---


*********

<br>

🚫 테스트의 모든 내용이 아닌 기록하고 싶은 내용만 담았습니다. <br>
🚫 해당 테스트는 정해진 시간 동안 진행된 테스트 입니다.

<br>

# ✔ SQL 요청 내용

1) 클릭을 가장 많이 받은 상품 100개를 많이 받은 순으로, 클릭을 받은 수와 함께 나타내십시오. <br>
(해당 요청 자체는 간단하지만, 테이블의 특성을 명확히 이해해야 쿼리 작성이 가능했습니다) <br>

2) "downtown" 페이지에서 노출된 상품의 총 클릭수를 나타내십시오. <br>

3) 유저별로 앱에 제일 처음 방문한 시간과 마지막으로 방문한 시간을 구하고, 유저별로 두 시간 차이를 분 단위로 구하십시오. <br>
(해당 요청은 BA 분석을 진행하면서 배운 내용을 활용해서 해결했습니다) <br>

4) 전체 기간 동안 앱에서 클릭이 1회 이상 있는 경우 "1", 0회인 경우 "0"으로 표기하는 칼럼을 나타낸 후, "1"과 "0"의 각 그룹별 유저 수를 구하십시오. <br>

<br>

**********************************

<br>

# ✔ SQL 쿼리 작성

1) 참조 테이블(a_log_table)에는 유저가 클릭을 하는 이벤트가 발생할 때 유저의 로그가 기록됩니다. 따라서 클릭 이벤트가 있으면 해당 테이블로 기록이 되므로 product_id 별 총합이 곧 클릭 수라고 이해했습니다.

```sql
-- 클릭을 가장 많이 받은 상품 100개를 많이 받은 순으로, 클릭을 받은 수와 함께 나타내십시오.
-- product_id로 그룹핑을 해서 카운트를 하면 상품별 클릭수를 추출

SELECT
    product_id
    , COUNT(product_id) AS click_count
FROM a_log_table
GROUP BY product_id
ORDER BY click_count DESC
LIMIT 100
;
```

<br>

2) 유저에게 노출됐을 경우의 유저 로그를 기록하는 테이블(b_log_table)과 클릭 이벤트를 담은 테이블(a_log_table)을 join 해서 쿼리를 작성합니다. 먼저 b_log_table에서 page_id = 'downtown'인 경우를 필터링, 노출된 product_id 별 각각의 총 클릭 수를 추출했습니다.

```sql
SELECT
    exposured_product
    , SUM(CASE WHEN clicked_product is not null then 1 else 0 end)  AS clicked_amount
    -- al 테이블은 클릭 이벤트가 발생했을 경우 로그가 기록되기 때문에 null이 아닌 경우 해당 프로덕트를 클릭한 것으로 이해
FROM (
SELECT
    bl.product_id       AS exposured_product
    , bl.imp_id         AS exposured_imp_id
    , al.product_id     AS clicked_product

FROM b_log_table AS bl
    LEFT JOIN a_log_table AS al
    ON bl.imp_id = al.imp_id

WHERE bl.page_id = 'downtown'       -- bl에서 page_id가 'downtown'인 경우 필터링
) base  

GROUP BY exposured_product          -- 노출된 product로 그룹핑하면 노출된 모든 상품 각각의 총 클릭수 추출          

```

<br>

3) min()과 max() 함수를 이용해 유저의 방문 접속 시간(처음 방문, 마지막 방문)을 뽑았고, `timestamp` 데이터의 속성을 활용하여 시간 차를 구해습니다. (유저의 방문 로그를 기록한 c_log_table 이용)

```sql
-- 해당 요청은 with문을 활용했습니다.

WITH
    user_visit_time AS (

SELECT
    user_id
    , MIN(server_time) AS first_visit_time
    , MAX(server_time) AS last_visit_time

FROM c_log_table
GROUP BY user_id    -- user_id로 그룹핑을 하면 유저별 처음 방문과 마지막 방문 접속 시간이 추출
    )
SELECT
    *
    , FLOOR(EXTRACT EPOCH FROM (
        last_visit_time::TIMESTAMP - first_visit_time::TIMESTAMP
    )::NUMERIC/60)  AS diff_minute

FROM user_visit_time        

```

<br>

4) 풀이 순서 <br> 
   a. 클릭 이벤트 테이블(a_log_table)에서 device_type이 'iphone' 혹은 'android'라면 1을 부여 => user_id별 총합을 계산.  <br>
   b. 모바일 기종의 클릭수가 0이상이면 1, 아니면 0의 값을 부여하는 'clicks' 컬럼 생성. <br>
   c. 해당 컬럼의 값별 총합을 구하면 각 그룹별 유저 수를 추출 가능 <Br>

```sql
WITH
-- a 과정
cnt_app AS (                            
SELECT
	DISTINCT user_id                    -- 중복 제거하고 유저 별 모바일 사용여부 파악
	, SUM(CASE WHEN device_type = 'iphone' THEN 1 
               WHEN device_type = 'android' THEN 1 
               ELSE 0 END) AS app_click

FROM a_log_table 
GROUP BY user_id
    )
-- b 과정
, cnt_click AS (                       
SELECT 
	*,
	CASE WHEN app_click > 0 THEN 1 ELSE 0 END AS clicks
FROM cnt_app
)
-- c 과정
SELECT                                  
	clicks,
	COUNT(clicks)                      -- clicks 값의 그룹 별 카운팅 ==> 그룹 별 유저 수 추출
FROM cnt_click
GROUP BY clicks
;
```

<br>

*****************************

<br>


# ✔ 해당 테스트를 진행하며 느낀 점

- 테이블과 컬럼, 요청 내용의 이해를 담당자 분께 직접 문의하며 요청 내용에 맞게 쿼리를 짜고 원하는 추출 결과를 볼 때면 너무나도 뿌듯하고 재미있었습니다.
- 데이터 분석 업무 못지 않게 가공, 정제, 추출하는 업무도 중요하다고 생각합니다. 요청에 맞게 데이터를 추출하고 리포팅하는 부분을 더욱 많이 훈련해야 겠다고 느꼈습니다.
- Business inteligence 동영상 강의를 통해 배운 것들이 많은데, 해당 과제를 수행하며 비슷한 요청 혹은 쿼리를 작성하며 실무에서 필요한 부분임을 실감했습니다.
- 비즈니스 파트에서 데이터 분석 업무에서는 어쩌면 파이썬이나 R 보다도 SQL과 대시보드 툴이 더 중요하지 않을까 생각하는 계기가 되었고, 어떤 부분에 더 집중해야 할지 정리하는 시간이 갖기도 했습니다. 
- 결론은 SQL 쿼리를 다재다능하게, 상황에 맞게 잘 작성할 수 있도록 더 많이 노력해야겠습니다.

<br>
<br>