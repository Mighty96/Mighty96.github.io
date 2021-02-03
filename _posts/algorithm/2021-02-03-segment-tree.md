---
title: "세그먼트 트리(Segment Tree)"

categories:
  - Algorithm
tags:
  - 세그먼트 트리

toc: true
toc_sticky: true
---

# 세그먼트 트리

[백준 10868번 : 최솟값](https://www.acmicpc.net/problem/10868)

구간이 주어지면 그 구간내에서 가장 작은 값을 구해야 한다.  
이는 어렵지 않으나, 문제는 최대 10만번을 반복해야 할 수도 있다는 것!

한눈에 봐도 일반적으로 구하는 방법으로는 당연히 안될거라고 생각했다.

문제를 보면서 이런저런 아이디어를 떠올렸다가도, 곰곰히 생각해보면 기존 방법과  
효율에 있어서 별차이가 없는 것 같아서 기각!

결국 검색의 도움을 받으니, `세그먼트 트리`라는 자료구조를 사용해야 한다고 한다.
그래서.... 공부했다!

`세그먼트 트리`는 이진트리 구조를 사용해서 주어진 배열의 부분합들을 나누어 저장하는 자료구조이다.

# 문제에서의 활용

```py
long seg(int start, int end, int node)
{
    if (start == end)
    {
        tree[node] = number[start];
        return tree[node];
    }
    int mid = (start + end) / 2;
    tree[node] = min(seg(start, mid, node * 2), seg(mid + 1, end, node * 2 + 1));
    return tree[node];
}
```

세그먼트 트리는 재귀함수로 구현했다.

1. 만약 `start`와 `end`의 값이 같다면, 즉 구간이 하나라면 그 수를 `node`인덱스에 저장하고 리턴한다.
2. 새로운 `mid`를 `start`와 `end`의 평균값으로 저장한다.
3. 자신의 두 자식을 `start ~ mid`, `mid + 1 ~ end` 로 구성하고, 그 중 최솟값을 자신이 갖고 리턴한다.

즉, 1번 노드를 호출하여 재귀적으로 가장 하단의 노드들까지 호출한 뒤,  
다시 가장 하단부터 값을 저장해가며 쌓아올리는 방식이다.

```py
long find(int start, int end, int node, int left, int right)
{
    if (left > end || right < start) return 1000000001;
    if (start >= left && end <= right) return tree[node];
    int mid = (start + end) / 2;
    return min(find(start, mid, node * 2, left, right),find(mid + 1, end, node * 2 + 1, left, right));
}
```

원하는 값을 찾아낼 때에는 이런 함수를 활용했다.

1. 원하는 구간과 현재 구간이 겹치는 구간이 없다면, 그 노드부터는 더이상 탐색하지 않고 최솟값이 될 수 없는 값을 리턴한다.
2. 만약 현재 구간이 원하는 구간에 포함된다면, 현재 구간의 (저장된)최솟값을 리턴한다.
3. 새로운 `mid`를 `start`와 `end`의 평균값으로 저장한다.
4. 자신의 두 자식 `start ~ mid`, `mid + 1 ~ end` 중 더 작은 값을 리턴한다.

세그먼트 트리를 구현할 때와 마찬가지로, 1번 노드를 호출하여 구간에  
완벽하게 포함되는 노드들 중 가장 상단의 노드들만을 모아서 그 최솟값을 최종리턴한다.

## 테스트케이스 해설

원래는 부분합이 저장되는 것이 맞으나, 문제에 맞추어 구간에서의 최솟값을 각각 저장하였다.

그림으로 표현하면 이렇게 된다.

![세그먼트1](https://user-images.githubusercontent.com/68958979/106721051-811ffb00-6647-11eb-9281-84352d41d45c.png)

위 그림처럼, 가장 상단의 노드에는 전체 값중 최솟값을 저장한다.

그리고 이를 계속 반씩 나누어 이진트리를 구성해나가면서,  
그 구간 내에서의 최솟값을 다시 저장한다.

![세그먼트2](https://user-images.githubusercontent.com/68958979/106721056-81b89180-6647-11eb-8d0e-760cf9e7d37e.png)

실제 문제상의 예시를 세그먼트 트리로 구현하면 이렇게 나타난다.

한번 저장한 세그먼트 트리를 활용할 떄에는, 구하고자 하는 구간 내에 있는 값들 중 최솟값만을 구하면 된다.

![세그먼트3](https://user-images.githubusercontent.com/68958979/106721058-82512800-6647-11eb-834a-288f857044ca.png)

예를들어 3~5번째 수 중 최솟값을 구하고자 한다면, 그림처럼 3번째 수의 정보를 담고있는 9번 인덱스와, 4~5번째 수 중 최솟값의 정보를 담고 있는 5번 인덱스만 비교한다면 우리가 원하는 값을 얻을 수 있다.

비록 예시에서는 단 한번의 계산 과정을 줄였을 뿐이지만, 수가 많아질수록 기하급수적으로 효율이 높아질 것이다.
