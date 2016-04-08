---
layout: post
title: Check power of 2
categories: [Algorithm]
tags: [lintcode, bit manipulation]
description: Using O(1) time to check whether an integer n is a power of 2.
---

Using O(1) time to check whether an integer n is a power of 2

The challenge is to solve this problem within $$O(1)$$ time.

Example:

- For ```n=4```, return true;

- For ```n=5```, return false;

## Solution
For a positive integer $$n$$, which is ```xxxxxx100```, we know that the binary form of $$n-1$$ is ```xxxxxx011```. The first 1 from right turns to 0 and all the 0 on its right side turn to be 1.

To check if an integer is power of 2, we can check if there's only one 1 in its binary form. If it is, for $$n-1$$, the unique 1 turns to be 0 while all the 0 turn to be 1, the result of ```&``` is 0. Otherwise, there must be other 1 and the result of ```&``` isn't zero.

    class Solution:
        """
        @param n: An integer
        @return: True or false
        """
        def checkPowerOf2(self, n):
            # write your code here
            if n == 0:
                return False
            return n & (n - 1) == 0
