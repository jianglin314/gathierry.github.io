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
