---
title:  "[Review] 데이터 분석 인턴 과제 리뷰"
excerpt: "데이터 분석 인턴 전형에서 과제로 진행한 내용을 정리했습니다."

categories:
  - Review
tags:
  - [Review, DataAnlysis, Python, Algorithm]

toc: true
toc_sticky: false
toc_label: "어떻게 진행할 것이냐"
toc_icon: "cookie-bite"

date: 2021-03-05
last_modified_at: 2021-03-13
---


*********

<br>

# 🔎 주제: 서비스 현황 분석과 서비스 관계분석, 유저 세그먼트 <br>

🔔  데이터 컬럼 설명 🔔
{% highlight html linenos %}
- monthCode: 날짜(월 단위)
- serviceId: 서비스 id
- userId: 유저 id
- timeSpent: 방문 시간(분 단위)
- numberOfDays 방문 일수 (일 단위)
- timePerDay: 유저의 일일 평균이용시간(timeSpent / numberOfDays, 해당 컬럼은 제가 추가한 컬럼입니다.)
{% endhighlight %}

🚫 해당 데이터 저작권은 'Nexon'에 귀속된 바, 상세내용은 담지 않았습니다. 어떻게 이 과제를 진행했는지를 기록하였습니다.
<br>

*********************************

<br>

# ✔ 분석 방향

A. 2020년의 전체 서비스 현황을 요약하기<br>
<ol>
1) 2020년 '월별 중복 포함 유저 수'와 '월별 중복 제외 유저 수' 현황 비교<br>
2) 2020년 서비스별 이용자 수 현황 및 이용자 수가 높은 서비스 파악<br>
3) 2020년 서비스별 '일일 평균 이용시간' 현황(mean, median)<br>
4) 2020년 서비스별 '한달 이용 일수' 현황(mean, median)<br>
5) 2020년 유저별 서비스 이용 횟수 현황<br>
</ol>

> 🙋‍♀️ 전반적인 서비스의 현황을 어떻게 살펴보는 게 좋을지 고민했습니다. 일단 각 카테고리 별 서비스 이용 현황을 살펴보는 게 좋겠다고 생각했습니다.

<br>

B. 각 서비스별로 특이사항이 있는지 찾아보고 발견한 내용을 설명하기
<ol>
1) 어떤 서비스가 상대적으로 높은 <u>고정 유저 수</u>를 확보하고 있을까?<br>
<ul>
- '서비스 별 이용자 수' <u>랭킹 3위</u>의 서비스 필터링 => 해당 서비스의 <u>이용자 수 평균과 표준편차</u>를 구한 후, <strong>표준편차가 가장 작은 서비스</strong>가 무엇인지 살펴보았습니다.<br>
</ul>
<br>
2) 매달 이용자 수가 안정적인 서비스는 무엇일까?<br>
<ul>
- '안정적'인 서비스는 <strong>서비스별 이용자 수 변동이 크지 않음</strong>이라고 정의하였습니다.<br>
- 월별 서비스 이용자수의 평균, 표준편차 분포 및 <strong>'변동계수'</strong>를 비교하여 <u>매달 이용자 수가 가장 '안정적'이라고 할 수 있는 서비스가 무엇</u>인지 파악해 보았습니다.
</ul>
<br>
3) 각 서비스의 일일 접속시간은 안정적인가?<br>
<ul>
- '서비스 별 일일 평균 이용시간' <u>랭킹 3위</u>의 서비스 필터링 => 월별 해당 서비스의 <u>일일 평균시간의 평균과 표준편차</u>를 구한 후, <strong>분포 및 '변동계수'</strong>를 비교했습니다.<br>
- 그 결과를 바탕으로 <u>헤딩 서비스의 일일 접속시간은 '안정적'</u>인지 파악해 보았습니다.
</ul>
<br>
4) 각 서비스의 한달 이용 일수 분포는 어떻게 될까?<br>
<ul>
- '서비스 별 이용 일수' <u>랭킹 3위</u>의 서비스 필터링 => 월별 해당 서비스의 <u>이용일수 평균, 표준편차</u>를 구한 후, <strong>표준편차가 작은 서비스</strong>가 무엇인지 살펴보았습니다.<br>
</ul>
<br>
</ol>

> 🙋‍♀️ 각각의 서비스를 하나씩 고려해서 찾아보지 않고, 위에서 진행한 서비스 현황으로 얻은 결과를 바탕으로 필터링을 진행했습니다. 필터링 된 서비스들이 보여주는 결과로 토대로 '무엇을' 의미하는지 해석해 보았습니다.

<br>
<br>

C. 각 서비스들 사이의 관계를 분석해서 서비스 기획에 활용할 만한 게 있다면 제안하기<br>

- '연관규칙분석'은 <u>"'A 서비스'를 이용하면 'B 서비스'도 이용하더라"</u>라는 서비스 간의 연관관계를 파악하기 위함이며, <em>Apriori 알고리즘</em>을 이용해 분석을 진행했습니다.<br>

> 🙋‍♀️ 유저들 중 서비스 이용 횟수가 높은 유저들이 어떤 서비스를 이용했고, 해당 서비스 세트에 <u>'연관규칙분석(Association Rules)'</u>을 적용해 <strong>서비스간 연관성이 높은 세트가 무엇인지</strong> 확인해 보았습니다. 
이는 <em>연관성이 높은 서비스 세트를 파악해 추천 시스템</em>으로 활용 가능 하다고 생각했습니다. <br>

<br>
<br>

D. 유저들을 여러 그룹으로 나눠서 관리하고자 하는데, 유저들의 특성을 분석해서 적절한 전략을 마련하기<br>
<ol>
1). feature별 상관관계는 어떠할까?<br>
<ul>
- 서비스 feature별 어떤 feature끼리 상관관계가 있는지 먼저 파악한 후, 관련 feature를 기준으로 유저들을 그룹화할 수 있겠다고 생각했습니다.<br>
- 3개의 요인 설정: 서비스별 유저 수('count_total_users'), 서비스별 일일 평균이용시간('time_mean'), 서비스별 한달 이용일수('days_mean')<br>
</ul>
2). '서비스별 유저 수'와 '서비스별 이용 일수'를 고려해 유저 분류 및 분석<br>
3). '서비스별 일일 평균이용시간'과 '서비스별 이용 일수' 고려해 유저 분류 및 분석<br>
</ol>

> 🙋‍♀️ 상관관계가 있는 요인들을 고려해 유저를 여러 그룹으로 나눌 수 있겠다고 생각했고, 2가지 경우를 고려해 유저 분류 및 분석을 진행했습니다.

<br>

***************

<br>

# ✔ 연관규칙분석(Association Rules) 알고리즘 진행 과정 및 코드

> 🔔 해당 개념에 대해 간단하게 설명하자면, 모든 서비스들 간의 조합을 생성 => 연관성이 있다고 볼 수 있는 "후보" 규칙을 파악할 수 있는 분석 방식입니다. 이 방법을 통해 '연관성이 높은 세트'가 발견되면 서비스 <u>'추천시스템'</u>으로 혹은 마케팅으로 활용할 수 있겠다고 생각했습니다. <br>

A. 유저의 서비스 이용 횟수 카운팅, 조건 필터링

2020년 한 해 동안의 각 유저의 서비스 이용횟수를 카운팅 후, 36회 이상인 유저 필터링

```python
# 유저별 이용횟수가 담긴 'users' 테이블에서 'count' 컬럼을 기준으로 내림차순, 해당 테이블을 'users_sort'로 저장
users_sort = users.sort_values(by='count', ascending=False)

# 'users_sort'테이블의 'count' 컬럼 값이 36 이상만 추출
users_sort = users_sort[users_sort['count'] >= 36]

```

 💡 횟수를 36회로 정한 이유는 한달에 3가지 이상의 서비스를 이용(12*3)했을 때를 고려하였습니다.

<br>

B. 조건 충족한 유저의 'userId' 추출

```python
# 'users_sort' 테이블의 index(userId)를 데이터프레임으로 저장
users_filtering = pd.DataFrame(users_sort.index)

# 해당 테이블('users_filtering')의 컬럼 'userId' 값을 range_users로 저장
range_users = users_filtering['userId']

```
<br>

C. for loop으로 모든 서비스 세트를 리스트에 담기

```python
total_trans = []            # 빈 리스트 생성

count = 0
for i in range_users:       
    # range_users(필터링된 userId)를 for loop
    
    find_user = df[df['userId'] == i]
    # 원시 데이터 테이블에서 'userId'가 동일하면 'find_user'에 저장
    count_trans = list(find_user['serviceId'].unique())
    # 해당 유저가 이용한 distinct한 'serviceId'를 리스트로 묶어서 'count_trans'에 저장
    count += 1
    
    total_trans.append(count_trans)
    # total_trans에 append

```
<br>

D. apriori 관련 패키지 'apyori' 설치 및 파라미터 설정 
<br>
🔔 'Apriori' 알고리즘은 3개의 중요한 지표를 가지고 후보 세트를 선정합니다.
- Support(지지도): <strong>규칙의 중요성에 대한 척도</strong>로 'A'와 'B' 서비스를 함께 이용한 경우의 수와 전체 경우의 수를 나눠 구합니다. 
- Confidence(신뢰도): <strong>서비스들간의 연관성의 정도</strong>를 파악, 'A'를 이용할 시 얼마나 'B'를 이용하는 것으로 이어지는지에 대한 것입니다.
- Lift(향상도): <strong>서비스 A, B가 독립적인지, 상관이 있는지 파악</strong>하는 것으로 <em>1보다 크면 항목간 <u>양의 상관관계</u>를 의미하고 1일 경우는 <u>독립적</u>, 0과 1 사이일 경우에는 항목간 <u>음의 상관관계</u></em>로 봅니다.
<br>

```python
import sys
!{sys.executable} -m pip install apyori
from apyori import apriori


# 최소지지도: min_support, 최소신뢰도: min_confidence, 최소향상도: min_lift
# 최소아이템개수: min_length, 최대아이템개수: max_length

records = total_trans

associations = apriori(records, min_support=0.03, min_confidence=0.6, min_lift=2, min_length=2, max_length=3)

# 리스트 형태로 담기
associations = list(associations)

print("모든 조건을 만족하는 서비스 세트는", len(associations), '개 입니다.')

>>> 모든 조건을 만족하는 서비스 세트는 3 개 입니다.
```

💡 여러번의 파라미터 조정을 진행하였고, 가장 적합한 기준을 찾았습니다. 그 결과 총 3개의 서비스 세트가 나왔습니다.
<br>

💡 'apriori' 파라미터에 대해 설명하자면,
- min_support(최소지지도): 지지도의 최저 기준
- min_confidence(최소신뢰도): 신뢰도의 최저 기준
- min_lift(최소향상도): 향상도의 최저 기준
- min_length(최소아이템개수): 최소 아이템 개수(최소한 n개 이상은 되어야 한다)
- max_length(최대아이템개수): 최대 아이템 개수(최대 n개 이하만 고려한다)

<br>

E. 최종 서비스 세트 살펴보기

```python

for item in associations:
    # 서비스 세트 하나씩 loop

    # 내부 리스트의 첫번째 인덱스
    # 서비스 세트 항목을 'pair'에 저장
    pair = item[0] 
    print(pair)
    
    items = [x for x in pair]
    # "'A'서비스를 이용하면 'B'서비스도 이용한다"는 연관관계를 보여주는
    print("Rule: " + str(item[2][0][0]) + " -> " + str(item[2][0][1]))

    # 내부 리스트의 두번째 인덱스
    # 서비스 세트의 연관관계의 지지도
    print("Support: " + str(round(item[1], 5)))

    # 서비스 세트의 연관관계의 신뢰도와 향상도
    print("Confidence: " + str(round(item[2][0][2], 5)))
    print("Lift: " + str(round(item[2][0][3], 5)))
    print("=====================================")


    >>> 예시 결과

    frozenset({'apple', 'forest', 'frogs'})
    Rule: frozenset({'apple'}) -> frozenset({'forest', 'frogs'})
    Support: 0.075
    Confidence: 0.66
    Lift: 2.1
    =====================================

```

```
# 결과 해석

- 서비스 {'apple', 'forest', 'frogs'}은 36회 이상 서비스를 이용하는 유저(N명)중 7.5%가 함께 이용한 서비스 세트이다. (Support)
- ('forest', 'frogs') 두 개의 서비스를 다 이용한 유저들 중 약 66%는 'apple' 서비스도 함께 이용했다.
- ('forest', 'frogs') 서비스를 이용한 유저는 'apple' 서비스만 단독으로 이용한 유저'보다 'apple' 서비스를 이용할 가능성이 2.1배 더 높다.

```

<br>

***************************************

<br>

# ✔ 해당 과제를 진행하며 배운 점
- 해당 과제 주제가 '서비스 현황 분석과 서비스 관계분석, 유저 세그먼트'라는 것은 명시됐지만, 그 외의 질문은 따로 받지 않았습니다. 그래서 어떠한 방향과 방식으로 풀어나갈지 많이 고민하고 여러 번 수정하며 진행했지만 그럼에도 아쉬운 점이 많이 남았습니다.
- '각 서비스들 사이의 관계를 분석'하기 위해 연관규칙 분석 방법을 이용해서 연관성이 높은 서비스 세트를 추출하긴 했지만 이걸 어떻게 서비스 마케팅 혹은 기획으로 활용할 수 있는가에 대한 전략적인 설명은 많이 미약했습니다.
- 데이터를 가지고 분석을 한다는 건 목적이 될 수 없고 도구일 뿐인데, 그 동안 도구를 잘 다루고 방법을 많이 아는 게 중요하다고 생각했지만 이번 과제를 진행하며 그게 아니라는 것을 배웠습니다. 
- 그럼에도 데이터를 분석하기 위해 파이썬으로 데이터 가공 및 정제하는 과정은 너무나도 보람차고 재미있었습니다. 하지만 데이터 시각화는 너무나도 미흡했기에 앞으로 계속 보충해 나가야겠다고 다짐하는 시간이기도 했습니다.
- 통계 공부를 열심히 해야겠다는 것도 많이 느꼈습니다🥲🥲🥲🥲🥲🥲

<br>
<br>