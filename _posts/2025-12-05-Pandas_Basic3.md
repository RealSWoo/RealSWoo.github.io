---
layout: post
title:  "10minutes to Pandas 데이터 선택, 결측치 처리, 연산 중 결측치 처리, 연산"
date:   2025-12-05 00:00:00 +0900
categories: Pandas
---
pandas 공식 홈페이지에서 "10minutes to pandas"를 보고 공부 및 번역한 내용입니다.

* 해당 코드는 Visual Studio Code에서 실행하였습니다.
* 오타나 오번역 있을 시 말씀해주시면 수정 반영하겠습니다.

&nbsp;

&nbsp;

먼저 결측치 처리에 대해 알아보겠습니다.

&nbsp;

NumPy 데이터 타입에서 np.nan은 결측 데이터를 나타냅니다. 이 데이터는 기본적으로 계산에 포함되지 않습니다.

Reindexing은 지정된 축에 있는 인덱스를 변경/추가/삭제할 수 있게 해주며 이 연산은 데이터의 복사본을 반환합니다.

즉, Reindexing은 Series나 DataFrame의 행 인덱스 또는 열 레이블을 새로운 순서, 길이, 레이블로 변경하는 연산이라고 할 수 있습니다.

데이터 복사본을 반환하기 때문에 DataFrame 결과를 사용하려면 변수에 다시 할당해야합니다.

```ruby
[In]
df1 = df.reindex(index=dates[0:4], columns=list(df.columns) + ["E"]

[In]
df1.loc[dates[0] : dates[1], "E"] = 1

[In]
print(df1)

[Out] 
                   A         B         C    D    F    E
2013-01-01  0.000000  0.000000 -1.509059  5.0  NaN  1.0
2013-01-02  1.212112 -0.173215  0.119209  5.0  1.0  1.0
2013-01-03 -0.861849 -2.104569 -0.494929  5.0  2.0  NaN
2013-01-04  0.721555 -0.706771 -1.039575  5.0  3.0  NaN
```

&nbsp;

DataFrane.dropna()는 결측 데이터를 가지고 있는 모든 행을 제거합니다.(데이터 정리하는데 사용)[

```ruby
[In]
print(df1.dropna(how="any"))

[Out] 
                   A         B         C    D    F    E
2013-01-02  1.212112 -0.173215  0.119209  5.0  1.0  1.0
```

&nbsp;

DataFrame.fillna()는 데이터프레임 내에 있는 결측값(np.nan)을 지정된 값이나 방법으로 채우는데 사용됩니다.

```ruby
[In]
print(df1.fillna(value=5))

[Out] 
                   A         B         C    D    F    E
2013-01-01  0.000000  0.000000 -1.509059  5.0  5.0  1.0
2013-01-02  1.212112 -0.173215  0.119209  5.0  1.0  1.0
2013-01-03 -0.861849 -2.104569 -0.494929  5.0  2.0  5.0
2013-01-04  0.721555 -0.706771 -1.039575  5.0  3.0  5.0
```

&nbsp;

isna()는 값이 nan인 위치의 불리언 마스크를 얻는데, 데이터 내의 결측 값을 찾아 각 셀의 값이 nan인지 여부를 나타내어 불리언 DataFrame을 반환하는 것입니다.

```ruby
[In]
print(pd.isna(df1))

[Out] 
                A      B      C      D      F      E
2013-01-01  False  False  False  False   True  False
2013-01-02  False  False  False  False  False  False
2013-01-03  False  False  False  False  False   True
2013-01-04  False  False  False  False  False   True
```

&nbsp;

&nbsp;

이제 연산(Operations)에 대하여 알아보겠습니다.

&nbsp;

통계(Stats)에서 일반적으로 연산은 결측 데이터를 제외합니다.

왜냐하면 NaN은 값이 아니므로, 이를 포함하여 계산할 경우 결과 통계치가 왜곡되거나 무의미해지기 때문입니다.

```ruby
[In]
print(df.mean())

[Out] 
A   -0.004474
B   -0.383981
C   -0.687758
D    5.000000
F    3.000000
dtype: float64
```

mean() 메서드는 인자 없이 호출하면 axis = 0으로 연산이 수행됩니다.

즉, 각 열의 평균을 계산하여 Series로 반환하는 것입니다.

&nbsp;

이제 각 행에 대한 평균값을 볼 차례입니다.(axis=1)

```ruby
[In]
print(df.mean(axis=1))

[Out] 
2013-01-01    0.872735
2013-01-02    1.431621
2013-01-03    0.707731
2013-01-04    1.395042
2013-01-05    1.883656
2013-01-06    1.592306
Freq: D, dtype: float64
```

Pandas DataFrame에서 각 행의 평균을 계산하려면, DataFrame 메서드인 df.mean()을 호출할 때 axis=1 인수를 명시해주어야 합니다.

다른 인덱스나 열을 가진 Series나 DataFrame을 연산할 경우, 그 결과는 인덱스 또는 열 레이블의 합집합으로 맞추어 정렬됩니다.

또한 Pandas는 지정된 차원을 따라 자동으로 Broadcasting을 수행하며, 정렬되지 않은 레이블은 np.nan으로 채웁니다.

&nbsp;

★ Broadcasting이란?

서로 다른 차원(dimension)을 가진 DataFrame과 Series를 연산할 때, Pandas는 Numpy와 유사하게 작은 차원의 객체를 큰 차원의 객체의 모든 행(또는 열)에 걸쳐 확장하여 연산을 가능하게 하는 것입니다.

즉, 복잡한 데이터 정렬이나 반복 루프 없이 다양한 형태의 데이터를 안전하게 결합하고 연산할 수 있게 됩니다.

&nbsp;

```ruby
[In]
s = pd.Series([1, 3, 5, np.nan, 6, 8], index=dates).shift(2)

[In]
print(s)

[Out]: 
2013-01-01    NaN
2013-01-02    NaN
2013-01-03    1.0
2013-01-04    3.0
2013-01-05    5.0
2013-01-06    NaN
Freq: D, dtype: float64

[In]
print(df.sub(s, axis="index"))

[Out] 
                   A         B         C    D    F
2013-01-01       NaN       NaN       NaN  NaN  NaN
2013-01-02       NaN       NaN       NaN  NaN  NaN
2013-01-03 -1.861849 -3.104569 -1.494929  4.0  1.0
2013-01-04 -2.278445 -3.706771 -4.039575  2.0  0.0
2013-01-05 -5.424972 -4.432980 -4.723768  0.0 -1.0
2013-01-06       NaN       NaN       NaN  NaN  NaN
```

&nbsp;

사용자 정의 함수에서 DataFrame.agg()와 DataFrame.transform()은 각각 결과를 축소하거나 브로드캐스트하는 사용자 정의 함수를 적용합니다.

```ruby
[In]
print(df.agg(lambda x: np.mean(x) * 5.6))

[Out] 
A    -0.025054
B    -2.150294
C    -3.851445
D    28.000000
F    16.800000
dtype: float64

[In]
print(df.transform(lambda x: x * 101.2))

[Out] 
                     A           B           C      D      F
2013-01-01    0.000000    0.000000 -152.716721  506.0    NaN
2013-01-02  122.665737  -17.529322   12.063922  506.0  101.2
2013-01-03  -87.219115 -212.982405  -50.086843  506.0  202.4
2013-01-04   73.021382  -71.525239 -105.204988  506.0  303.6
2013-01-05  -43.007200   57.382459   27.954680  506.0  404.8
2013-01-06  -68.177398   11.501219 -149.616767  506.0  506.0
```

&nbsp;

value_counts() 메서드를 이용해 값의 개수를 셀 수 있습니다.

```ruby
[In]
s = pd.Series(np.random.randint(0, 7, size=10))

[In]
print(s)

[Out] 
0    4
1    2
2    1
3    2
4    6
5    4
6    4
7    6
8    4
9    4
dtype: int64

[In]
print(s.value_counts())

[Out] 
4    5
2    2
6    2
1    1
Name: count, dtype: int64
```

value_counts는 Pandas series에서 고유한 값(Unique Values)의 출현 빈도를 계산하는 메서드입니다.

이 메서드는 데이터의 분포를 파악하는데 가장 핵심적인 메서드입니다.

&nbsp;

문자열 메서드 Series는 str 속성 내에 일련의 문자열 처리 메서드를 갖추고 있습니다.

str 속성의 역할은 Series 객체 내의 각 문자열 요소에 대해 Python 내장 문자열 메서드와 정규 표현식 기반 메서드를 반복 없이 빠르게 적용할 수 있도록

벡터화된 인터페이스를 제공합니다.

```ruby
[In]
s = pd.Series(["A", "B", "C", "Aaba", "Baca", np.nan, "CABA", "dog", "cat"])

[In]
print(s.str.lower())

[Out] 
0       a
1       b
2       c
3    aaba
4    baca
5     NaN
6    caba
7     dog
8     cat
dtype: object
```

즉, Vectorized String Methods는 Pandas에서 텍스트 데이터 클리닝과 전처리 작업을 빠르고 효율적으로 할 수 있게 해주는 핵심 기능입니다.

&nbsp;

&nbsp;

지금까지 결측치 데이터와 연산에 대하여 알아보았습니다.

&nbsp;

감사합니다.
