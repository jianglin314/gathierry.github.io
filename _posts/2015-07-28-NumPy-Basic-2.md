---
layout: post
title: NumPy Tutorial (2)
categories: [data science]
tags: [python, numpy, Python for Data Analysis]
description: Analysis with numpy
---

### where

    In [5]: xarr = np.array([1.1, 1.2, 1.3, 1.4, 1.5])
    In [6]: yarr = xarr + 1
    In [8]: cond = np.array([True, False, True, True, False])
    In [9]: result = np.where(cond, xarr, yarr)
    In [10]: result
    Out[10]: array([ 1.1,  2.2,  1.3,  1.4,  2.5])

```xarr``` and ```yarr``` could also be 2 scalar variables. With ```where``` the matrix in the condition will be tested. Replace the elements passed with xarr (or the value at the same position if xarr is a matrix). Replace the ones failed with elements in yarr.

### Tricks for the boolean array
    
    % use sum to count the positive number
    In [16]: arr = np.random.randn(100)
    In [17]: (arr > 0).sum()
    Out[17]: 50
