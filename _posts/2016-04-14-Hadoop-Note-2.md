---
layout: post
title: Hadoop笔记（二）
categories: [data science]
tags: [Hadoop, machine learning, Java]
description: 继上一篇文章，这一次我们在本地Hadoop上实现一个简单的例子。
---

## 编写mapper和reducer
这里我们用python实现一个计算均值和方差的程序。

*mrMeanMapper.py*

    import sys
    from numpy import mat, mean, power
    def read_input(file):
        for line in file:
            yield line.rstrip()

    input = read_input(sys.stdin)
    input = [float(line) for line in input]
    numInputs = len(input)
    input = mat(input)
    sqInput = power(input, 2)
    print "%d\t%f\t%f" % (numInputs, mean(input), mean(sqInput))

    print >> sys.stderr, "report: still alive"
    
*mrMeanReducer.py*

    import sys
    from numpy import mat, mean, power
    def read_input(file):
        for line in file:
            yield line.rstrip()

    input = read_input(sys.stdin)
    mapperOut = [line.split('\t') for line in input]
    cumVal = 0.0
    cumSumSq = 0.0
    cumN = 0.0
    for instance in mapperOut:
        nj = float(instance[0])
        cumN += nj
        cumVal += nj * float(instance[1])
        cumSumSq += nj * float(instance[2])
    mean = cumVal / cumN
    varSum = (cumSumSq - 2 * mean * cumVal + cumN * mean * mean) / cumN
    print "%d\t%f\t%f" % (cumN, mean, varSum)
    print >> sys.stderr, "report: still alive"
    
*inputFile.txt* 是一个多行的文本文件，每一行有一个0到1之间的有理数。

通过

    $cat inputFile.txt | python mrMeanMapper.py | python mrMeanReducer.py
    
测试，如果正常输出

    report: still alive
    100	0.509570	0.084777
    report: still alive
    
则说明程序可以正常运行。这里被打印的inputFile.txt作为输入传入mapper，同理mapper的输出作为输入传入reducer。


## 启动本地Hadoop
如上一篇笔记所描述的那样，我们通过```ssh localhost```和```hstart```在本地启动Hadoop。

## Hadoop实验
下面就可以在Hadoop上进行实验了。

首先将inputFile.txt上传到HDFS中，存成名为*mrmean-i*的文件：

    $hadoop fs -copyFromLocal inputFile.txt mrmean-i

然后通过streaming，在Hadoop上执行python脚本：

    $jar /usr/local/Cellar/hadoop/2.7.2/libexec/share/hadoop/tools/lib/hadoop-streaming-2.7.2.jar -input mrmean-i -output mrmean-o -file mrMeanMapper.py mrMeanReducer.py -mapper "python mrMeanMapper.py" -reducer "python mrMeanReducer.py"
    
这里要注意选项file不能少，输出文件在自动生成的文件夹*mrmean-o*中。通过以下命令可以查看和下载：

    $hadoop fs -cat mrmean-o/part-00000
    $hadoop fs -copyToLocal mrmean-o/part-00000
    
最后执行```hstop```, ```exit```关闭Hadoop并断开ssh连接。

到这里我们就完成了一个简单的mapReduce程序。后面的学习中，将要考虑如何把机器学习的各种算法使用的分布式系统上来。

*参考：  
http://hustlijian.github.io/tutorial/2015/06/19/Hadoop入门使用.html  
机器学习实战 by Peter Harrington*