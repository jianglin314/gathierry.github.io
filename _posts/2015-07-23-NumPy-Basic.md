---
layout: post
title: NumPy Tutorial
categories: [engineering]
tags: [python, data science]
description: NumPy is the fundamental package for scientific computing with Python. Besides its obvious scientific uses, NumPy can also be used as an efficient multi-dimensional container of generic data. Arbitrary data-types can be defined. This allows NumPy to seamlessly and speedily integrate with a wide variety of databases.
---
## A multidimensional object - ndarray
	In [3]: import numpy as np

### Initiation
- vector column  

		In [4]: vector = [1, 2, 3]
		In [5]: arr_vec = np.array(vector)
		In [6]: arr_vec
		Out[6]: array([1, 2, 3])

- matrix

		In [10]: matrix = [[1, 2, 3], [4, 5, 6]]
		In [11]: arr_mat = np.array(matrix)
		In [12]: arr_mat
		Out[12]: 
		array([[1, 2, 3],
       		       [4, 5, 6]])
