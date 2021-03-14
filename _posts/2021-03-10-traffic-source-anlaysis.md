---
title:  "[BA] MySQL로 웹 트래픽 세션(Web Traffic Session) 분석하기"
excerpt: ""

categories:
  - Business Analysis
tags:
  - [BA, MySQL, traffic, session, UTM, analysis]

toc: false
toc_sticky: false
toc_label: "어떻게 진행할 것이냐"
toc_icon: "cookie-bite"
 
date: 2021-03-10
last_modified_at: 2021-03-14
---
<br>

# ✔ 유료 캠페인 트래픽(Paid Campaign Traffic)은 뭐죠? 

: 먼저 '트래픽(traffic)'은 뭐라고 설명해야 할까? e-Commerce, 혹은 온라인 사이트에서 서비스를 제공해 대가를 지불하는 형태라면 뭐든,에서 그 해당 웹 사이트를 방문하는 고객, 혹은 고객의 흐름을 'traffic'이라고 볼 수 있다.
<br>
그리고 서비스를 운영하기 위해서는 이 'traffic'에 신경을 쓸 수 밖에 없는데, 트래픽을 모으는 방식에는 다음과 같은 것들이 있다.
 - Direct traffic(직접유입 트래픽): 이용자가 직접 URL을 입력해서 접속하거나 알수 없는 채널로 들어올 경우
 - Referrel traffic(간접유입 트래픽): 다른 웹사이트에서 해당 URL을 클릭해서 들어올 경우
 - Organic Search traffic(자연 검색 트래픽): 검색 엔진(구글, 네이버, 야후)에 키워드를 입력해서 해당 웹사이트를 발견하거나 클릭해서 들어올 경우
 - Campaign traffic(캠페인 트래픽): 특정한 목적(마케팅) 달성을 위해 특정 메시지를 지속적으로 전달, 특정 트래킹 파라미터(tracking parameters)를 클릭해 해당 웹사이트에 들어오는 경우 

<br>
그리고 campaign traffic에 paid가 붙으면 '유료 캠페인 트래픽'이 되는 것이고, 말 그대로 트래픽을 돈을 주고 사 오는 것이다. '유료 캠페인 트래픽'을 통해 얼마나 돈을 쓰고, 그에 견주어 얼마나 '판매'로 이어지고 있는지(전환율)를 관찰하고 측정을 하는데, 그것을 위해 'tracking (UTM) parameter'를 이용한다. 웹 사이트 URL 뒤에 이어지는 특정 트래픽 소스와 캠페인 파라미터를 통해 웹사이트의 트래픽 활동을 분석하고 그에 맞춰 마케팅 혹은 판매 전략을 만드는 것이다.

<center>**utm parameters 예시**</center>

![image](https://polkadotdata.com/wp-content/uploads/2019/07/pdd-utm-parameters-example-1-1024x361.png){: .align-center}{: width="70%" height="70%"}

<center>(출처: https://polkadotdata.com/google-analytics/utm-tracking-links-for-enhanced-analytics/)</center>  

<br>

*******************************

<br>

# ✔ '트래픽 소스(Traffic Source)'로 무엇을, 어떻게 분석할까? 
  트래픽 소스를 이용한 분석은 '이용자들이 어떻게 유입되고, 어떠한 채널의 세션에서 판매나 수익으로 전환되었는지(전환율)' 등을 파악하는 것이다. 

### 1. 주요 table 소개
1) website_sessions 테이블

- website_session_id: primary key, 웹 사이트 세션에 부여된 id
- created_at: timestamp, 웹 사이트 세션이 발생될 때의 시간
- user_id: 웹 사이트 쿠키로부터 받는 유저 고유 id(유저 로그인 아이디 X)
- is_repeat_session: 유저가 해당 웹사이트에 방문한 적 있는지의 여부(0: 없음, 1: 있음)
- utm_source: 트랙킹 파라미터
- utm_campaign: 트랙킹 파라미터
- utm_content: 트랙킹 파라미터
- device_type: PC / mobile 유저가 접속한 장비 
- http_referer: 트래픽 유입 URL(마케팅 캠페인 X)

2) website_pageviews 테이블

- website_pageview_id: primary key, 웹 사이트 페이지뷰에 부여된 id
- created_at: timestamp, 해당 페이지뷰에 접속할 때의 시간
- website_session_id: foreign key, website_sessions 테이블의 주요키이자 website_pageviews 테이블의 외래키
- pageview_url: 접속한 페이지뷰 URL

3) orders 테이블(모든 이커머스 회사에서 중요한 테이블)

- order_id: primary key, 주문에 부여된 id
- created_at: timestamp, 주문이 발생했을 때의 시간
- website_session_id: foreign key, website_sessions 테이블의 주요키이자 orders 테이블의 외래키
- user_id: 유저 고유 id
- primary_product_id: 유저가 카트에 처음 넣은 상품 id 
- items_purchased: 유저가 구매한 상품 개수
- price_usd: 판매가(USD)
- cogs_usd: 매출원가(USD)


### 2. 분석 objects
1). 데이터베이스(MySQL)에 저장된 `utm parameter`를 이용해 유료 웹사이트 세션(paid website session)을 파악.<br>
2). sessions 테이블과 orders 테이블을 'join'해 유료 캠페인으로 얼마나 수익을 만들어 냈는지(전환율이 어떠한지)를 파악.

* 분석 쿼리 예제
* utm_content별 session volume & order volume, conversion rates 파악해보기

```sql

SELECT 
  website_sessions.utm_content,
  COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
  COUNT(DISTINCT orders.order_id) AS orders
  COUNT(DISTINCT orders.order_id) / COUNT(DISTINCT website_sessions.website_session_id) AS session_to_order_conv_rt

FROM website_sessions
    LEFT JOIN orders
    ON orders.website_session_id = website_sessions.website_session_id
  
GROUP BY 1
ORDER BY 4 DESC;

```
<br>

*****************************
<br>

# ✔ 어떤 요청이 들어올 수 있을까?

1. 웹 사이트 세션이 어떤 경로로 유입되고 있는지 알고 싶습니다. UTM source, UTM campaign, referring domain 을 가지고 세션 양(volume)이 어떠한지 파악해 주세요. (2012년 4월 12일 기준)

2. `gsearch` - `nonbrand`의 캠페인 세션 유입이 구매로 이어지는 전환율(CVR)이 어떠한지 궁금합니다. 최소 4% 이상이면 유료 캠페인 비용을 늘리고, 그렇지 않으면 감축할 예정입니다.

<br>

******************************

<br>

# ✔ 어떻게 SQL 쿼리를 작성할까?

### 1. utm source, utm campaign, referring domain => Group By & website_session_id => Count 

```sql

SELECT
  utm_source,
  utm_campaign,
  http_remferer,
  COUNT(DISTINCT website_session_id) AS sessions

FROM website_sessions
-- created_at이 2012년 4월 12일 이전인 경우만 고려
WHERE created_at < '2012-04-12'
GROUP BY 1, 2, 3
ORDER BY 4 DESC; 

```
🔔 처음에 utm_source와 utm_campaign만 group by를 진행했는데 계속 에러가 났습니다. http_referer를 함께 그룹핑하지 않으면 group by 자체가 진행되지 않기 때문에 문법 에러가 난 것이었습니다. SQL은 진행 과정을 보면서 결과를 확인하는 게 아니라 정확한 쿼리를 작성해야만 결과를 확인할 수 있다는 것을 다시 한 번 배웠습니다.
<br>

💡 결과 해석하기
  - 'utm_source=gsearch & utm_campaign=nonbrand'에서 총 3613개로 가장 많은 세션이 있다. 따라서 해당 캠페인에 대해 더 자세하게 살펴볼 필요가 있다.
  - 그 다음이 utm_source와 utm_campaign 모두 null 값이 경우인데, 이보다 더 적은 수의 utm 파라미터는 어떻게 운영을 해 나갈 것인지에 대한 전략 혹은 개선이 필요해 보인다. 
  - http_referer를 보면 'gsearch'의 경우가 'bsearch'의 경우보다 훨씬 더 많은 세션 수를 갖고 있다. 'bsearch'를 계속해서 운영할 필요에 대한 의문이 제기된다. 다른 채널을 고려하는 게 낫지 않을까?

<br>

### 2. website_sessions, orders => 테이블 조인 & website_session_id, order_id => Count 

```sql

SELECT 
  COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
  COUNT(DISTINCT orders.order_id) AS orders,
  COUNT(DISTINCT orders.order_id) / COUNT(DISTINCT website_sessions.website_session_id) AS sessions_to_order_conversion_rt

FROM website_sessions
  -- session 유입량에서 구입 전환율을 파악하는 것이므로 left join 진행
  LEFT JOIN orders
  ON website_sessions.website_session_id = orders.website_session_id

WHERE website_sessions.created_at < '2012-04-12'
  AND utm_source = 'gsearch'
  AND utm_campagin = 'nonbrand'

```

🔔 첫번째 요청과 동일하게 '2012-04-12' 이전을 조건으로 걸었고, 첫번째 쿼리 결과로 나온 가장 많은 세션을 차지하는 `source='gsearch' & utm_campaign = 'nonbrand'` 조건을 추가했습니다.  

💡 결과 해석하기
  - 세션-구입 전환율은 0.0296으로 약 3% 정도인 것으로 나왔으며, 이는 유료 마케팅 캠페인 비용을 감축할 필요가 있는 것으로 보인다.
  - 가장 세션 유입량이 높은 조건의 전환율이 역 3%라는 것은 동종업계 혹은 이커머스에서 경쟁력이 있는 것인가 싶고, 어떻게 이 비율을 높일 것인지에 대한 많은 고민과 시도가 필요해 보인다.


<br>

****************

<br>

# ✔ 무엇을 남겼나

- 유료 캠페인 트래픽(Paid Campaign Traffic), utm parameters(UTM)의 기본적인 내용을 이해하고 그걸 바탕으로 테이블을 파악함과 동시에 쿼리를 작성했다. 
단순히 결과 추출만 하는 것보다 배경지식과 이해가 수반되어야 쿼리를 짜고 결과를 해석하는 데 더 용이하다고 생각했기 때문이다. 이 과정을 통해 비즈니스 차원에서 원하는 데이터 분석 및 추출이 어떤 것일지 대략적으로 알게 되었고, 더 많은 부분들을 연습해보고 알아가야겠다는 것을 배웠다.


<br><br>