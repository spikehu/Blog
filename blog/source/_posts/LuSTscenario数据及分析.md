---
title: LuSTscenario数据及分析
date: 2022-10-11 20:09:02
categories: VirusPropagation
author: spikeHu
tags: 
    -VirusPropagation
    -LuSTscenario
---

# Luxembourg SUMO Traffic (LuST) Scenario

## 数据集场景分析

卢森堡数据集分析，该数据集共有4中场景：

Mobility: shortest path with rerouting.

<!--more-->

- `dua.static.sumocfg` with static traffic lights.
- ` dua.actuated.sumocfg` with actuated traffic lights.

Mobility: Dynamic user equilibrium.

- `sumo -c due.static.sumocfg` with static traffic lights.

- `sumo -c due.actuated.sumocfg` with actuated traffic lights.

  

**static:**表示红绿灯按照固定的规律进行运行，不随着交通状况进行变化

**actuated：**表示红绿灯会根据附近交通状况进行适当调整

 **shortest path with rerouting：**可能是指车辆按照走的最短的路径进行了重新的规划

##### **Dynamic user equilibrium：** *<u>不是很清楚</u>*

### 交通车辆种类以及ID的分析：

#### dua-actuated -- 分析

##### vtypes.add.xml

~~~xml

<routes>
    <vTypeDistribution id="private">
        <vType vClass="passenger" id="passenger1"  color=".8,.2,.2" accel="2.6" decel="4.5" sigma="0.5" length="5.0" minGap="1.5" maxSpeed="70" probability=".40" speedDev="0.1" guiShape="passenger/sedan"/>
        <vType vClass="passenger" id="passenger2a" color=".8,.8,.8" accel="3.0" decel="4.5" sigma="0.5" length="4.5" minGap="1.5" maxSpeed="50" probability=".20" speedDev="0.1" guiShape="passenger/hatchback"/>
        <vType vClass="passenger" id="passenger2b" color=".2,.2,.8" accel="2.8" decel="4.5" sigma="0.5" length="4.5" minGap="1.0" maxSpeed="50" probability=".20" speedDev="0.1" guiShape="passenger/hatchback"/>
        <vType vClass="passenger" id="passenger3"  color=".3,.3,.3" accel="2.7" decel="4.5" sigma="0.5" length="6.0" minGap="1.5" maxSpeed="70" probability=".10" speedDev="0.1" guiShape="passenger/wagon"/>
        <vType vClass="passenger" id="passenger4"  color=".9,.9,.9" accel="2.4" decel="4.5" sigma="0.5" length="5.5" minGap="1.5" maxSpeed="30" probability=".05" speedDev="0.1" guiShape="passenger/van"/>
        <vType vClass="passenger" id="passenger5"  color=".8,.8,.0" accel="2.3" decel="4.5" sigma="0.5" length="7.0" minGap="2.5" maxSpeed="30" probability=".05" speedDev="0.1" guiShape="delivery"/>
    </vTypeDistribution>
	<vType vClass="bus" id="bus" accel="2.6" decel="4.5" sigma="0.5" length="12.0" minGap="3" maxSpeed="30" speedDev="0.1" guiShape="bus">
    <param key="has.rerouting.device" value="false"/>
  </vType>
</routes>
~~~

可以知道大致有两种，一种是私人车辆，一种是公交。

- id="bus" ---表示公交
- id="passenger1"---sedan（轿车）
- id="passenger2a"---hatchback（掀背式汽车）
- id="passenger2b"---hatchback（掀背式汽车）
- id="passenger3"---wagon（也是一种轿车）
- id="passenger4"---van（厢式货车）
- id="passenger5"---delivery（货车）

除了公交，其他的车辆的差别主要是在各种汽车参数上，比如maxSpeed,probability,minGap等等。

##### transit.rou.xml

该文件定义了，从该城市外的地方出发，经过该城市，目的地在该城市外的车辆轨迹。

类型都是passenger类型，

~~~xml
<vehicle id="0DEtoFR.1" type="passenger2b" depart="8.00" departSpeed="13.89">
        <route edges="-31622#0 -31622#0-AddedOffRampEdge -31622#1 -31622#2-AddedOnRampEdge -31622#2 -31622#2-AddedOffRampEdge -31622#3 -31622#4-AddedOnRampEdge -31622#4 -31622#4-AddedOffRampEdge -31622#5 -31622#5-AddedOffRampEdge -31622#6 -31622#7-AddedOnRampEdge -31622#7 -31622#7-AddedOffRampEdge -31622#8 -31622#8-AddedOffRampEdge -30482 -31366#1-AddedOnRampEdge -31366#1 -31366#1-AddedOffRampEdge -31366#2 -31366#3-AddedOnRampEdge -31366#3"/>
    </vehicle>
    <vehicle id="0LUtoBE.0" type="passenger2a" depart="12.00" departSpeed="13.89">
        <route edges="-30706#0 -30706#0-AddedOffRampEdge -31268#0 -31840#0 -31840#1 -31622#7-AddedOnRampEdge -31622#7 -31622#7-AddedOffRampEdge -31622#8 -31622#8-AddedOffRampEdge -31622#9 -31622#10-AddedOnRampEdge -31622#10 -31622#10-AddedOffRampEdge -31622#11 -31622#12-AddedOnRampEdge -31622#12 -31622#12-AddedOffRampEdge -31622#13 -31622#14-AddedOnRampEdge -31622#14 -31622#14-AddedOffRampEdge -31622#15 -31622#16-AddedOnRampEdge -31622#16"/>
    </vehicle>
~~~

id形式分析，id分布如下：

 id="10DEtoFR.0  -----id="0FRtoLU.0" -------------- id="8DEtoFR.41"

表示从高速公路一条道路到另一条高速公路的第几辆车。

##### buslines.rou.xml

该文件对于公交路线的主要定义如下

~~~xml
   <vehicle id="CNA____NBNA_0" type="bus" depart="2.00" departLane="best" departPos="0.00" arrivalPos="-1.00" color="green">
        <route edges="-32278#1 -32278#2 -32278#3 -30738 -31778#3 -31778#4 -31708#0 -31708#1 -32674#4 -32674#5 -32674#6 -32674#7 -32674#8 -32674#9 --31648#5 --31648#4 --31648#3 --31648#2 --31648#1 --31648#0 -30528#5 -30528#6 --32924#4 --32924#3 --32924#2 --32924#1 --32924#0"/>
        <stop busStop="607" duration="20.00"/>
        <stop busStop="58" duration="20.00"/>
        <stop busStop="557" duration="20.00"/>
        <stop busStop="560" duration="20.00"/>
        <stop busStop="217" duration="20.00"/>
        <stop busStop="239" duration="20.00"/>
    </vehicle>
~~~



id的种类大致如下,其中如果序号不一致的主要原因是在于depart的值不一样（NBSKMS_Nightbus除外）：

- CNA_MBNA_k：k是一个变量，从0开始，应该是具体公交车的序号

- AVL---_CN_m_k：m应该是AVL--_CN下的一条公交路线，k是车辆序号

- NBSKMS_Nightbus：属于夜班车，发车时间间隔1000秒到2000左右，一次发2班(从23点左右到凌晨4点半左右)

- AVL---_J1_3类型

- RGM---_114_62类型

  



~~~xml
 <vehicle id="AVL---_CN_4_1" type="bus" depart="36.00" departLane="best" departPos="0.00" arrivalPos="-1.00" color="green">
        <route edges="-31648#1 -31648#2 -31648#3 -32162#0 --30528#0 --31492#1 --31492#0 --30648#15 --30648#14 --30648#13 -32552#0 -30850#0 -30850#1 -30850#2 -30850#3 -30850#4 -30850#5 -30850#6 -30850#7 -30850#8 -30850#9 -31424 --30444#2 --30444#1 --30444#0 --31070#7 --31070#6 --31070#5 --31070#4 --31070#3 --31070#2 --31070#1 --31070#0 -32524#0 -32524#1 -32340#0 -32340#1 -32340#2 -32340#3 -32340#4 -32340#5 -32340#7 -32770#6 -32278#0 -32278#1 -32278#2 -32278#3 -32278#4 -32278#5 -32278#6 --30620 --31068#1 --31068#0 -30528#2 -30528#3 -30812 --31648#1"/>
        <stop busStop="218" duration="20.00"/>
        <stop busStop="561" duration="20.00"/>
        <stop busStop="328" duration="20.00"/>
        <stop busStop="227" duration="20.00"/>
        <stop busStop="572" duration="20.00"/>
        <stop busStop="445" duration="20.00"/>
        <stop busStop="649" duration="20.00"/>
        <stop busStop="34" duration="20.00"/>
        <stop busStop="499" duration="20.00"/>
        <stop busStop="225" duration="20.00"/>
        <stop busStop="171" duration="20.00"/>
        <stop busStop="261" duration="20.00"/>
        <stop busStop="635" duration="20.00"/>
        <stop busStop="455" duration="20.00"/>
        <stop busStop="185" duration="20.00"/>
        <stop busStop="128" duration="20.00"/>
        <stop busStop="607" duration="20.00"/>
        <stop busStop="70" duration="20.00"/>
        <stop busStop="312" duration="20.00"/>
        <stop busStop="278" duration="20.00"/>
        <stop busStop="134" duration="20.00"/>
        <stop busStop="87" duration="20.00"/>
        <stop busStop="621" duration="20.00"/>
        <stop busStop="217" duration="20.00"/>
    </vehicle>
~~~



~~~XML
<vehicle id="AVL---_CN_1_0" type="bus" depart="21.00" departLane="best" departPos="0.00" arrivalPos="-1.00" color="green">
        <route edges="--31272#8 --31272#7 -30892#17 -31320#1 -31320#2 --31272#4 --31272#3 --31272#2 --31272#1 --31272#0 --30528#20 --30528#19 --30528#18 --30528#17 --30528#16 --30528#15 --30528#14 --30528#13 --30528#12 --30528#11 --30528#10 --30528#9 --30528#8 --30528#7 --30528#6 --30528#5 -31648#0 -31648#1 -31648#2 --32672 -31068#0 -31068#1 -32574 -30338 --32652#9 --30796 -31208#1 --31056#0 -31780 --31068#1 --31068#0 -32672 -31648#3 -31648#4 -31648#5 --32674#9 --32674#8 --32674#7 --32674#6 --32674#5 --32674#4 --32674#3 --32674#2 --31134#1 --31134#0 --30856#15 --30856#14 --30856#13 --30856#12 --30856#11 --30856#10 --30856#9 --30856#8 --30856#7 --30856#6 --30856#5 --30856#4 --30856#3 --30856#2 --30856#1 --30856#0 -32674#7 -32674#8 -32674#9 --31492#2 -30528#0 -30528#1 -30528#2 -30528#3 -30812 --31648#1 --31648#0 -30528#5 -30528#6 --32924#4 --32924#3 --32924#2 --32924#1 --32924#0 --32410#3 --32410#2 --32410#1 --32410#0 -30528#16"/>
        <stop busStop="633" duration="20.00"/>
        <stop busStop="523" duration="20.00"/>
        <stop busStop="50" duration="20.00"/>
        <stop busStop="371" duration="20.00"/>
        <stop busStop="457" duration="20.00"/>
        <stop busStop="135" duration="20.00"/>
        <stop busStop="259" duration="20.00"/>
        <stop busStop="218" duration="20.00"/>
        <stop busStop="613" duration="20.00"/>
        <stop busStop="620" duration="20.00"/>
        <stop busStop="79" duration="20.00"/>
        <stop busStop="124" duration="20.00"/>
        <stop busStop="561" duration="20.00"/>
        <stop busStop="556" duration="20.00"/>
        <stop busStop="334" duration="20.00"/>
        <stop busStop="96" duration="20.00"/>
        <stop busStop="192" duration="20.00"/>
        <stop busStop="211" duration="20.00"/>
        <stop busStop="57" duration="20.00"/>
        <stop busStop="410" duration="20.00"/>
        <stop busStop="73" duration="20.00"/>
        <stop busStop="422" duration="20.00"/>
        <stop busStop="155" duration="20.00"/>
        <stop busStop="487" duration="20.00"/>
        <stop busStop="300" duration="20.00"/>
        <stop busStop="354" duration="20.00"/>
        <stop busStop="242" duration="20.00"/>
        <stop busStop="47" duration="20.00"/>
        <stop busStop="515" duration="20.00"/>
        <stop busStop="605" duration="20.00"/>
        <stop busStop="254" duration="20.00"/>
        <stop busStop="462" duration="20.00"/>
        <stop busStop="563" duration="20.00"/>
        <stop busStop="335" duration="20.00"/>
        <stop busStop="557" duration="20.00"/>
        <stop busStop="217" duration="20.00"/>
        <stop busStop="239" duration="20.00"/>
        <stop busStop="187" duration="20.00"/>
        <stop busStop="81" duration="20.00"/>
    </vehicle>
~~~

local.0.rou.xml文件分析

没有分析

##### 总结

可以得知bus的类型有夜间和白天的，夜间的buf的id为NBSKMS_Nightbus开头，白天的有CNA_MBNA_k，AVL---_CN_m_k，类AVL---_J1_3型，类RGM---_114_62型。



















