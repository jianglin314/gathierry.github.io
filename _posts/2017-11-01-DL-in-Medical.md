---
layout: post
title: Deep Learning in Medical Imaging Application
categories: [data science]
tags: [computer vision, deep learning, medical image]
description: 
---

_Abstract_  
This report is basically a summary for the paper _A Survey on Deep Learning in Medical Image Analysis_ [Litjens et al.] to introduce some deep learning techniques in medical image analysis. For more details, please refer to the original paper.

## Introduction
The referenced paper is a survey on deep learning applied to medical image analysis including over 300 papers before February 1st, 2017 from PubMed, ArXiv and conferences such as MICCAI, SPIE, ISBI and EMBC. Papers that didn't report results on medical image data or only used standard feed-forward neural networks with handcrafted features are excluded. Figures below shows the distributions of model used, task addressed, imaging modality and anatomical application area in this survey.
<img src="/images/2017-11-01-DL-in-Medical/distribution.png" width="400px"/>

## Deep Learning Uses in Medical Imaging
### 1. Classification
#### 1.1 Image/exam classification
For this application, one or multiple images (an exam) are taken as input and a single diagnostic variable is output (e.g., disease present or not).

- **Multi-class grade assessment of knee osteoarthritis**: This seems like an application on Xray images. But could be used on surgery as well. It demonstrate the possibility and potential to assess osteoarthritis with deep learning method.
  
- **Alzheimer's disease based on Brain MRI**

#### 1.2 Object or lesion classification
Object classification focuses on the classification of a small (previous identified) part of the mdical image.

- **Nodule classification in chest CT**: In fact, deep learning could be applied to any lesion classification problem, not only pulmonary nodule, but also liver lesion for instance.
- **Assess survival in patients suffering from high-grade gliomas**

