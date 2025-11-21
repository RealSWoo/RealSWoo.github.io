---
layout: post
title:  "Pandas Basic"
date:   2025-11-21 00:00:00 +0900
categories: Pandas
---
pandas 공식 홈페이지에서 "10minutes to pandas"를 보고 공부 및 번역한 내용입니다.

<해당 코드는 Visual Studio Code에서 실행하였습니다.>

<오타나 오번역 있을 시 말씀해주시면 수정 반영하겠습니다.>


우선 다음과 같이 import 합니다

```
import numpy as np
import pandas as pd
```

- pandas 라이브러리를 import(불러오기)하고 pd 라고 명명합니다. 이렇게 하면 코드 작성시 pandas 대신 pd를 사용하여 pandas 기능에 접근할 수 있습니다.
- numpy도 마찬가지로 np라고 명명합니다.


Pandas에서 데이터를 다루기 위해 제공하는 두 가지 기본 자료 구조는 다음과 같습니다.

1. Series : 1차원의 레이블이 지정된 배열(labeled array)입니다.
   차원 : 1차원(One-dimensional)
   구성 : 모든 유형의 데이터(정수, 문자열, 파이썬 객체 등)를 담을 수 있으며, 각 데이터 요소에는 인덱스(index)라고 불리는 레이블이 붙습니다.
2. DataFrame : 2차원의 테이블 형태의 자료 구조입니다.
   차원 : 2차원(Two-dimensional)
   구성 : 행(rows)과 열(columns)을 가진 테이블이나 2차원 배열처럼 데이터를 담습니다.
   * 각 열(column)은 하나의 Series 객체로 간주될 수 있습니다.
   * 각 열은 서로 다른 데이터 유형을 가질 수 있습니다.


Series 생성을 합니다.

```
[In]
s = pd.Series([1, 3, 5, np.nan, 6, 8])
print(s)

[Out]
0   1.0
1   3.0
2   5.0
3   NaN
4   6.0
5   8.0
dtype : float64
```

각 원소에 인덱스를 붙이는데, np.nan은 결측치(값이 없는 상태)를 의미합니다.

결측치 Nan 때문에 데이터 타입은 정수가 아닌 float64로 실수입니다.



DataFrame 생성을 합니다.

Numpy 배열과 date_range()를 사용한 Date time index 및 레이블이 지정된 열(labled columns)을 이용하여 DataFrame을 생성하는 방법입니다.

```
[In]
dates = pd.date_range("20130101", periods=6)
print(dates)

[Out] 
DatetimeIndex(['2013-01-01', '2013-01-02', '2013-01-03', '2013-01-04',
               '2013-01-05', '2013-01-06'],
              dtype='datetime64[ns]', freq='D')
```

```
[In]
df = pd.DataFrame(np.random.randn(6, 4), index=dates, columns=list("ABCD"))
print(df)

[Out]
                   A         B         C         D
2013-01-01  0.469112 -0.282863 -1.509059 -1.135632
2013-01-02  1.212112 -0.173215  0.119209 -1.044236
2013-01-03 -0.861849 -2.104569 -0.494929  1.071804
2013-01-04  0.721555 -0.706771 -1.039575  0.271860
2013-01-05 -0.424972  0.567020  0.276232 -1.087401
2013-01-06 -0.673690  0.113648 -1.478427  0.524988
```

날짜 시퀀스를 생성하는데, periods=6 즉, 총6일치 날짜를 만듭니다.

freq='D'가 기본값인데 하루 단위(Daily)로 증가하는 날짜를 생성합니다.

그리고 pd.DataFrame에서 2차원 표 형태 데이터 구조 생성함수를 만드는데 NumPy 함수를 6x4 배열을 생성합니다.

여기서 정규분포(평균 0, 표준편차 1)를 따르는 랜덤 결과값을 가집니다.

index=dates는 이전에 만든 dates를 행 인덱스로 지정하고 열 이름(columns=list)은 "A","B","C","D"입니다.

위 DataFrame이 갖는 특징은 날짜 기반 인덱스를 활용할 수 있어 시간 시계열 데이터 처리에 유리합니다.


키(Keys)를 열 레이블(Column Labels)로, 값(Values)을 열 값(column Values)으로 하는 딕셔너리(dictionary)를 이용하여 DataFrame을 생성 할 수 있습니다.

```
[In]
df2 = pd.DataFrame(
    {
        "A": 1.0,
        "B": pd.Timestamp("20130102"),
        "C": pd.Series(1, index=list(range(4)), dtype="float32"),
        "D": np.array([3] * 4, dtype="int32"),
        "E": pd.Categorical(["test", "train", "test", "train"]),
        "F": "foo",
    }
)
print(df2)

[Out] 
     A          B    C  D      E    F
0  1.0 2013-01-02  1.0  3   test  foo
1  1.0 2013-01-02  1.0  3  train  foo
2  1.0 2013-01-02  1.0  3   test  foo
3  1.0 2013-01-02  1.0  3  train  foo
```

* A : 모든 행에 같은 실수 1.0을 채움

* B : 모든 행에 같은 날짜/시간 값, pandas의 Timestamp 타입을 사용

* C : 4행짜리 Series 생성, 값은 1, 데이터 타입은 float32

* D : 4행짜리 NumPy배열, 값은 3, 데이터 타입은 int32

* E : 범주형 데이터(Categorical)

* F : 모든 행에 같은 문자열 "foo"


결과 DataFrame의 Column별 데이터 타입입니다.

```
[In]
print(df2.dtypes)

[Out] 
A          float64
B    datetime64[s]
C          float32
D            int32
E         category
F           object
dtype: object
```

* A : float64 (기본 실수형)

* B : datatime64[ns] (나노초 정밀도의 날짜/시간형)

* C : float32 (32비트 실수형, Series 생성 시 지정)

* D : int32 (32비트 정수형, NumPy 배열 생성 시 지정)

* E : category (범주형 데이터)

* F : object (문자열 또는 혼합형 데이터 타입)


다음으로 DataFrame.head()와 DataFrame.tail()을 사용하여 DataFrame의 위쪽(top) 행과 아래쪽(bottom) 행을 각각 볼 수 있습니다.

```
[In]
print(df.head())

[Out]
                   A         B         C         D
2013-01-01  0.469112 -0.282863 -1.509059 -1.135632
2013-01-02  1.212112 -0.173215  0.119209 -1.044236
2013-01-03 -0.861849 -2.104569 -0.494929  1.071804
2013-01-04  0.721555 -0.706771 -1.039575  0.271860
2013-01-05 -0.424972  0.567020  0.276232 -1.087401

[In]
print(df.tail(3))

[Out]
                   A         B         C         D
2013-01-04  0.721555 -0.706771 -1.039575  0.271860
2013-01-05 -0.424972  0.567020  0.276232 -1.087401
2013-01-06 -0.673690  0.113648 -1.478427  0.524988
```

분석 초기에 데이터 구조와 값 분포를 빠르게 확인할 때 유용합니다.

지금 사용한 df의 값들은 np.random.randn(6, 4) 때문에 생성된 랜덤 값으로 위 결과 값은 다르게 보일 수 있습니다. (평균 0, 표준편차 1인 실수)


DataFrame의 행 레이블(index) 또는 열 레이블(columns)을 확인하는 방법입니다.

```
[In]
print(df.index)

[Out] 
DatetimeIndex(['2013-01-01', '2013-01-02', '2013-01-03', '2013-01-04',
               '2013-01-05', '2013-01-06'],
              dtype='datetime64[ns]', freq='D')

[In]
print(df.columns)

[Out]
Index(['A', 'B', 'C', 'D'], dtype='object')
```

위와 마찬가지로 지정해준 내역을 볼 수 있습니다.


DataFrame.to_numpy() 메서드를 사용하여 인덱스나 열 레이블 없이 DataFrame의 기본 데이터만 NumPy 배열(array)로 추출 할 수 있습니다.

```
[In]
print(df.to_numpy())

[Out] 
array([[ 0.4691, -0.2829, -1.5091, -1.1356],
       [ 1.2121, -0.1732,  0.1192, -1.0442],
       [-0.8618, -2.1046, -0.4949,  1.0718],
       [ 0.7216, -0.7068, -1.0396,  0.2719],
       [-0.425 ,  0.567 ,  0.2762, -1.0874],
       [-0.6737,  0.1136, -1.4784,  0.525 ]])
```

행 인덱스/열 이름 등은 제외하여 순수 2차원으로 숫자를 배열합니다.

이는 빠른 수치 계산에 유리한 특징이 있습니다.


★ Pandas DataFrame과 NumPy 배열 간의 근본적인 차이점과 DataFrame을 NumPy 배열로 변환할 때(DataFrame.to_numpy()) 발생하는 데이터 타입 처리 및 성능 문제를 살펴 볼 필요가 있습니다.

* NumPy 배열의 특징 : NumPy 배열은 전체 배열에 대해 하나의 데이터 타입을 가집니다.
* Pandas DataFrame 특징 : Pandas DataFrame은 열(column)별로 하나의 데이터 타입을 가집니다.(즉, 각 열은 서로 다른 타입을 가질 수 있습니다.)
* DataFrame.to_numpy() 변환시 : DataFrame.to_numpy()를 호출하면, Pandas는 DataFrame 내 모든 dtype을 담을 수 있는 단 하나의 공통 dtype을 찾습니다.
* object 타입으로 변환 시 : 만약 공통 데이터 타입이 object라면 DataFrame.to_numpy는 데이터 복사하는 과정을 거치는데, 이는 메모리 사용량과 처리 시간을 증가시키므로 성능 저하의 요인이 될 수 있습니다.
* 이러한 문제를 해결하고 데이터 변환의 효율성을 높이기 위해선 DataFrame 내 데이터 타입을 통일시키거나 명확하게 지정하는 것입니다.
  ```
  [In]
  print(df2.dtypes)

  [Out] 
  A          float64
  B    datetime64[s]
  C          float32
  D            int32
  E         category
  F           object
  dtype: object

  [In]
  print(df2.to_numpy())

  [Out]
  array([[1.0, Timestamp('2013-01-02 00:00:00'), 1.0, 3, 'test', 'foo'],
         [1.0, Timestamp('2013-01-02 00:00:00'), 1.0, 3, 'train', 'foo'],
         [1.0, Timestamp('2013-01-02 00:00:00'), 1.0, 3, 'test', 'foo'],
         [1.0, Timestamp('2013-01-02 00:00:00'), 1.0, 3, 'train', 'foo']],
        dtype=object)
  ```
