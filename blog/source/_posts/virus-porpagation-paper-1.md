---
title: Effects of Social Network Structure on Epidemic Disease Spread Dynamics with Application to Ad Hoc Networks
date: 2022-10-05 09:25:00
author: spikeHu
cover: true
coverImg: /images/cover1.jpg
summary: 病毒传播论文
categories: VIRUSPROPAGATION
tags:
  - VIRUS
  - NETWORK
---

# Effects of Social Network Structure on Epidemic Disease Spread Dynamics with Application to Ad Hoc Networks

### Abstract

In this article, we briefly survey the literature on network structure effect on disease spread models.  Subsequently, we discuss an SEIR based epidemic model for different levels of aggregation of population. we propose two security  architectures for these networks based on several design criteria including scalability, power, and computation limitations. 

简单的调查了一下网络结构对于病毒传播的影响，讨论了对于不同聚集程度人群的模型。基于网络的各种安全设计标准包括伸缩性、性能、计算能力限制提出了2种安全的架构。
<!--more-->

## Introduction

一些流行病模型可以用于对移动网络中的病毒传播进行分析，每个设备就相当于一个可以通过通信技术如Wifi，蓝牙，和蜂窝网络的agent与其他设备进行交互。目前主要的对抗措施就是对软件进行更新。但是对于大型网络，进行全局的更新是不太可行的，更好的是对特定的分组和区域进行更新。本文中我们评估了常用的病毒模型和网络拓扑结构的影响，并提供了SEIR模型在流行病学以及Mobile Ad-hoc Networks中的病毒传播中不同聚集程度的有效性评估。

SIR或SEIR模型可以进一步分类为概率模型和确定性模型。在确定性模型中，每一个agent的状态的保持时间都是固定的，这是一种较弱的假设。而概率模型则可以更加准确地抓住传播非确定性这一概念，在这些模型中每个状态的保持时间都是跟随一个已知的概率分布的。在SEIR模型中最重要的因素就是agents之间的相互作用。这一因素可以更加深入的理解社交网络结构如何影响病毒的传播的。

## Related Work

## Agent Based Model

该模型允许可视化网络中病毒传播的详细行为,是一个主要关注详细行为和个体特点的微型模拟模型。

![image-20221006224329676](typora-user-images\image-20221006224329676.png)

将AB模型用与模拟SEIR模型的传播，每个agent一天中可以有3个活动：home ,work, transprotation.基于这些活动，节点可以表示agent, 边表示2个节点参与了一样的活动。

![image-20221006224729196](typora-user-images\image-20221006224729196.png)

A susceptible agent can be infected if it comes in contact with both exposed and infected agents. 

CES：exposed 和susceptible之间的接触率。

CIS：Infected和Susceptible之间的接触率

heterogeneity factor ：两个agent之间连接的强度，表示 被感染的可能性大小。

## Spatial Based Model

该模型以网状结构将人群进行表示， 并考虑到距离因素，所以可以分析病毒的速度和方向。

## Global Population Model

全球人口（GP）模型是一种宏观模拟模型，它代表了整个人口的粗粒流行病动态,GP模型是AB模型的一种泛化，GP模型的感染率和接触率是基于AB模型的。GP不考虑网络结构以及agent的属性。

Eulers’ method can be used to estimate both transmission rate (b) and contact rate (k) for the GP model through the SIR equations listed below [9]: 

![image-20221007092853792](typora-user-images\image-20221007092853792.png)

The AB, SB, and GP models can be presented as an SEIR-based FSM, as shown in Fig. 2. 

<img src="typora-user-images\image-20221007093046909.png" alt="image-20221007093046909" style="zoom: 67%;" />

## Effect of NetWork Topology On The Dynamics of Epidemic

Rahmandad and Sterman [8] explored the eff ect of the AB and GP models on fi ve diff erent types of network structures/topologies: small world, scale-free, random, connected, and lattice.

Low clustered networks:

- 与邻居节点和较远距离节点的接触概率是相等的
- 会有更快的增长率和更早的衰退

High clustered networks:

- contain hubs of connected nodes with sparse connections among hubs
- such as scale free and lattice networks exhibit slower disease spread after initial neighbors are infected since the chance of contacting a susceptible agent declines

Therefore, for high clustered networks, the die out rate of the epidemic is slightly slower than that of a fully connected or random network, as can be seen from Table 2.The major contributor of the diff erence in metrics is the high or low clustering in networks. 

<img src="typora-user-images\image-20221007100457137.png" alt="image-20221007100457137" style="zoom:80%;" />

## Developed SEIR AB Model

SEIR-FSM模型是基于高斯混合模型发展来的。

transmission probability equation of an individual agent :![image-20221007102804213](typora-user-images\image-20221007102804213.png)

M是活动的数量，X是D维的包含agent 人口统计和活动特征的向量。Wi是活动的权重，可由Table 2明确，u是病毒传播率β。

![image-20221007103259974](typora-user-images\image-20221007103259974.png)

β accounts for the heterogeneity factor (δi) based on the diff erent activities of the social network. 

## Experiment And Result For A Synthsized Population



实验所使用的数据集是包含322,000个agent的小世界网络，每个agent有3个活动（home,work,transport），以及相应的人口统计数据。实验将8：00至11：59分为4个时间段，分别表示4个活动：home,transport,work, and transport.对于AB模型，使用Table 1作为参数。

Scenario 1（Fig 3a and 3b）：I0 = 600 and estimated disease parameters b and k are 1.24 and 0.24.

*Note, the GP model closely follows the trend of the AB model for the individuals in both state S and state I.*

Scenario 2（Fig 3c and cd）: I0 = 153 and estimated disease parameters b =0.848 ,k  = 0.099

<img src="typora-user-images\image-20221007152231994.png" alt="image-20221007152231994" style="zoom:80%;" />

场景2的密集程度是小于场景1的，场景1更早的到达峰值，持续时间是10天，场景二持续了13天。由于场景二的estimated disease parameters rate b 和 k要小一些，所以得出结论：These results conclude that both the size and the location of the initial infection play an important role in terms of the dynamics of disease spread. 

## Virus Spread In Mobile Ad Hoc networks 

MANET具有如下特点：一点范围内的设备可以通信，不同范围的设备也可以通过中间节点进行通信，设备可以随时加入或者退出网络，这就导致了MANNET的网络结构是动态变化的。

![image-20221007153617687](typora-user-images\image-20221007153617687.png)

上面的实验结果可以扩展应用到MANET中的病毒传播。

For this information ,we utilize a Traffic Flow Table (TFT) and a Traffic Information Table (TIT) maintained by each mobile device. These tables are used for two purposes. First, they provide connectivity information among nodes. Second, a TFT can be used to identify infected agents through characteristics such as packet drop, buffer overflow, bandwidth consumption, and so on. 

## Disease spread rules In Ad Hoc Mobile Networks

为了将流行病模型应用到MANET，我们利用CVSS（Common Vulnerability Scoring System),CVSS提供了一个开放的框架来描述设备在不同病毒攻击方面的漏洞。[11] P. Mell et al., ”A Complete Guide to the Common Vulnerability Scoring System (CVSS) Version 2.0, Forum of Incident Response and Security Teams,” http://www.first.org/cvss/ cvss-guide.html, June 2007.

Basic CVSS metrics : access distance (access vector), number of times contacted (authenticity),and strength of connectivity (access complexity) at which a mobile agent can be reached. 



The contact rate is generally a function of the access vector and authenticity metrics, that is: 

*Contact rate = f(access vector, authenticity).* 

A high level of authenticity denotes that an agent while trying to connect to another agent needs to go through several steps of authentication.()
认证水平越高代表两个节点的连接就需要更多的认证过程。

Access complexity :与网络的异构系数δ对应。

![image-20221007165036091](typora-user-images\image-20221007165036091.png)

where AV is the access vector, AC is the access complexity, and Au is the authentication level. 