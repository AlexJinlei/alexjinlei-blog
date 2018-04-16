---
title: "Leetcode804: Unique Morse Code Words"
date: 2018-04-15T23:47:57-04:00
lastmod: 2018-04-15T23:47:57-04:00
draft: false
categories: ["Coding Exercise"]
tags: ["leetcode"]
hiddenFromHomePage: true
---

### Question:
International Morse Code defines a standard encoding where each letter is mapped to a series of dots and dashes, as follows: `"a"` maps to `".-"`, `"b"` maps to `"-..."`, `"c"` maps to `"-.-."`, and so on.

For convenience, the full table for the 26 letters of the English alphabet is given below:

<!--more-->

<pre>
[".-","-...","-.-.","-..",".","..-.","--.","....","..",".---","-.-",".-..","--","-.","---",".--.","--.-",".-.","...","-","..-","...-",".--","-..-","-.--","--.."]
</pre>

Now, given a list of words, each word can be written as a concatenation of the Morse code of each letter. For example, `"cab"` can be written as `"-.-.-....-"`, (which is the concatenation `"-.-."` + `"-..."` + `".-"`). We'll call such a concatenation, the transformation of a word.

Return the number of different transformations among all words we have.

### Example:
<pre>
Input: words = ["gin", "zen", "gig", "msg"]
Output: 2

Explanation: 
The transformation of each word is:
"gin" -> "--...-."
"zen" -> "--...-."
"gig" -> "--...--."
"msg" -> "--...--."

There are 2 different transformations, "--...-." and "--...--.".
</pre>

### Note:
The length of words will be at most 100.
Each words[i] will have length in range [1, 12].
words[i] will only consist of lowercase letters.

### Solution:
Runtime: 40 ms. Beats 93.72% of python3 submissions.
```python
class Solution:
    def __init__(self):
        self.encoder = dict(zip([chr(i) for i in range(97, 97+27)],[".-","-...","-.-.","-..",".","..-.","--.","....","..",".---","-.-",".-..","--","-.","---",".--.","--.-",".-.","...","-","..-","...-",".--","-..-","-.--","--.."]))
    def uniqueMorseRepresentations(self, words):
        """
        :type words: List[str]
        :rtype: int
        """
        encoded_set = set([]) # Create an empty set from an empty list.
        for word in words:
            encoded = '' # Initialize encoded as a empty string.
            for letter in word:
                encoded += self.encoder[letter] # Encode each letter in a word, append to encoded string.
            encoded_set.add(encoded) # Add encoded string to set.
        return len(encoded_set)

words = ["gin", "zen", "gig", "msg"]
counter = Solution().uniqueMorseRepresentations(words)
```
