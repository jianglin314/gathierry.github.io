---
layout: post
title: Subarray Sum
categories: [Algorithm]
tags: [lintcode, array, hash table]
description: Given an integer array, find a subarray where the sum of numbers is zero. Your code should return the index of the first number and the index of the last number.
---

Given an integer array, find a subarray where the sum of numbers is zero. Your code should return the index of the first number and the index of the last number.

Example:

- Given ``[-3, 1, 2, -3, 4]``, return ``[0, 2]`` or ``[1, 3]``.

## Solution
For an array $$[a_0, a_1, a_2, ..., a_n]$$, we can deduce a new array $$[S_0, S_1, S_2, ..., S_n]$$ with $$S_n = \sum_{i=0}^{n}a_i$$. If $$S_p = S_q$$, the sum $$\sum_{i=p+1}^q a_i= 0$$.

    class Solution:
        """
        @param nums: A list of integers
        @return: A list of integers includes the index of the first number and the index of the last number
        """
        def subarraySum(self, nums):
            l = len(nums)
            dict = {}
            s = 0
            for idx in range(1, l+1):
                s = s + nums[idx - 1]
                if s == 0:
                    return [0,idx-1]
                if s in dict.keys():
                    return [dict[s], idx-1]
                else:
                    dict[s] = idx