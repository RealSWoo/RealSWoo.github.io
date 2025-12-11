---
layout: post
title:  "10minutes to Pandas 병합, 그룹화, 재구조화"
date:   2025-12-05 09:00:00 +0900
categories: Pandas
---
pandas 공식 홈페이지에서 "10minutes to pandas"를 보고 공부 및 번역한 내용입니다.

* 해당 코드는 Visual Studio Code에서 실행하였습니다.
* 오타나 오번역 있을 시 말씀해주시면 수정 반영하겠습니다.

&nbsp;

&nbsp;

이번에는 병합을 먼저 알아보겠습니다.

연결(Concat) Pandas는 Series와 DataFrame 객체를 결합하는 기능을 제공합니다.

이는 인덱스에 대한 다양한 집합 논리와 조인/병합 유형 연산의 경우 관계형 대수 기능을 포합합니다.

```ruby
[In]
df = pd.DataFrame(np.random.randn(10, 4))

[In]
print(df)

[Out] 
          0         1         2         3
0 -0.548702  1.467327 -1.015962 -0.483075
1  1.637550 -1.217659 -0.291519 -1.745505
2 -0.263952  0.991460 -0.919069  0.266046
3 -0.709661  1.669052  1.037882 -1.705775
4 -0.919854 -0.042379  1.247642 -0.009920
5  0.290213  0.495767  0.362949  1.548106
6 -1.131345 -0.089329  0.337863 -0.945867
7 -0.932132  1.956030  0.017587 -0.016692
8 -0.575247  0.254161 -1.143704  0.215897
9  1.193555 -0.077118 -0.408530 -0.862495

# break it into pieces
[In]
pieces = [df[:3], df[3:7], df[7:]]

[In]
print(pd.concat(pieces))

[Out] 
          0         1         2         3
0 -0.548702  1.467327 -1.015962 -0.483075
1  1.637550 -1.217659 -0.291519 -1.745505
2 -0.263952  0.991460 -0.919069  0.266046
3 -0.709661  1.669052  1.037882 -1.705775
4 -0.919854 -0.042379  1.247642 -0.009920
5  0.290213  0.495767  0.362949  1.548106
6 -1.131345 -0.089329  0.337863 -0.945867
7 -0.932132  1.956030  0.017587 -0.016692
8 -0.575247  0.254161 -1.143704  0.215897
9  1.193555 -0.077118 -0.408530 -0.862495
```

df[:3]은 처음부터 3번째 위치 직전까지 행을 선택하고 df[3:7]은 인덱스 위치 3부터 7까지 행을 선택하는 것입니다.

df[7:]은 인덱스 위치 7번째부터 끝까지 모든 행을 선택하는거로 최종적으로 pieces라는 Python 리스트에 순서대로 저장됩니다. (처음부터 끝까지)

&nbsp;

merge() 함수는 특정 열을 기준으로 SQL 스타일의 조인(Join)을 가능하게 합니다.

```ruby
[In]
left = pd.DataFrame({"key": ["foo", "foo"], "lval": [1, 2]})

[In]
right = pd.DataFrame({"key": ["foo", "foo"], "rval": [4, 5]})

[In]
print(left)

[Out] 
   key  lval
0  foo     1
1  foo     2

[In]
print(right)

[Out] 
   key  rval
0  foo     4
1  foo     5

In [81]: pd.merge(left, right, on="key")
Out[81]: 
   key  lval  rval
0  foo     1     4
1  foo     1     5
2  foo     2     4
3  foo     2     5
```

merge()의 고유 키를 기준으로 함수를 사용합니다.

```ruby
[In]
left = pd.DataFrame({"key": ["foo", "bar"], "lval": [1, 2]})

[In]
right = pd.DataFrame({"key": ["foo", "bar"], "rval": [4, 5]})

[In]
print(left)

[Out] 
   key  lval
0  foo     1
1  bar     2

In [85]: right
Out[85]: 
   key  rval
0  foo     4
1  bar     5

[In]
print(pd.merge(left, right, on="key"))

[Out] 
   key  lval  rval
0  foo     1     4
1  bar     2     5
```

&nbsp;

다음으로 그룹화에 대해 알아보겠습니다.

'group by'는 아래 방법 중 한가지 이상을 포함하는 프로세스입니다.

* Splitting : 특정 기준에 따라 데이터를 그룹으로 분할
* Applying : 각 그룹에 독립적으로 함수를 적용
* Combining : 결과를 하나의 데이터 구조로 결합

다시, 아래의 group section을 살펴보겠습니다.

```ruby
[In]
df = pd.DataFrame(
    {
        "A": ["foo", "bar", "foo", "bar", "foo", "bar", "foo", "foo"],
        "B": ["one", "one", "two", "three", "two", "two", "one", "three"],
        "C": np.random.randn(8),
        "D": np.random.randn(8),
    }
)
 

[In]
print(df)

[Out] 
     A      B         C         D
0  foo    one  1.346061 -1.577585
1  bar    one  1.511763  0.396823
2  foo    two  1.627081 -0.105381
3  bar  three -0.990582 -0.532532
4  foo    two -0.441652  1.453749
5  bar    two  1.211526  1.208843
6  foo    one  0.268520 -0.080952
7  foo  three  0.024580 -0.264610
```

이제 열 레이블을 기준으로 그룹화하고, 특정 열 레이블들을 선택한 다음 그 결과 그룹들에 DataFrameGroupBy.sum() 함수를 적용합니다.

```ruby
[In]
print(df.groupby("A")[["C", "D"]].sum())

[Out] 
            C         D
A  
bar  1.732707  1.073134
foo  2.824590 -0.574779
```

다중 열 레이블을 기준으로 그룹화를 하면 다중 인덱스(MultiIndex)가 형성됩니다.

```ruby
[In]
print(df.groupby(["A", "B"]).sum())

[Out] 
                  C         D
A   B  
bar one    1.511763  0.396823
    three -0.990582 -0.532532
    two    1.211526  1.208843
foo one    1.614581 -1.658537
    three  0.024580 -0.264610
    two    1.185429  1.348368
```

&nbsp;

다음으로 재구조화(Reshaping)를 알아보겠습니다.

재구조화는 두 가지로 볼 수 있습니다.

첫번째 스택(Stack), 두번째 피봇 테이블(Pivot Table)입니다.

&nbsp;

Stack에 관한 코드는 Pandas의 다중 인덱스를 가진 DataFrame에서 데이터를 긴 형태(Long Form)로 쌓는 stack()과 다시 펼치는 unstack() 과정을 한번에 설명드리려고 합니다.

먼저 DataFrame을 생성합니다.

```ruby
[In]
arrays = [
   ["bar", "bar", "baz", "baz", "foo", "foo", "qux", "qux"],
   ["one", "two", "one", "two", "one", "two", "one", "two"],
]
 

[In]
index = pd.MultiIndex.from_arrays(arrays, names=["first", "second"])

[In]
df = pd.DataFrame(np.random.randn(8, 2), index=index, columns=["A", "B"])

[In]
df2 = df[:4]

[In]
print(df2)

[Out] 
                     A         B
first second  
bar   one    -0.727965 -0.589346
      two     0.339969 -0.693205
baz   one    -0.339355  0.593616
      two     0.884345  1.591431
```

다음으로 stack() 메서드를 통하여 DataFrame의 열 레이블에 있는 하나의 레벨을 stack합니다.

```ruby
[In]
stacked = df2.stack(future_stack=True)

[In]
print(stacked)

[Out] 
first  second   
bar    one     A   -0.727965
               B   -0.589346
       two     A    0.339969
               B   -0.693205
baz    one     A   -0.339355
               B    0.593616
       two     A    0.884345
               B    1.591431
dtype: float64
```

stack한 DataFrame을 unstack하는 것입니다.

기본적으로 unstack은 가장 마지막 레벨을 stack 해제 합니다.

```ruby
[In]
print(stacked.unstack())

[Out] 
                     A         B
first second  
bar   one    -0.727965 -0.589346
      two     0.339969 -0.693205
baz   one    -0.339355  0.593616
      two     0.884345  1.591431

[In]
print(stacked.unstack(1))

[Out] 
second        one       two
first  
bar   A -0.727965  0.339969
      B -0.589346 -0.693205
baz   A -0.339355  0.884345
      B  0.593616  1.591431

[In]
print(stacked.unstack(0))

[Out] 
first          bar       baz
second  
one    A -0.727965 -0.339355
       B -0.589346  0.593616
two    A  0.339969  0.884345
       B -0.693205  1.591431
```

&nbsp;

&nbsp;

여기까지 Pandas의 병합, 그룹화, 재구조화에 대하여 알아보았습니다.

&nbsp;

&nbsp;

감사합니다.
