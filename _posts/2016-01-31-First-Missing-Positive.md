---
layout: post
title: First Missing Positive
categories: [Algorithm]
tags: [lintcode, array]
description: Given an unsorted integer array, find the first missing positive integer.
---

Given an unsorted integer array, find the first missing positive integer.

Example:

- Given ``[1,2,0]`` return 3,
- Given ``[3,4,-1,1]`` return 2.

## Solution
For questions with integer arrays, we have to consider 2 special cases:

- empty array
- repeated elements

### First approach
This appoach need to sort the integer array at first so that the time complexity is $$O(nlogn)$$

    class Solution:
        # @param A, a list of integers
        # @return an integer
        def firstMissingPositive(self, A):
            A.sort()
            curr = 1
            for ele in A:
                if ele >= 0:
                    if ele == curr:
                        curr = curr + 1
                    elif ele < curr:
                        continue
                    else:
                        return curr
            return curr

### Second approach
To make the algorithm run in O(n) time and uses constant space, we got a second approach

    class Solution:
        # @param A, a list of integers
        # @return an integer
        def firstMissingPositive(self, A):
            idx = 0
            while idx != len(A):
                ele = A[idx]
                if ele > 0 and ele < len(A):
                    # ignore repeated element 
                    if ele != A[ele - 1]:
                        A[idx] = A[ele - 1]
                        A[ele - 1] = ele
                        # the element swapped back should still be considered
                        if idx != ele - 1:
                            idx = idx - 1
                idx = idx + 1
            for idx in range(0, len(A)):
                if idx != A[idx] - 1:
                    return idx+1
            return len(A)+1
