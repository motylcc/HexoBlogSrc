---
title: UE5地形生产流程
date: 2023-01-12 08:59:08
tags: 地形
description: 地形制作流程
typora-root-url: UE5地形生产流程
---

具体操作可以参考文档 —— UE5 Landscape

相关的软件/插件版本： 

* UE - 5.1
* HoudiniEngine - H19.5.432 - Version 2.0
* World Creator 2022.2



## 地形的制作流程  

<center>
    <img src="CreateLandscape.png" width="15%"> 
    <div style="padding: 15px;"> Landscape创建流程 </div>
    <br>
</center>

1. **概念美术**

   从美术参考中获取地形的高度信息、地形走势和区域分布

   <center>
       <img src="ArtRef.png" width="90%"> 
       <div style="padding: 1px;"> 美术参考图 </div>
       <br>
   </center>

2. **在World Creator中绘制地形**

   * 地形尺寸参考UE官方提供的地形大小（实际制作可以大于参考值，但输出的高度图必须符合参考尺寸）

   * 绘制完成后需记录地形高度的最小和最大值（用于计算UE Landscape的高度缩放和偏移值）

   * 导出高度图时注意格式和尺寸

   <center>
       <img src="SculptInWorldCreator.png" width="90%"> 
       <div style="padding: 1px;"> World Creator绘制地形 </div>
       <br>
   </center>

3. **在UE中创建Landscape**

   * 启用世界分区

   * 开启地形编辑层（Enable Edit Layers）

   * 根据地形尺寸合理设置世界分区的“Grid Size”（影响地形的分块数量）

   * 计算高度缩放和偏移值（可以在创建地形后再设置）
   
   * 设置地形的组件数量和分段
   
   <center>
       <img src="UELandscapeSetting.png" width="100%"> 
       <div style="padding: 2px;"> UE导入高度图创建地形的相关设置 </div>
       <br>
   </center>

4. **编辑基础层**

   使用高度图生成地形后，根据需要细化编辑地形的基础层

5. **添加地形编辑层**
   
   <center>
       <img src="LandscapeEditLayers.png" width="35%"> 
       <div style="padding: 1px;"> 地形编辑层 </div>
       <br>
   </center>
   可以手动创建新的地形编辑层（一些特殊区域的绘制），或者使用HDA添加地形编辑层（创建大范围的地形结构，如水，道路等）。
   
   注意 HDA Actor的Transform信息需要归零。HDA的输入可以选择“**Landscape Input**”或“**World Outliner Input**”
   
   <center>
       <img src="InputMode.png" width="100%"> 
       <div style="padding: 1px;"> HDA的输入方式 </div>
       <br>
   </center>



## 地形的更新流程

<center>
    <img src="UpdateLandscape.png" width="75%"> 
    <div style="padding: 12px;"> Landscape更新流程 </div>
    <br>
</center>

**地形的更新涉及基础层和其他编辑层的修改**

* **修改基础层**
  
  涉及大结构的调整时，建议在World Creator重新编辑后更新高度图。小范围的精修可以使用UE的Sculpt雕刻。修改基础层后，对于使通过HDA创建或者更新的编辑层需要Recook相应的HDA（重新计算其他层的高度信息）
<center>
    <img src="UpdateBaseLayerAndRecookHDA.png" width="80%"> 
    <div style="padding: 1px;"> 修改基础层后Recook其他编辑层的HDA </div>
    <br>
</center>

* **修改其他编辑层**
  
  * 通过HDA（Recook）
  * 使用UE Sculpt修改



