---
title:  "[BA] MySQL로 웹 페이지 조회 수와 랜딩 페이지 파악하기(webpage analytics)"
excerpt: "조회 수가 가장 높은 페이지는 무엇이고, 랜딩 페이지는 무엇인지 어떻게 쿼리를 짤 수 있을까?"

categories:
  - Business Analysis
tags:
  - [BA, MySQL, website, webpage, landingPage, analysis]

toc: true
toc_sticky: false
toc_label: "어떻게 진행할 것이냐"
toc_icon: "cookie-bite"
 
date: 2021-03-19
last_modified_at: 2021-03-24
---

<br>


# ✔ 어떤 목적으로 활용될 수 있을까?

: 웹사이트 컨텐츠 분석(Website Content Analysis)는 어떤 웹페이지에 가장 많은 유저가 방문하는지(조회 수)를 이해해 비즈니스 차원에서 개선시킬 점들에 집중하기 위함.

예를 들어, 페이지 A, 페이지 B, 페이지 C가 있다고 하자. 각 페이지의 세션 유입량이 550, 1750, 625 라고 할 때, 페이지 B는 다른 두 페이지들에 비해 세션 유입량(페이지 방문 유저 수)이 가장 많다. 이를 이용해 페이지 B를 시작 페이지로 설정함으로써 세션 유입량을 유지 혹은 더 증가하도록 시도해 볼 수 있다. 

보통은 다음과 같은 상황에서 이 방법이 활용된다.
 - 가장 많은 세션 유입량 혹은 조회수가 높은 페이지를 찾기 위해
 - 웹사이트에서 랜딩 페이지(landing page)가 무엇인지 알기 위해 ==> 랜딩 페이지가 무엇인지 알아냄으로써 '어떤 내용 혹은 컨텐츠'가 유저로 하여금 해당 웹사이트로 들어오게 하는지 파악하는 것은 비즈니스적 관점에서 중요한 포인트가 될 수 있음.
 - 세션 유입 혹은 조회수가 높은 페이지와 랜딩 페이지를 파악해 비즈니스적 목표 혹은 의도에 맞춰지고 있는지, 그렇지 않다면 어떻게 개선해나갈 수 있을지 등을 고려하기 위해

<br>

** Top website pages 쿼리 예제 - 조회수가 높은 페이지는?

```sql

SELECT
  pageview_url,                                      -- 페이지 url
  COUNT(DISTINCT website_pageview_id) AS pvs         -- 새로운 페이지 유입이 있을 때마다 자동 생성되는 website_pageview_id를 카운팅 해 어떤 페이지의 유입이 제일 많은지를 파악 

FROM website_pageviews
WHERE website_pageview_id <= 1000                    -- website_pageview_id가 1000이하인 경우만 고려(1000개의 경우만 고려)
GROUP BY pageview_url                                -- pageview_url로 그룹핑
ORDER BY pvs DESC                                    -- pvs로 내림차순 

```
💡 결과적으로 `/home` 페이지가 약 52%로 가장 조회수가 높은 것으로 나타났다. 이어서 `/products`는 약 19.5%로 나타났다.


<br>

**************************************

<br>

# ✔ 임시 테이블이 필요할 땐? 
: `CREATE TEMPORARY TABLE`로 임시 테이블을 만들면 된다. 쿼리문에서 사용할 수 있는 하나의 데이터셋 테이블로 일시적인 혹은 임시 테이블이라고 볼 수 있다.

** TEMPORARY TABLE 작성 쿼리 예제

```sql

CREATE TEMPORARY TABLE newTempTableName   -- 'newTempTableName'의 임시 테이블을 만든다고 SQL 서버에 알림 
SELECT * FROM website_sessions            -- website_session 테이블에 있는 모든 컬럼을 선택
WHERE website_session_id <= 100           -- 해당 테이블(website_session)의 컬럼(website_session_id) 값이 100 이하인 경우만 추출해 임시 테이블 생성

```
❗️ 'temporary table'은 현재 이용중인 MySQL 워크벤치 세션에서만 유효함. 한번 워크벤치를 닫고 다시 세션을 시작하면 'temporary table'은 유효하지 않고 없으므로 <u>분석 중에 계속 필요한 테이블이라면 해당 쿼리 혹은 코드를 미리 저장</u>!

<br>

** Top website landing pages 쿼리 예제 - 조회수가 높은 랜딩 페이지는?

```sql

-- 각 website_session_id 별 첫번째로 들어간 페이지(랜딩 페이지) id만 뽑아내 임시 테이블(first_pageview) 생성

CREATE TEMPORARY TABLE first_pageview
SELECT 
  website_session_id,
  MIN(website_pageview_id) AS min_pv_id 

FROM website_pageviews
GROUP BY website_session_id


-- first_pageview 테이블과 website_pageviews 테이블을 join 해서 group by

SELECT
  website_pageviews.pageview_url AS landing_page,
  COUNT(DISTINCT first_pageview.website_session_id) AS session_hitting_this_lander

FROM first_pageviews
  LEFT JOIN
  ON first_pageveiws.min_pv_id = website_pageviews.website_pageview_id

GROUP BY website_pageviews.pageview_url

```
💡  website_pageviews 테이블에서 website_pageview_id <= 1000 이하인 경우의 세션 id 별 랜딩 페이지는 모두 `/home` url 인 것으로 나타났다. 모든 방문자는 해당 url로 처음 유입되었다.


<br>

**************************************

<br>


# ✔ 어떤 요청이 들어올 수 있을까?

A. 가장 조회수가 높은 페이지가 무엇인지 궁금합니다. (2012년 6월 9일 기준)

B. 랜딩 페이지들이 어떻게 되는지, 그리고 각 랜딩 페이지의 크기(volume)는 어떻게 되는지 궁금합니다. (2012년 6월 12일 기준) 

<br>

**************************************

<br>

# ✔ 어떻게 SQL 쿼리를 작성할까?

A. website_pageview_id를 카운팅 하는데, pageview_url을 기준으로 group by

```sql

SELECT
  pageview_url,       
  COUNT(DISTINCT website_pageview_id) AS pvs   -- 새로운 페이지 유입이 있을 때마다 자동 생성되는 website_pageview_id를 카운팅 해 어떤 페이지의 유입이 제일 많은지를 파악 

FROM website_pageviews
WHERE created_at < '2012-06-09'     -- 2012-06-09일을 기준으로 제한
GROUP BY 1
ORDER BY 2 DESC

```
🔔 `COUNT(DISTINCT website_pageview_id)`와 `COUNT(DISTINCT website_session_id)`를 pageview_url로 그룹핑하면 어떻게 다를까? 일단 새로운 페이지 유입이 있을 때마다 자동 생성되는 고유키인 `website_pageview_id`를 pageview_url로 그룹핑해 카운팅을 하게 되면 각 url의 모든 유입을 카운팅하기 때문에 '조회수'를 추출할 수 있게 된다. 하지만 `website_session_id`는 고유한 session id가 어떤 페이지들을 방문했는지를 파악할 때 용이하다. 따라서 하나의 session id가 여러 페이지들을 한 번씩 방문했을 경우 각 페이지의 조회수가 모두 카운팅되지만 만약 한 페이지를 다섯 번 방문했어도 한 번만 카운팅이 되는 것이다. <em>어떤 컬럼값을 기준으로 카운팅하느냐에 따라 다른 결과가 추출될 수 있으므로 항상 주의해야 한다.</em>

💡 결과 해석하기
'2012-06-09' 일자를 기준으로 20876 번의 페이지 조회와 10402 개의 세션이 기록되었다. 이 중 `/home` 페이지가 10398 회로 전체 조회 수의 절반 가량을 차지하면서 가장 많은 조회 수를 기록했고, 거의 모든 세션이 해당 페이지를 방문한 것으로 파악된다.  다음으로 `/products` 페이지가 4238 값으로 두 번째를 기록했으나 `/home` 페이지와 많은 차이를 보이고 있다. 그럼에도 세션의 절반에 미치지 못하는 수가 `/products` 페이지를 방문하고 있다. 

<br> 
<br> 

B. 각 세션의 랜딩 페이지를 추출한 임시 테이블 생성 후, 랜딩 페이지 조회 수 추출하기

```sql

-- step 1. 각 세션의 랜딩 페이지를 확인할 수 있는 id or timestamp 추출한 임시 테이블 생성하기

CREATE TEMPORARY TABLE session_landing_id
SELECT
  website_session_id,
  MIN(website_pageview_id) AS min_pv_id

FROM website_pageviews
WHERE created_at < '2012-06-12'
GROUP BY 1


-- step 2. session_landing_time 테이블을 조인해서 각 랜딩 페이지의 조회 수 추출하기

SELECT 
  website_pageviews.pageview_url AS landing_page_url,
  COUNT(DISTINCT session_landing_id.website_session_id) AS session_hitting_landing_page

FROM session_landing_id
  LEFT JOIN website_pageviews
  ON website_pageviews.website_pageview_id = session_landing_id.min_pv_id

GROUP BY 1
-- session_landing_id 테이블에서 '2012-06-12'로 제한을 뒀기 때문에 해당 쿼리에서는 그 조건을 넣을 필요가 없다.

```
🔔 각 테이블의 'created_at' 컬럼 역시 각 로우의 고유성을 갖고 있다고 생각했습니다. 그래서 처음에 임시 테이블을 만들 때 `website_pageview_id`가 아니라 `created_at`을 MIN()에 넣어서 세션 id별 가장 빠른 생성 시간을 도출했습니다. 하지만 website_pageviews 테이블과 조인하면서 더 많은 랜딩 페이지와 조회 수가 출력 되었습니다. 생각해 보면 아주 드물 수 있지만 같은 시간에 웹페이지 유입이 있을 수 있기 때문인 것 같습니다. 그래서 이번 경험으로 timestamp, 즉 `created_at`은 고유성을 띄기에는 어렵다는 것을 알게 되었습니다. 결국 `website_pageview_id`를 이용해 임시 테이블을 만들었습니다.

💡 결과 해석하기
'2012-06-12' 일자를 기준으로 10714 세션, 즉 모든 트래픽이  `/home` 페이지를 랜딩 페이지인 것으로 나왔습니다. 이 결과를 통해 해당 페이지가 이용자에게 어떤 경험을 주고 있는지(user experience)와 페이지가 해당 사이트를 대표하고 있다고 볼 수 있는지 등을 고려하여 해당 페이지를 더욱 발전시킬 필요가 있습니다. 

<br>

**************************************

<br>

# ✔ 무엇을 남겼나

- 이전 과제들 보다 더 복잡해서 쿼리를 짜는 게 생각보다 쉽지 않았습니다. '그러겠지' 라고 생각했던 게 사실 그렇지 않았습니다. 그래서 여러 번 쿼리를 실행해 보고 수정하며 제가 생각한 게 틀렸을 때 저의 논리에 따라가 보며 어떤 오류가 있었는지를 파악해 나갔습니다. 쉽다고 여겼지만 한 번 더 확인하고 천천히 진행해 나가야 실수를 줄일 수 있다는 것을 계속 느끼게 해준 과제였습니다. 
- SQL은 참 직관적이고 테이블 간의 연결고리를 생각하며 쿼리를 짜나가는 게 재밌으면서도 한 번 논리가 잘못되면 결과는 너무나도 크게 달라진다는 걸, 하지만 어떤 오류도 주지 않는다는 걸 마주할 때면 무섭기도 합니다. 그래서 더더욱 겸손한 자세가 필요한 것 같습니다.🥲