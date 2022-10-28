---
title: Veins实验操作记录
date: 2022-10-11 21:52:16
categories: VirusPropagation
author: spikeHu
tags:
    -VirusPropagation
    -LuSTscenario
---

# Experiment Operation Record

为了早点看到病毒的传播功能是否加入场景，将最早出现的车辆ID为0DEtoFR.0的车辆设置为感染状态，同时为了保证病毒会被传播出去将ID为CNA____NBNA_0的公交也设置为感染状态。

<!--more-->

- antenna.xml中的信号模型进行了修改

​			原来部分

~~~xml
 <AnalogueModel type="SimplePathlossModel" thresholding="true">
            <parameter name="alpha" type="double" value="2.0"/>
        </AnalogueModel>
        <AnalogueModel type="SimpleObstacleShadowing" thresholding="true">
            <obstacles>
                <type id="building" db-per-cut="9" db-per-meter="0.4" />
            </obstacles>
~~~

修改成

~~~xml

    <Antenna type="SampledAntenna1D" id="monopole">
        <!-- monopole antenna on roof -->
        <!-- samples taken from "Effects of Antenna Characteristics and Placements on a Vehicle-to-Vehicle Channel Scenario" -->
        <parameter name="samples" type="string" value="-0.888 -0.942 -1.109 -1.29 -1.543 -1.717 -1.898 -1.902 -1.979 -2.018 -2.18 -2.336 -2.354 -2.287 -2.181 -2.008 -1.837 -1.667 -1.538 -1.553 -1.687 -1.819 -1.921 -1.977 -1.902 -1.768 -1.672 -1.741 -1.888 -2.167 -2.304 -2.326 -2.114 -1.838 -1.53 -1.36 -1.275 -1.331 -1.524 -1.759 -2.046 -2.212 -2.251 -2.04 -1.732 -1.519 -1.476 -1.579 -1.713 -1.775 -1.73 -1.585 -1.423 -1.339 -1.263 -1.433 -1.62 -1.857 -1.973 -2.059 -2.114 -2.097 -1.991 -1.95 -1.865 -1.865 -1.736 -1.606 -1.371 -1.17 -0.986 -0.893"/>
    </Antenna>
    <Antenna type="SampledAntenna1D" id="patch">
  <parameter name="samples" type="string" value="2.0 1.1 -4.0 0.9"/>
  <parameter name="random-offsets" type="string" value="uniform -1 1"/>
  <parameter name="random-rotation" type="string" value="uniform -1 1"/>
</Antenna>
    <Antenna type="SampledAntenna1D" id="panorama">
        <!-- monopole antenna on panorama glass roof -->
        <!-- samples taken from "Influence of car panorama glass roofs on Car2Car communication" -->
        <parameter name="samples" type="string" value="-14.962 -14.531 -14.035 -15.912 -13.103 -11.064 -9.902 -4.728 -6.49 -4.516 -2.66 -0.206 -1.223 2.692 3.219 2.568 3.52 5.896 6.006 6.384 5.405 5.279 5.243 5.433 3.03 2.296 1.664 0.618 1.708 -0.457 1.822 -0.799 1.658 2.735 0.948 0.622 1.156 2.046 1.655 2.611 1.335 -0.108 1.857 0.207 1.221 1.316 2.706 3.575 5.188 7.051 5.599 6.507 7.22 6.805 6.252 6.154 4.16 2.247 3.291 2.866 -1.093 -0.769 -2.331 -4.004 -5.806 -4.79 -10.014 -12.566 -15.903 -14.306 -11.265 -14.368"/>
    </Antenna>
~~~



- 选择车辆进行感染（选择4个角落的）node[4] node[2]  node[6]  node[7],不过得先解决ID问题，可以Nic Id进行识别定位
- node[2] 的话就是nic id = CNA____NBNA_0
- node[4] 对应 nic id = randUni834:1
- node[6] 对应nic id = 0LUtoBE.0
- node[7] 对应 nic id = randUni20481:1
