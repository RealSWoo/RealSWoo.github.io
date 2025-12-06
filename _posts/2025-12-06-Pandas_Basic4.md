---
layout: post
title:  "Pandas Basic 병합, 그룹화, 재구조화"
date:   2025-12-05 00:00:00 +0900
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
