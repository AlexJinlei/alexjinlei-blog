---
title: "Leetcode461: Hamming Distance"
date: 2018-04-16T01:39:58-04:00
lastmod: 2018-04-16T01:39:58-04:00
draft: false
categories: ["Coding Exercise"]
tags: ["leetcode"]
hiddenFromHomePage: true
weight:
---

### Question:
The [Hamming distance](https://en.wikipedia.org/wiki/Hamming_distance) between two integers is the number of positions at which the corresponding bits are different.

Given two integers `x` and `y`, calculate the Hamming distance.

### Note:
0 ≤ `x`, `y` < 2<sup>31</sup>.

### Example:
<pre>
Input: x = 1, y = 4
Output: 2

Explanation:
1   (0 0 0 1)
4   (0 1 0 0)
       ↑   ↑
</pre>

The above arrows point to positions where the corresponding bits are different.

### Solution1:
Runtime: 40 ms. Beats 91.25 % of python3 submissions.
```python
# python3
class Solution:
    def count1s(self, n):
        cnt = 0 # Initialize counter cnt to 0.
        while n:
            cnt += n%2 # Add binary bits value to cnt.
            n = n>>1 # Shift 1 bit to the right.
        return cnt
    def hammingDistance(self, x, y):
        """
        :type x: int
        :type y: int
        :rtype: int
        """
        return self.count1s(x^y) # x^y is x XOR y.

x = 1
y = 4

counter = Solution()
counter.hammingDistance(x,y)
```

### Solution2:
Runtime: 40 ms. Beats 91.25 % of python3 submissions.
```python
# python3
class Solution:
    def hammingDistance(self, x, y):
        """
        :type x: int
        :type y: int
        :rtype: int
        """
        return bin(x^y).count("1")

x = 1
y = 4

counter = Solution()
counter.hammingDistance(x,y)
```

### Solution3:
Runtime: 32 ms. Beats 100.00 % of python3 submissions.
```python
# python3
class Solution(object):
    def hammingDistance(self, x, y):
        """
        :type x: int
        :type y: int
        :rtype: int
        """
        xor = x ^ y
        count = 0
        for _ in range(32):
            count += xor & 1
            xor = xor >> 1
        return count
```
