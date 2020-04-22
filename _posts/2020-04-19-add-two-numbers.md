---
layout: post
title: "Add Two Numbers"
categories: code
---

***

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.


**Example**

```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```

The main things to keep track of in this problem and the current place value and if that value has a carry bit. Most basic iterative solution has multiple pointers that go to the end of each list and then check if a given number is longer than the other, summing accordingly.

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def addTwoNumbers(self, l1: ListNode, l2: ListNode) -> ListNode:
        total = ListNode(0)
        carry = 0
        p1 = l1
        p2 = l2
        pt = total
        
        while p1 and p2:
            pv = p1.val + p2.val + carry
            pv, carry = pv % 10, pv // 10
            
            pt.next = ListNode(pv)
            
            p1 = p1.next
            p2 = p2.next
            pt = pt.next
        
        # l1 is longer
        while p1:
            pv = p1.val + carry
            pv, carry = pv % 10, pv // 10
            
            pt.next = ListNode(pv)
            
            p1 = p1.next
            pt = pt.next
        
        # l2 is longer
        while p2:
            pv = p2.val + carry
            pv, carry = pv % 10, pv // 10
            
            pt.next = ListNode(pv)
            
            p2 = p2.next
            pt = pt.next
        
        if carry:
            pt.next = ListNode(carry)
        
        return total.next
```

Simpler solution that consolidates the repeated code I have by checking if the carry has a value throughout the loop. Tracks the place value and carry together and resets the carry after each iteration. You need the dummy node because the next value is set in the while loop. If you set the current pointer equal to a new node, there could be a situation where the last node is a zero, ie `0->7->3->0`

```python
def addTwoNumbers(self, l1, l2):
    dummy = cur = ListNode(0)
    carry = 0
    while l1 or l2 or carry:
        if l1:
            carry += l1.val
            l1 = l1.next
        if l2:
            carry += l2.val
            l2 = l2.next
        cur.next = ListNode(carry%10)
        cur = cur.next
        carry //= 10
    return dummy.next
```

And this solution can handle an arbitrary number of numbers as linked lists

```python
class Solution:
    def addTwoNumbers(self, l1, l2):
        addends = l1, l2
        dummy = end = ListNode(0)
        carry = 0
        while addends or carry:
            carry += sum(a.val for a in addends)
            addends = [a.next for a in addends if a.next]
            end.next = end = ListNode(carry % 10)
            carry /= 10
        return dummy.next
```

All these solutions run in `O(n)` where `n` is the length of the longest number.

Follow up question in an interview could be ***"When could adding numbers in linked lists be useful?"***. If the numbers are huge then they might not be able to be stored in memory and you would need another way to represent them.

But this [wouldn't be a problem](https://stackoverflow.com/a/538583) in Python3 because Python is dope.
