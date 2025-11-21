---
layout: post
title:  "Pandas Basic"
date:   2025-11-21 00:00:00 +0900
categories: Project
---
pandas 공식 홈페이지에서 "10minutes to pandas"를 보고 공부 및 번역한 내용입니다.


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

DataFrame 생성을 합니다.

Numpy 배열과 date_range()를 사용한 Date time index 및 레이블이 지정된 열(labled columns)을 경로하여 DataFrame을 생성하는 방법입니다.

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
