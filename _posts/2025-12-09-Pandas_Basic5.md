---
layout: post
title:  "10minutes to Pandas 시계열, 범주, 그래프, 데이터 가져오기/내보내기, 주의사항"
date:   2025-12-09 09:00:00 +0900
categories: Pandas
---
pandas 공식 홈페이지에서 "10minutes to pandas"를 보고 공부 및 번역한 내용입니다.

* 해당 코드는 Visual Studio Code에서 실행하였습니다.
* 오타나 오번역 있을 시 말씀해주시면 수정 반영하겠습니다.

&nbsp;

&nbsp;

시계열(Time series)을 먼저 알아보겠습니다.

시계열 Pandas는 빈도 변환 중 리샘플링 연산을 수행하는 간단하고, 강력하며, 효율적인 기능을 가지고 있습니다.

* 리샘플링은 시계열 데이터의 빈도(시간 간격)를 변경하는 작업입니다.
* 다운샘플링(Downsampling) : 높은 빈도를 낮은 빈도로 변환하는데 일반적으로 구간별 합계나 평균 같은 집계 연산이 필요합니다.
* 업샘플링(Upsampling) : 낮은 빈도를 높은 빈도로 변환하는데 일반적으로 결측값 보간이 필요합니다.

&nbsp;아래 코드는 100초 데이터가 5분 구간 하나에 포함되어 해당 구간의 합계 값(24182)으로 반환하는 코드입니다.

```ruby
[In]
rng = pd.date_range("1/1/2012", periods=100, freq="s")

[In]
ts = pd.Series(np.random.randint(0, 500, len(rng)), index=rng)

[In]
print(ts.resample("5Min").sum())

[Out] 
2012-01-01    24182
Freq: 5min, dtype: int64
```

tz_convert는 시간대 인식 시계열을 다른 시간대로 변환하는 것입니다.

아래 코드는 2012년 3월 6일부터 시작하여 일단위로 5개의 시간 인덱스를 생성하는데 시간대 정보가 없는 ts에 UTC(협적 세계시)를 설정하는 코드입니다.

```ruby
[In]
rng = pd.date_range("3/6/2012 00:00", periods=5, freq="D")

[In]
ts = pd.Series(np.random.randn(len(rng)), rng)

[In]
print(ts)

[Out] 
2012-03-06    1.857704
2012-03-07   -1.193545
2012-03-08    0.677510
2012-03-09   -0.153931
2012-03-10    0.520091
Freq: D, dtype: float64

[In]
ts_utc = ts.tz_localize("UTC")

[In]
print(ts_utc)

[Out] 
2012-03-06 00:00:00+00:00    1.857704
2012-03-07 00:00:00+00:00   -1.193545
2012-03-08 00:00:00+00:00    0.677510
2012-03-09 00:00:00+00:00   -0.153931
2012-03-10 00:00:00+00:00    0.520091
Freq: D, dtype: float64
```

Series.tz_convert() 메서드는 시간대 인식 시계열을 다른 시간대로 변환하는 것입니다.

```ruby
[In]
print(ts_utc.tz_convert("US/Eastern"))

[Out] 
2012-03-05 19:00:00-05:00    1.857704
2012-03-06 19:00:00-05:00   -1.193545
2012-03-07 19:00:00-05:00    0.677510
2012-03-08 19:00:00-05:00   -0.153931
2012-03-09 19:00:00-05:00    0.520091
Freq: D, dtype: float64
```

고정되지 않은 기간(영업일, BusinessDay)을 시계열에 더합니다.

```ruby
[In]
print(rng)

[Out] 
DatetimeIndex(['2012-03-06', '2012-03-07', '2012-03-08', '2012-03-09',
               '2012-03-10'],
              dtype='datetime64[ns]', freq='D')

[In]
print(rng + pd.offsets.BusinessDay(5))

[Out] 
DatetimeIndex(['2012-03-13', '2012-03-14', '2012-03-15', '2012-03-16',
               '2012-03-16'],
              dtype='datetime64[ns]', freq=None)
```

&nbsp;

&nbsp;

범주형 데이터(Categoricals) 처리 분석하는 방법을 알아보겠습니다.

범주형 Pandas는 DataFrame에 범주형 데이터를 포함할 수 있고 범주형 데이터 타입은 문자열이나 정수 같은 데이터를 제한된 수의 고유한 값으로 인코딩하여 저장하는 타입입니다.

아래 코드로 범주형 타입으로 변환 후 확인해보겠습니다.

```ruby
[In]
df = pd.DataFrame(
    {"id": [1, 2, 3, 4, 5, 6], "raw_grade": ["a", "b", "b", "a", "a", "e"]}
)

[In]
df["grade"] = df["raw_grade"].astype("category")

[In]
print(df["grade"])

[Out] 
0    a
1    b
2    b
3    a
4    a
5    e
Name: grade, dtype: category
Categories (3, object): ['a', 'b', 'e']
```

"id"와 "raw_grade"를 포함하는 DataFrame df를 생성하였고, raw_grade 열을 가져와 "category" 메서드를 이용하여 새로운 열 grade에 범주형 타입으로 할당합니다.

grade 열은 dtype: category로 표시되며, 'a', 'b', 'e'를 Categories로 정의했음을 알 수 있습니다.

&nbsp;

카테고리 이름을 변경해보겠습니다.

```ruby
[In]
new_categories = ["very good", "good", "very bad"]

[In]
df["grade"] = df["grade"].cat.rename_categories(new_categories)
```

그리고 카테고리 순서를 재지정하고 동시에 누락된 카테고리들을 추가합니다.

(Series.cat() 하위 메서드는 기본적으로 새로운 Series를 반환합니다.)

```ruby
[In]
df["grade"] = df["grade"].cat.set_categories(
    ["very bad", "bad", "medium", "good", "very good"]
) 

[In]
print(df["grade"])

[Out] 
0    very good
1         good
2         good
3    very good
4    very good
5     very bad
Name: grade, dtype: category
Categories (5, object): ['very bad', 'bad', 'medium', 'good', 'very good']
```

정렬(Sorting)은 카테고리 내의 순서에 따라 이루어집니다.

```ruby
[In]
print(df.sort_values(by="grade"))

[Out] 
   id raw_grade      grade
5   6         e   very bad
1   2         b       good
2   3         b       good
0   1         a  very good
3   4         a  very good
4   5         a  very good
```

obsered=False 메서드를 사용하여 범주형 열을 기준으로 그룹화하면, 비어 있는 카테고리도 같이 표시됩니다.

```ruby
[In]
print(df.groupby("grade", observed=False).size())

[Out] 
grade
very bad     1
bad          0
medium       0
good         2
very good    3
dtype: int64
```

이 기능은 모든 가능한 응답 항목(카테고리)을 보고서에 표시해야 할 때나 데이터가 부족하여 발생하지 않은 그룹도 "0"으로 명시적으로 표현하여 데이터 누락을 방지하고 구조의 일관성을 유지할 때 유용하게 쓸 수 있습니다.

&nbsp;

&nbsp;

다음으로 그래프(시각화)를 알아보겠습니다.

matplotlib API를 참조하여 시각화(Plotting) 기능을 사용하겠습니다.

```ruby
[In]
import matplotlib.pyplot as plt

[In]
plt.close("all")
```

먼저 matplotlib.pyplot을 import 하고 plt로 명명합니다.

plt.close 메서드는 그림(figure) 창을 닫는데 사용합니다.

```ruby
[In]
ts = pd.Series(np.random.randn(1000), index=pd.date_range("1/1/2000", periods=1000))

[In]
ts = ts.cumsum()

[In]
 ts.plot();
```
