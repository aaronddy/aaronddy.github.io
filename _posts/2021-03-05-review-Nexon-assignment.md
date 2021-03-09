---
title:  "[Review] Nexon 데이터 분석 인턴 과제 리뷰"
excerpt: "Nexon - Data Anlysis 인턴 직군으로 지원, 과제로 진행한 분석 주제와 방향, 코드를 뜯어보며 무엇을 남겼는지 고민해 보자."

categories:
  - Review
tags:
  - [Review, DataAnlysis, Python, Algorithm]

toc: true
toc_sticky: true
 
date: 2021-03-05
last_modified_at: 2021-03-09
---
*********

# ✔ 분석 방향
### # 주제: 서비스 현황 분석과 서비스 관계분석, 유저 세그먼트 <br>
<strong>🔔  데이터 컬럼 설명</strong>
- monthCode: 날짜(월 단위)
- serviceId: 서비스 id
- userId: 유저 id
- timeSpent: 방문 시간(분 단위)
- numberOfDays 방문 일수 (일 단위)
- timePerDay: 유저의 일일 평균이용시간(timeSpent / numberOfDays, 해당 컬럼은 제가 추가한 컬럼입니다.)

<ol>
1. 2020년의 전체 서비스 현황을 요약하기<br>
  &nbsp; &nbsp; 🙋‍♀️ 전반적인 서비스의 현황을 어떻게 살펴보는 게 좋을지 고민했습니다. 일단 각 카테고리 별 서비스 이용 현황을 살펴보는 게 좋겠다고 생각했습니다.
<ol>
1) 2020년 '월별 중복 포함 유저 수'와 '월별 중복 제외 유저 수' 현황 비교<br>
2) 2020년 서비스별 이용자 수 현황 및 이용자 수가 높은 서비스 파악<br>
3) 2020년 서비스별 '일일 평균 이용시간' 현황(mean, median)<br>
4) 2020년 서비스별 '한달 이용 일수' 현황(mean, median)<br>
5) 2020년 유저별 서비스 이용 횟수 현황<br>
</ol>
<br>
2. 각 서비스별로 특이사항이 있는지 찾아보고 발견한 내용을 설명하기<br>
  &nbsp; &nbsp; 🙋‍♀️ 각각의 서비스를 하나씩 고려해서 찾아보지 않고, 위에서 진행한 서비스 현황으로 얻은 결과를 바탕으로 필터링을 진행했습니다. 필터링 된 서비스들이 보여주는 결과로 토대로 '무엇을' 의미하는지 해석해 보았습니다.
<ol>
1) 어떤 서비스가 상대적으로 높은 <u>고정 유저 수</u>를 확보하고 있을까?<br>
<ul>
- '서비스 별 이용자 수' <u>랭킹 3위</u>의 서비스 필터링 => 해당 서비스의 <u>이용자 수 평균과 표준편차</u>를 구한 후, <strong>표준편차가 가장 작은 서비스</strong>가 무엇인지 살펴보았습니다.<br>
</ul>
2) 매달 이용자 수가 안정적인 서비스는 무엇일까?<br>
<ul>
- '안정적'인 서비스는 <strong>서비스별 이용자 수 변동이 크지 않음</strong>이라고 정의하였습니다.<br>
- 월별 서비스 이용자수의 평균, 표준편차 분포 및 <strong>'변동계수'</strong>를 비교하여 <u>매달 이용자 수가 가장 '안정적'이라고 할 수 있는 서비스가 무엇</u>인지 파악해 보았습니다.
</ul>
3) 각 서비스의 일일 접속시간은 안정적인가?<br>
<ul>
- '서비스 별 일일 평균 이용시간' <u>랭킹 3위</u>의 서비스 필터링 => 월별 해당 서비스의 <u>일일 평균시간의 평균과 표준편차</u>를 구한 후, <strong>분포 및 '변동계수'</strong>를 비교했습니다.<br>
- 그 결과를 바탕으로 <u>헤딩 서비스의 일일 접속시간은 '안정적'</u>인지 파악해 보았습니다.
</ul>
4) 각 서비스의 한달 이용 일수 분포는 어떻게 될까?<br>
<ul>
- '서비스 별 이용 일수' <u>랭킹 3위</u>의 서비스 필터링 => 월별 해당 서비스의 <u>이용일수 평균, 표준편차</u>를 구한 후, <strong>표준편차가 작은 서비스</strong>가 무엇인지 살펴보았습니다.<br>

</ul>
</ol>
3. 각 서비스들 사이의 관계를 분석해서 서비스 기획에 활용할 만한 게 있다면 제안하기<br>
&nbsp; &nbsp; 🙋‍♀️ 유저들 중 서비스 이용 횟수가 높은 유저들이 어떤 서비스를 이용했고, 해당 서비스 세트에 <u>'연관규칙분석(Association Rules)'</u>을 적용해 <strong>서비스간 연관성이 높은 세트가 무엇인지</strong> 확인하고 분석해 보았습니다. 이는 <em>연관성이 높은 서비스 세트를 파악해 추천 시스템</em>으로 활용 가능 하다고 생각했습니다. <br>
<ul>
- '연관규칙분석'은 <u>"'A 서비스'를 이용하면 'B 서비스'도 이용하더라"</u>라는 서비스 간의 연관관계를 파악하기 위함이며, <em>Apriori 알고리즘</em>을 이용해 분석을 진행했습니다.<br>
</ul>
<br>
4. 유저들을 여러 그룹으로 나눠서 관리하고자 하는데, 유저들의 특성을 분석해서 적절한 전략을 마련하기<br>
<ol>
1). feature별 상관관계는 어떠할까?<br>
<ul>
- 서비스 feature별 어떤 feature끼리 상관관계가 있는지 먼저 파악한 후, 관련 feature를 기준으로 유저들을 그룹화할 수 있겠다고 생각했습니다.<br>
- 3개의 요인 설정: 서비스별 유저 수('count_total_users'), 서비스별 일일 평균이용시간('time_mean'), 서비스별 한달 이용일수('days_mean')<br>
</ul>
2). '서비스별 유저 수'와 '서비스별 이용 일수'를 고려해 유저 분류 및 분석<br>
3). '서비스별 일일 평균이용시간'과 '서비스별 이용 일수' 고려해 유저 분류 및 분석<br>
</ol>
</ol>

***************
<br>

# ✔ 분석 코드
### 연관규칙분석(Association Rules) 알고리즘 진행 과정 및 코드<br>
> 🔔 해당 개념에 대해 간단하게 설명하자면, 모든 서비스들 간의 조합을 생성 => <strong>연관성이 있다고 볼 수 있는 "후보" 규칙을 파악</storng>할 수 있는 분석 방식입니다. 이 방법을 통해 '연관성이 높은 세트'가 발견되면 서비스 <u>'추천시스템'</u>으로 혹은 마케팅으로 활용할 수 있겠다고 생각했습니다. <br>

<ol>
1) 유저들의 서비스 이용횟수를 카운트 해, 2020년 한 해 동안 36회 이상 이용한 유저를 필터링<br>

```python
# 유저별 이용횟수가 담긴 'users' 테이블에서 'count' 컬럼을 기준으로 내림차순, 해당 테이블을 'users_sort'로 저장
users_sort = users.sort_values(by='count', ascending=False)

# 'users_sort'테이블의 'count' 컬럼 값이 36 이상만 추출
users_sort = users_sort[users_sort['count'] >= 36]

```
💡 횟수를 36회로 정한 이유는 한달에 3가지 이상의 서비스를 이용(12*3)했을 때를 고려하였습니다.
<br><br>
2) 36회 이상 이용한 유저들의 'userId'만 추출

```python
# 'users_sort' 테이블의 index(userId)를 데이터프레임으로 저장
users_filtering = pd.DataFrame(users_sort.index)

# 해당 테이블('users_filtering')의 컬럼 'userId' 값을 range_users로 저장
range_users = users_filtering['userId']

```
<br><br>
3) for loop을 이용해 모든 서비스 세트를 리스트에 담기

```python
total_trans = []            # 빈 리스트 생성

count = 0
for i in range_users:       # range_users(필터링된 userId)를 for loop
    find_user = df[df['userId'] == i]
    count_trans = list(find_user['serviceId'].unique())
    count += 1
    
    total_trans.append(count_trans)

```

<br><br>
4) apriori 관련 패키지 'apyori' 설치 및 파라미터 설정 

```python
import sys
!{sys.executable} -m pip install apyori
from apyori import apriori


# 최소지지도: min_support, 최소신뢰도: min_confidence, 최소향상도: min_lift
# 최소아이템개수: min_length, 최대아이템개수: max_length)

records = total_trans

associations = apriori(records, min_support=0.03, min_confidence=0.6, min_lift=2, min_length=2, max_length=3)
associations = list(associations)

print("모든 조건을 만족하는 서비스 세트는", len(associations), '개 입니다.')

>>> 모든 조건을 만족하는 서비스 세트는 3 개 입니다.
```
<br><br>
5) 조건을 만족하는 서비스 세트 3개 상세하게 살펴보기

```python
for item in associations:

    # first index of the inner list
    # Contains base item and add item
    pair = item[0] 
    print(pair)
    
    items = [x for x in pair]
    print("Rule: " + str(item[2][0][0]) + " -> " + str(item[2][0][1]))
    
    #second index of the inner list
    print("Support: " + str(round(item[1], 5)))

    #third index of the list located at 0th
    #of the third index of the inner list

    print("Confidence: " + str(round(item[2][0][2], 5)))
    print("Lift: " + str(round(item[2][0][3], 5)))
    print("=====================================")

```
</ol>
***************************************
<br>

# ✔ 분석 결과

# ✔ 배운 점 및 느낀 점