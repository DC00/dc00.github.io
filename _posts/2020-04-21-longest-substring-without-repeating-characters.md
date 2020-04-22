---
layout: post
title: "Longest Substring Without Repeating Characters"
categories: code
---

***

Given a string, find the length of the longest substring without repeating characters.

**Example 1**

```
Input: "abcabcbb"
Output: 3 
Explanation: The answer is "abc", with the length of 3. 
```

**Example 2**

```
Input: "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3. 
Note that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

Should use a sliding window. `left` and `right` are indices that point to the edges of the window. `chars` is a set of characters in the current window. Whenever `s[right]` encounters a unique character that is not in `chars`, add it to the set. If `s[right]` encounters a duplicate, update `left` until it removes the duplicate character. This way, the window will look at every possible range containing unique characters. Runs in `O(n)` time.

```python
def lengthOfLongestSubstring(self, s: str) -> int:
    longest = 0
    left, right = 0, 0
    chars = set()

    while right < len(s):
        if s[right] not in chars:
            chars.add(s[right])
            right += 1
            longest = max(longest, right - left)
        else:
            chars.remove(s[left])
            left += 1
    return longest
```