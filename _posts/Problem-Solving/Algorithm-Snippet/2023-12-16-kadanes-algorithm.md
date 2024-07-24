---
title: "[Algorithm] Kadane’s Algorithm: 연속 부분 수열의 최대 합 구하기 (+ DP의 Space Complexity 최적화하기)"
date: 2023-12-16 19:50:00 +0900
categories: [Problem Solving, Algorithm Snippet]
tags: [algorithm, python, dp, optimization, subarray]
math: true
---

## Maximum Subarray Problem

Maximum Subarray Problem이란, **주어진 수열에 대해서 연속 부분 수열의 최대 합**을 구하는 문제이다.

예제로 LeetCode [**53. Maximum Subarray**](https://leetcode.com/problems/maximum-subarray/) 문제의 첫 번째 예시를 살펴보면 다음과 같이 수열이 주어진다. 이때 maximum subarray는 노란색으로 하이라이트한 부분이 되며, 최대 합은 6이 된다.

![leetcode #53](/assets/img/posts/Problem-Solving/Algorithm/23-12-16-01.png)
_ref: <https://medium.com/@rsinghal757/kadanes-algorithm-dynamic-programming-how-and-why-does-it-work-3fd8849ed73d>_

<br>

이 문제를 어떻게 풀 수 있을까?

가장 먼저 떠오르는 **brute force** 방식으로 해결한다면 $O(N^2)$의 time complexity를 가지게 된다. 하지만 **dynamic programming**으로 접근하면 time complexity를 $O(N)$으로 개선할 수 있다. 그리고 여기에서 space complexity도 개선하는 방법이 바로 **Kadane’s Algorithm** 이다.

dynamic programming 접근법을 기반으로 Kadane's Algorithm을 적용해보자!

<br>

### [방법 1] Dynamic Programming: $O(N)$ Space

`dp[i]`를 `i` 번째 원소가 포함되는 subarray들의 합 중 가장 큰 값이라고 하면, 다음과 같이 1D DP로 해결할 수 있다.

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        dp = [0] * len(nums)
        dp[0] = nums[0]
        max_sum = nums[0]

        for i in range(1, len(nums)):
            dp[i] = nums[i] + max(dp[i - 1], 0)
            max_sum = max(max_sum, dp[i])
        
        return max_sum
```

이렇게 해결하면 다음과 같은 복잡도를 가지게 된다.

| --- | --- |
| **Time Complexity** | $O(N)$ |
| **Space Complexity** | $O(N)$  (for `dp` list) |

하지만 이 중 space complexity를 살펴보면, `i` 번째 원소를 확인할 때 **필요한 값은 `dp[i - 1]`**, 즉 **이전의 값**뿐이므로 굳이 $O(N)$의 리스트를 가질 필요가 없다는 것을 알 수 있다.

<br>

### [방법 2] <span class="hl">Kadane’s Algorithm</span>: $O(1)$ Space

위에서 살펴본 DP 풀이에서 `dp[i - 1]`을 리스트 `dp`가 아닌 **별도의 변수 `prev`로 트래킹**하도록 바꿔보면 다음과 같다.

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        prev = max_sum = nums[0]

        for i in range(1, len(nums)):
            prev = max(nums[i], nums[i] + prev)
            max_sum = max(max_sum, prev)
        
        return max_sum
```

이렇게 해결하면 다음과 같이 space complexity가 개선될 수 있으며, 이러한 풀이 방법을 **Kadane’s Algorithm**이라고 한다.

| --- | --- |
| **Time Complexity** | $O(N)$ |
| **Space Complexity** | $O(1)$ (only for `prev`) |

<br>

> 개인적인 경험에 의하면 Kadane’s Algorithm에서와 같이 <span class="hl">1D DP 문제를 $O(1)$ space로 개선할 수 있는 경우</span>, 혹은 <span class="hl">2D DP 문제를 $O(N)$ space로 개선할 수 있는 경우</span>가 종종 있었다. DP 문제를 풀 때는 **(1) 각 단계에서 특정 값만 참조하는지**, **(2) 그로 인해 space complexity를 개선할 수 있는지**도 함께 고민해보자!
{: .prompt-tip}

<br>

## Related Question

- <https://leetcode.com/problems/maximum-subarray/>

<br>

## References

- <https://medium.com/@rsinghal757/kadanes-algorithm-dynamic-programming-how-and-why-does-it-work-3fd8849ed73d>
- <https://en.wikipedia.org/wiki/Maximum_subarray_problem#Kadane's_algorithm>
