---
title:  "Rotate Array (Python 3)"
date:   2019-07-21
categories: Algorithm
tags: [Algorithm, Python]
---

Go to [this leetcode link][1] to try.

### Question
```
Given an array, rotate the array to the right by k steps, where k is non-negative.

Example 1:

Input: [1,2,3,4,5,6,7] and k = 3
Output: [5,6,7,1,2,3,4]
Explanation:
rotate 1 steps to the right: [7,1,2,3,4,5,6]
rotate 2 steps to the right: [6,7,1,2,3,4,5]
rotate 3 steps to the right: [5,6,7,1,2,3,4]

Example 2:

Input: [-1,-100,3,99] and k = 2
Output: [3,99,-1,-100]
Explanation:
rotate 1 steps to the right: [99,-1,-100,3]
rotate 2 steps to the right: [3,99,-1,-100]

Note:
- Try to come up as many solutions as you can, there are at least 3 different ways to solve this problem.
- Could you do it in-place with O(1) extra space?
```

### Approach

Let's assume `k` equals to the rotation number, and `n` the length of the given array.

The key ideas are
1. The results of rotating `k`, `k + n`, `k + 2n`, ... will be the same. We don't want to rotate as many times as `k + 100n` when it's exactly the same as rotating `k` times. So we reassign `k` to `k % n`.
2. Rotating `k` to the right means we take the last `k` elements to put them in the front.

### Solution

```python
class Solution:
    def rotate(self, nums: List[int], k: int) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        n = len(nums)
        k = k % n
        nums[:] = nums[n - k : ] + nums[ : n - k]
```

[1]:https://leetcode.com/problems/rotate-array/
