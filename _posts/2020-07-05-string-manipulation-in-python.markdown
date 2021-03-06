---
layout: post
title:  "[파이썬 코딩테스트 팁] 문자열 조작"
date:   2020-07-05
tags: [python, coding-test]
---
다음과 같은 문제가 있다고 해보자.
> 길이가 같은 두 문자열 `s1`, `s2`가 주어졌을 때, 다음 조건을 만족하는 문자열 `s`를 리턴하라.
>
> - `s`의 길이는 `s1`, `s2`와 같다.
> - 0 <= `i` < N인 모든 `i`에 대해 (N은 `s1`의 길이)
>   * `s1[i] >= s2[i]` 일 경우 `s[i] = s1[i]`
>   * 그 외의 경우 `s[i] = s2[i]`

이 문제를 푸는 방법으로 다음과 같은 코드를 생각할 수 있다.
```python
def solution(s1: str, s2: str) -> str:
    s = ""
    for i in range(len(s1)):
        if s1[i] >= s2[i]:
            s += s1[i]
        else:
            s += s2[i]
    return s
````
이 풀이는 간단하고 이해하기 쉽지만 비효율적이다. 왜 그럴까?

# 파이썬의 String concatenation
두 문자열을 합치는 것을 string concatenation이라고 한다. 파이썬에서는 이를 `+` 기호를 이용해 직관적이고 간단하게 할 수 있으며, 이렇게 문자열을 쉽게 다룰 수 있는 것이 파이썬의 강점 중 하나이다. 그러나 자료구조에 대한 면밀한 이해가 없으면 string concatenation을 오용하여 비효율적인 코드를 짜게 될 수 있다. 그러므로 string concatenation이 실제로 어떻게 이루어지는지 이해해야 한다.

파이썬에서 string concatenation은 새로운 문자열을 만드는 것이다. 예를 들어 `s = s1 + s2`를 실행할 때, 파이썬은 우선 `s1`을 새 공간에 복사하고 그 뒤의 공간에 이어서 `s2`를 복사한다. 그리고 이 새로운 문자열의 주소값을 `s`에 할당한다. 그러므로 이 연산에 걸리는 시간은 `s1`의 길이 + `s2`의 길이만큼이며, 필요한 공간도 마찬가지로 `s1`의 길이 + `s2`의 길이이다.

다시 위의 풀이를 보자. 매 for문마다 `s += s1[i]` 혹은 `s += s2[i]`가 실행되고 있다. 이는 곧 매 주기마다 문자열 `s`의 복사가 일어난다는 것을 의미한다. `s`의 길이는 1씩 증가하므로 for문이 다 돌때까지 걸리는 시간은 1 + 2 + ... + N = N(N + 1) / 2 이고 시간복잡도는 O(N^2)이다. 마찬가지로 공간복잡도 역시 O(N^2)이다. 직관적으로 보았을 때 선형 시간 안에 풀리는 문제를 다항 시간에 풀고 있으므로 이 풀이는 비효율적이다.

# 해결책: List 사용하기

이 문제의 좀 더 효율적인 풀이는 다음과 같다.

```python
def solution(s1: str, s2: str) -> str:
    s_as_list = []
    for i in range(len(s1)):
        if s1[i] >= s2[i]:
            s_as_list.append(s1[i])
        else:
            s_as_list.append(s2[i])
    return "".join(s_as_list)
````
이 풀이에서는 두 문자열을 순회하며 각 인덱스에서 더 큰 문자를 리스트에 append하고, 마지막에 join을 이용해 문자열을 만들어 리턴한다. 매 for문마다 일어나는 append는 상수 시간 연산이기 때문에 최종적인 시간복잡도는 O(n)이며, 리스트와 마지막에 리턴하는 문자열 외에 다른 자료구조를 사용하지 않기 때문에 공간복잡도도 O(n)이다. Immutable한 객체인 문자열 대신 mutable한 객체인 리스트를 사용함으로서 문자열을 가변적으로 관리하여 효율성을 달성하였다.

# 사고 확장하기: 문자열 내의 문자 교체

어떤 문자열 `s`의 `i`번째 문자를 문자열 `ch`로 교체한다고 해보자. 파이썬에서의 일반적인 방법은 다음과 같다.
```python
s = s[:i] + ch + s[i+1:]
```
이 방법은 string concatenation을 사용하므로 문자열의 복사가 일어난다. 만약 이런 문자 교체가 여러 번 일어나는데 중간값은 필요가 없다면 이 방법은 시간적, 공간적으로 비효율적이다. 반면 파이썬 리스트를 사용한다면 다음과 같이 할 수 있다.
```python
s_as_list = list(s)
s_as_list[i] = ch
```
이렇게 하면 문자 교체가 여러 번 일어나더라도 시간적으로 효율적이며 공간의 낭비가 일어나지 않는다.

또 다른 흔한 상황은 문자열 `s`의 `i`번째와 `j`번째 문자를 교체하는 것이다. String concatenation을 이용한 방법은 다음과 같다.
```python
if i < j:
    s = s[:i] + s[j] + s[i+1:j] + s[i] + s[j+1:]
elif i > j:
    s = s[:j] + s[i] + s[j+1:i] + s[j] + s[i+1:]
```
이 경우는 코드가 비효율적일 뿐만 아니라 길이도 길어지고 if-else문을 사용해야 한다. 반면 리스트를 사용하면
```python
s_as_list = list(s)
(s[i], s[j]) = (s[j], s[i])
```
이렇게 간단하게 할 수 있다.