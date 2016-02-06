---
layout: post
title: 3 Sum closest
categories: [Algorithm]
tags: [lintcode, array]
description: Given an array S of n integers, find three integers in S such that the sum is closest to a given number, target. Return the sum of the three integers.
---

Given an array S of n integers, find three integers in S such that the sum is closest to a given number, target. Return the sum of the three integers.

The challenge is to solve this problem within $$O(n^2)$$ time.

Example:

- Given ``S = {-1 2 1 -4}``, and ``target = 1``, return ``2`` since``-1+2+1=2``.

## Preliminary - 2 sum closest
For a 2 sum closest problem with a sorted array $$a_0, a_1, a_2, ..., a_n$$, we can set 2 indexes at the beginning and the end of the array. If their sum if greater than the target, we'll reduce the $$i_{end}$$ and vice-versa.

The time to find the 2 optimized elements is $$O(n)$$ whereas the time to sort is $$O(nlog(n))$$

## Solution for 3 sum closest
With 3 elements, we choose a priori one element $$a_i$$ out. Then it's equivalent to a 2 sum closest problem with a target $$ = target - a_i$$ and an array from $$a_{i+1}$$ to $$a_n$$. The demonstration is below.

If the sum $$a_{i-j} + a_i + a_{i+k}$$ with $$j, k > 0$$ is the solution, we could find it when $$a_{i-j}$$ is selected a priori.

    class Solution:
        """
        @param numbers: Give an array numbers of n integer
        @param target : An integer
        @return : return the sum of the three integers, the sum closest target.
        """
        def threeSumClosest(self, numbers, target):
            numbers.sort()
            s = sum(numbers[:3])
            diff = abs(target - sum(numbers[:3]))
            for ele in numbers:
                i = numbers.index(ele) + 1
                j = len(numbers) - 1
                while i < j:
                    if diff > abs(target - ele - numbers[i] - numbers[j]):
                        diff = abs(target - ele - numbers[i] - numbers[j])
                        s = ele + numbers[i] + numbers[j]
                    if target - ele - numbers[i] - numbers[j] < 0:
                        j = j - 1
                    elif target - ele - numbers[i] - numbers[j] > 0:
                        i = i + 1
                    else:
                        return ele + numbers[i] + numbers[j]
            return s
            
## Go further - 4 sum closest
A question extended to 4 sum could also be solved in $$O(n^2)$$ time. But it takes more space.

First, build a hashmap to save all the sums for each 2 elements in the array. After that, with a new array, we can solve the problem like a 2 sum closest problem.
