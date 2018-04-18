---
title: "Leetcode001 Two Sum"
date: 2018-04-18T00:43:38-04:00
lastmod: 2018-04-18T00:43:38-04:00
draft: false
categories: ["Coding Exercise"]
tags: ["leetcode"]
hiddenFromHomePage: true
weight:
---

### Question:
Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

### Example:
<pre>
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
</pre>

### Solution1:
Brute force method. Time complexity is $O(n^2)$.
```python
class Solution:
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        for i in range(len(nums)):
            for j in range(i+1, len(nums)):
                if (nums[i] + nums[j] == target):
                    return [i, j]

nums = [2, 7, 11, 15]
target = 9
result = Solution().twoSum(nums, target)
```
```python
def linkedlist2list(head):
    num = []
    current_ptr = head
    while current_ptr is not None:
        num.append(current_ptr.val)
        current_ptr = current_ptr.next
    return num
```
### Solution2:
Save the calculated results. Time complexity is $O(n)$.
```python
class Solution():
    def twoSum(self, nums, target):
        d = {}
        for i, n in enumerate(nums):
            diff = target - n
            if diff in d:
                return [d[diff], i]
            else:
                d[n] = i
            

nums = [2, 7, 11, 15]
target = 9
result = Solution().twoSum(nums, target)
```

