---
layout: post
title: 电商大数据应用——用户画像建模
categories: [data science]
tags: [user portrait, data analysis]
description: 慕课网“电商大数据应用之用户画像”课程的笔记。
---

## 1.客户消费订单表
在对用户画像进行建模时，通过对客户订单消费表的统计分析，可以提取客户标签，了解用户的消费习惯和能力，从而更好的进行营销。客户消费订单表的主要数据来源于：订单表、退货表、用户表、购物车表。

### 订单表的客户标签

- 第一次消费时间：用户什么时候来的
- 最近一次消费时间：用户多久没来了
- 近30天购买次数、金额（含退货/不含退货）：最近消费能力
- 最小、最大消费金额：用于个性化推荐
- 累计消费次数、金额，累计使用优惠券金额：总体消费情况
- 客单价（i.e.每一位顾客平均购买商品金额）和近90天客单价：消费水平
- 常用收货地区，常用支付方式：用于营销

### 退货表的客户标签

- 退货商品数量、金额：用户拒收、退货习惯
- 最近一次退货时间

### 用户表的客户标签

- 学校/家里/单位下单数：用户购物地点习惯
- 上午/下午/晚上下单数：用户购物时间习惯

### 购物车表的客户标签

- 最近30天购物车次数、商品件数、提交商品件数、放弃商品件数：用户购物车使用习惯

## 2.客户购买类目

同样的，根据客户购买类目表，也可以提取客户标签，同时了解类目的购买人群，针对某一类目进行营销。主要数据来源于：订单表、购物车表、类目维表。类目维表将商品分为若干级类别，电商通常有三级类目，不同类目之间的商品不存在交集。

### 订单表+类目维表

- 近30天（90天）购买类目次数、金额：用户最近购买了哪些类目
- 近180天（总共）购买类目次数、金额：用户喜欢哪些类目
- 最后后一次购买类目时间：反应用户流失情况

### 购物车表+类目维表

- 近30天（90天）购物车类目次数、金额：用户可能处在由于阶段，相关类目降价优惠时可以提醒，促成交易。

## 3.客户购买商店表

可以用于了解商店和品牌的购买人群情况，方便针对某一品牌的营销。主要数据来源于：订单表、购物车表、商店表、退货表。商店表包含商店和品牌的名称，用来观察用户购买过哪些品牌。商店与品牌是多对一的关系。

### 购物车表+商店表

- 近30天购物车次数、商品件数，提交、成功交易和放弃的件数：用户最近挑选品牌的情况
- 最后一次购物车时间，提交商品件数：多久没有挑选该品牌

### 订单表+商店表

- 最近90天排除退拒的商品件数、金额、购买订单数、货到付款订单数：用户最近购买的商店和品牌情况
- 最近90天退货、拒收件数或金额：用户最近在该商店退货和拒收的情况
- 最后一次购物时间、商品件数、金额、退货/拒收时间：最后一次购买情况

## 4.客户基本属性表
客户基本属性表，包含客户所填写的标签和推断出的标签，用于了解用户的人口属性基本情况，对用户属性做统计，并且营销。主要数据来源于：用户表、用户调查表，以及模型表例如孕妇模型表、马甲模型表、用户价值模型表等。

### 用户表

基本属性包括：ID、登录名、性别、生日、年龄、地域、邮箱、手机、注册时间、ip地址、登录来源、邀请人、会员积分、客户黑名单等。

由以上信息，可以推算出：星座、城市等级、邮箱运营商、手机运营商和归属地等。

### 用户调查表

除注册信息以外的详细信息，包括：婚姻状况、学历、收入、职业、是否有小孩、有车、手机品牌等。可以通过给予积分奖励用户填写。

### 模型算法

#### 性别模型

即使用户自己填了性别，还是要用算法计算一次。可以分为男(1)，女(0)，未知(-1)。

1. 商品性别得分
2. 根据购买记录计算用户性别得分
3. 训练阈值，根据阈值判断

模型验证方法，跟用户自己填的性别做对比，确定准确度。

#### 用户汽车模型

根据用户购买产品和浏览记录判断用户是否有车，或者是否有买车意向。

#### 疑似马甲标志

如果意思马甲，数据库中会进行标记，减小噪音。

- 多次访问ip地址相同的账号为同一人所有
- 同一手机登录多次的账号为同一人所有
- 收货人手机号相同的账号为同一人所有

#### 手机相关标签

通过App获取手机数据，可以得到一些很有用的信息，比如：

- 最常用手机品牌
- 手机档次
- 有多少不同手机
- 手机更换频率

## 5.客户营销信息表

将用户营销的相关信息放到一张表中方便使用。主要来源：用户表、订单表、活动表、购物车表、客户品类分群模型、用户价值模型。可以从中提取常用联系信息（营销当然要联系）、纠结商品等信息。

### 模型算法

#### 客户品类分群模型

计算用户在各一级品类的购买金额，使用聚类算法分类，如KMeans。

#### 用户活跃模型
可以分为

- 注册未购买
- 活跃（还可细分为低、中、高频）
- 沉睡（近60天无购买，90天有购买）
- 流失（近90天无购买）

#### 用户价值模型
体现用户对网站的价值，对于提高用户留存率非常有用。

使用RFM实现用户价值模型参考指标：

- Recency 最近一次消费时间
- Frequency 近180天消费频率（以180天为例，太远的数据往往没有参考价值）
- Monetary 近180天消费金额

通过对三个指标的打分，确定用户的价值。

## 6.客户活动信息表

根据客户参与活动提取标签，主要数据来源为：订单表、活动表、活动订单表、用户表。从中可以提取用户喜欢活动的类型。

促销类型包括了：用户促销、满减促销、满增促销、打折促销、换购促销、团购促销等。促销敏感度模型，根据用户购买的活动类型订单数和金额，计算各类促销优惠的订单和金额占比，使用聚类方法判断属于哪类人群。

用户偏好包括：店铺偏好、品类偏好、品牌偏好、颜色偏好。

用户指数：购买力指数、“败家”指数、冲动指数

- 购买力高、中、低端模型
	- 从购物车判断，购物车内金额及成功交易比例 - 聚类
	- 或者使用用户客单价 - 聚类

- 败家指数（1-5星）
	- 使用特征商品来识别，例如苹果新品、奢侈品
	- 单个订单金额

- 冲动指数（1-5星）
	- 特征商品（同品类价格较高商品）平均购物车停留时间
	- 结合特征商品购买数量

此外还可以观察用户使用积分和代金券的习惯。

## 7.客户访问信息表

根据客户访问情况提取标签，用于了解用户总体的访问情况。主要数据来源：网页、手机app、手机网页（含微信等应用内嵌网页）上PageView表和view(session)表。

### PV表

- 最近一次PC/App访问日期、操作系统、session、cookie、浏览器、IP地域、操作系统
- （有记录的）第一次PC/App访问日期、操作系统、session、cookie、浏览器、IP地域、操作系统

### View表
- PC/手机端，近7、15、30、60、90、180、365天访问次数
- 近30天PC端访问天数、购买次数、常用浏览器、常用操作系统等
- 近30天的访问时段







