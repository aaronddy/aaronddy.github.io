---
title:  "[PY] 간단하지만 유용한 데이터 클렌징 코드s in Pandas"
excerpt: "간단하지만 강력한 데이터 클렌징 코드를 알아보자."

categories:
  - Python
tags:
  - [Python, Pandas, data, cleansing]

toc: true
toc_sticky: false
toc_label: "어떻게 진행할 것이냐"
toc_icon: "cookie-bite"
 
date: 2021-05-19
last_modified_at: 2021-05-19
---


**************

<br>

🔔 판다스를 이용해 데이터를 가공할때 한 번쯤은 사용하는 코드들을 기록했습니다.

<br>

# ✔ 테이블 컬럼명을 변경하고 싶을 땐?

1) rename

```python
# 'A':'B' => 'A' 컬럼명을 'B'로 바꿔줘
df = df.rename(columns={'요청일자':'확인일자', '노출기간':'확인기간', '상품명', '상품이름'})
```

<br>

# ✔ 테이블 컬럼값의 개수 혹은 비율을 확인할 땐?

1) value_counts() 

```python
# 개수 확인
cnt_category = df['category'].value_counts()

# 개수 조건 걸기, 100개 이상인 카테고리만 보여줘
cnt_category_100 = cnt_category[cnt_category >= 100]
```

2) value_counts(normalize=True)

```python
# 비율 확인
rte_category = df['category'].value_counts(normalize=True)

# 비율 조건 걸기, 상위 비율 10개만 보여주는데, 소수점 두자릿 수까지 반올림해줘
rte_category_n = round(rate_category[:10], 2)
```

<br>

# ✔ 테이블 컬럼값을 sorting할 땐?

1) sort_values()

```python
# 올림차순 기본정렬, 'mean'이라는 컬럼명의 값을 올림차순 정렬 해줘
sort_category = df.sort_values(by='mean')
```

2) sort_values(ascending=False)

```python
# 내림차순 정렬, 'mean'이라는 컬럼명의 값을 내림차순 정렬 해줘
sort_category = df.sort_values(by='mean', ascendong=False)

# 정렬 컬럼이 여러개일 때, 'mean', 'std' 컬럼명의 값들을 내림차순 정렬해줘
sort_category = df.sort_values(['mean', 'std'], ascending=(0,0))

# 정렬 컬럼이 여러개일 때, 'mean'은 올림차순, 'std'는 내림차순 정렬해줘
sort_category = df.sort_values(['mean', 'std'], ascending=(1,0))
```

<br>

# ✔ 조건을 걸어서 행을 필터링하고 싶을 땐?

1) 연산 부호 혹은 등호를 쓸 경우

```python
n1 = 10
n2 = 40
category = 'fruits'

# 기준이 한개, 'balance-count' 컬럼값이 n1 이상인 행들만 cat_bnd_n1에 저장해줘
cat_bnd_n1 = cat_brands.loc[cat_brands[('balance','count')] >= n1]

# 기준이 한개, 'name-category' 컬럼값이 fruits인 행들만 cat_name에 저장해줘
cat_name = cat.loc[cat[('name','category')] == category]

# 기준이 두개 이상
# 'balance-count' 컬럼값이 n1 이상이면서 'balance-std' 컬럼값이 n2 이하인 행들만 cat_bnd_n1_n2에 저장해줘
cat_bnd_n1_n2 = cat_brands.loc[(cat_brands[('balance','count')] >= n1] & (cat_brands[('balance','std')] <= n2]) 
```

2) 리스트를 쓸 경우(포함 / 미포함)

```python
category_yes = ['숟가락', '나이프', '주걱', '선풍기', '거울']
category_no = ['책상', '창문', '블라인드', '소파']

# 원하는 조건에 해당하는 경우를 필터링할 경우
# df의 'cat_name' 칼럼값이 category_yes에 해당하는 행들만 depth_df에 저장해줘
depth_yes = df.loc[df['cat_name'].isin(category_yes)]


# 원하는 조건에 해당하지 않는 경우를 필터링할 경우
# df의 'cat_name' 컬럼값이 category_no에 해당하는 행들을 제외하고 depth_no에 저장해줘
pattern = '|'.join(category_no)
depth_no = df[df['cat_name'].str.contains(pattern) == False]

# 위의 코드를 다시 풀어 쓰면
depth_no = df[df['cat_name'].str.contains('책상|창문|블라인드|소파') == False]
```

<br>

**************

<br>

📌 추후에도 계속 업데이트할 예정입니다👩🏻‍💻