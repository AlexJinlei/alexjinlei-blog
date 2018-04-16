---
title: "Leetcode771: Jewles and Stones"
date: 2018-04-15T23:21:30-04:00
lastmod: 2018-04-15T23:21:30-04:00
draft: false
categories: ["Coding Exercise"]
tags: ["leetcode"]
hiddenFromHomePage: true
---

### Question:
You're given strings `J` representing the types of stones that are jewels, and `S` representing the stones you have.  Each character in `S` is a type of stone you have.  You want to know how many of the stones you have are also jewels.

The letters in `J` are guaranteed distinct, and all characters in `J` and `S` are letters. Letters are case sensitive, so `"a"` is considered a different type of stone from `"A"`.

### Example 1:
<pre>
Input: J = "aA", S = "aAAbbbb"
Output: 3
</pre>

### Example 2:
<pre>
Input: J = "z", S = "ZZ"
Output: 0
</pre>

### Note:
`S` and `J` will consist of letters and have length at most 50.
The characters in `J` are distinct.

### Solution1: 
Runtime: 44 ms. Beats 91.13% of python3 submissions.
```python
# python3
class Solution:
    def numJewelsInStones(self, J, S):
        """
        :type J: str
        :type S: str
        :rtype: int
        """
        n = 0
        J = list(J)
        S = list(S)
        for stone in S:
            if stone in J:
                n += 1
        return n

J = "aA"
S = "aAAbbbb"
counter = Solution()
counter.numJewelsInStones(J, S)
```
### Solution2: 
Runtime: 40 ms. Beats 99.33% of python3 submissions.
```python
# python3
class Solution:
    def numJewelsInStones(self, J, S):
        """
        :type J: str
        :type S: str
        :rtype: int
        """
        return sum(s in J for s in S)

J = "aA"
S = "aAAbbbb"

counter = Solution()
counter.numJewelsInStones(J, S)
```
### Solution3: 
Runtime: 44 ms. Beats 91.13% of python3 submissions.
```python
# python3
class Solution:
    def numJewelsInStones(self, J, S):
        """
        :type J: str
        :type S: str
        :rtype: int
        """
        return sum(map(J.count, S))

J = "aA"
S = "aAAbbbb"

counter = Solution()
counter.numJewelsInStones(J, S)
```




