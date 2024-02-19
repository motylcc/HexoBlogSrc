---
title: UE5 Landscape
date: 2023-01-12 08:56:34
mathjax: true
tags:
description: UE5 Landscape
typora-root-url: UE5-Landscape
---

相关的软件/插件版本： 

* UE - 5.1
* HoudiniEngine - H19.5.432 - Version 2.0
* World Creator 2022.2

<!--more-->

## 世界分区（World Partition）

[世界分区](https://docs.unrealengine.com/5.1/zh-CN/world-partition-in-unreal-engine/)是一种自动数据管理和基于距离进行加载的关卡流送系统，它针对大型世界管理提供了一套完整的解决方案。这个系统将整个世界划分为网格单元，保存在一个固定的关卡，使制作者不再需要划分繁冗的子关卡，并提供一个自动流送系统，基于与流送源之间的距离来加载和卸载这些网格。

与世界分区结合使用的一些功能：

- [一Actor一文件 (One Actor One File）](https://docs.unrealengine.com/5.1/zh-CN/one-file-per-actor-in-unreal-engine)）
- [数据层（Data Layers）](https://docs.unrealengine.com/5.1/zh-CN/world-partition---data-layers-in-unreal-engine)
- [关卡实例（Level Instancing）](https://docs.unrealengine.com/5.1/zh-CN/level-instancing-in-unreal-engine)
- [HLOD](https://docs.unrealengine.com/5.1/zh-CN/world-partition---hierarchical-level-of-detail-in-unreal-engine)

开启世界分区后，在创建地形时会根据"**World Partition Grid Szie**"对地形分块。区别于**Wrold Composition**的分块，**Wrold Partition**不会创建地形子关卡。

<center>
    <img src="LandscapeWithWP.png" width="70%"> 
    <div style="padding: 1px;"> 普通地形（a）,世界分区地形（b） </div>
    <br>
</center>

**创建世界分区地形的方法**：

1. 使用开放世界（Open World）模板创建新关卡。

<center>
    <img src="LevelTemplate.png" width="55%"> 
    <div style="padding: 1px;"> 使用开放世界关卡模板 </div>
    <br>
</center>

2. 转化已有的关卡来使用世界分区（Tools > Convert Level）。

   <center>
       <img src="ToolConvertWP.png" width="70%"> 
       <div style="padding: 1px;"> 将现有关卡转换为World Paritition模式 </div>
       <br>
   </center>
   
   

## 创建地形

### 标准化地形大小

为了在最大化面积的同时最小化地形组件的数量，官方建议的地形大小如下表所示：      

| 整体大小（Overall Size）(顶点) | 四边形/分段（Quads/Section） | 分段 / 组件 （Sections / Component） | 组件大小（Total Size） | 组件总数（Total Components） |
| :----------------------------- | ---------------------------- | ------------------------------------ | ---------------------- | ---------------------------- |
| 8129x8129                      | 127                          | 4 (2x2)                              | 254x254                | 1024 (32x32)                 |
| 4033x4033                      | 63                           | 4 (2x2)                              | 126x126                | 1024 (32x32)                 |
| 2017x2017                      | 63                           | 4 (2x2)                              | 126x126                | 256 (16x16)                  |
| 1009x1009                      | 63                           | 4 (2x2)                              | 126x126                | 64 (8x8)                     |
| 1009x1009                      | 63                           | 1                                    | 63x63                  | 256 (16x16)                  |
| 505x505                        | 63                           | 4 (2x2)                              | 126x126                | 16 (4x4)                     |
| 505x505                        | 63                           | 1                                    | 63x63                  | 64 (8x8)                     |

### 生成方式

1. **UE绘制**  
   使用UE的Landscape模式绘制一个完整的地形

   <center>
       <img src="UECreateLandscapeSetting.png" width="60%"> 
       <div style="padding: 1px;"> UE绘制地形的初始化设置 </div>
       <br>
   </center>

2. 加载高度图生成地形    

   由第三方软件绘制地形后导出高度图（以World Creator为例）  
   高度图格式：  

   * 16位灰阶PNG文件

   * 16位灰阶.RAW文件，以小端字节排序

<center>
    <img src="WorldCreatorInfo.png" width="100%"> 
    <div style="padding: 1px;"> World Creator绘制地形（注意地形的高度范围） </div>
    <br>
</center>
<center>
    <img src="WorldCreatorExportSetting.png" width="40%"> 
    <div style="padding: 1px;"> World Creator导出高度图 </div>
    <br>
</center>
<center>
    <img src="WorldCreatorToUE.png" width="100%"> 
    <div style="padding: 1px;"> UE导入高度图创建地形的相关设置 </div>
    <br>
</center>




### 地形分块的尺寸（World Partition Gird Size）

**World Partition Gird Size** 表示每个地形分块（LandscapeStreamingProxy）的组件数量（默认值为2）。对于一个组件数量为8x8的地形，当**World Partition Gird Size**的数值为2时，地形会被划分成16（4x4）个分块。

<center>
    <img src="WPGridSize.png" width="60%"> 
    <div style="padding: 1px;"> World Partition Grid Size (黄色线框)</div>
    <br>
</center>



### 地形的高度缩放与偏移

<center>
    <img src="LandscapeTransform.png" width="60%"> 
    <div style="padding: 1px;"> 地形的位移与缩放 </div>
    <br>
</center>
UE在计算地形高度时，读取的数值范围在-256到255.992之间，使用的精度为16位。这个数值与地形的Z缩放值相乘得到实际的地形高度范围。如果Z缩放值是100，则最大高度大约是256米，最大深度约-256米。  
由于高度范围总计有512个单位（-256~256），当输入的地形的高度范围（最高值-最低值）不等于512厘米时，需要通过Z轴缩放来获得正确的地形高度。高度缩放值的计算公式如下：
$$
ScaleZ = height * 100 / 512
$$
以高度为309米的山为例，代入公式计算得到的缩放比例为60.351562。

为了匹配地形的实际海拔（最低值可能为负值），高度缩放后的地形还需要修改Z轴的偏移值，计算公式如下：
$$
offsetZ = (Height/2 + MinHeight) * 100
$$
当地形的最低高度为-20米时，代入公式计算得到的Z轴偏移值为13450。

## 更新地形高度

### 地形编辑层（Landscape Edit Layers）

地形编辑层可以给地形添加多个层，每个层都可以独立编辑，实现非破坏性地更新地形高度。可以通过以下两个方式开启地形编辑层：  

* 创建地形时启用Enable Edit Layers  
* 创建地形后，在细节面板开启Enable Edit Layers     

<center>
    <img src="EnableEditLayers.png" width="45%"> 
    <div style="padding: 1px;"> 开启地形编辑层</div>
    <br>
</center>

### 使用高度图更新地形

选择目标编辑层，在Landscape → Sculpt → Layers 找到Heightmap，右键选择重新导入高度图。建议在修改地形大结构时才使用高度图更新地形。小范围的修改可以通过UE Landscape模式的Sculpt工具绘制。

<center>
    <img src="UpdateByHeightmap.png" width="40%"> 
    <div style="padding: 1px;"> 更新高度图修改目标编辑层</div>
    <br>
</center>





### 使用Houdini Engine更新地形

根据需求设计HDA，结合HoudiniEngine插件支持的一些属性可以直接更新地形的目标编辑层。详情参考文档 —— 使用Houdini Engine更新UE5地形。

<center>
    <img src="UpdateCollisionLayers.gif" width="90%"> 
    <div style="padding: 1px;"> 增加/更新地形编辑层 (坠毁层) </div>
    <br>
</center>
<center>
    <img src="UpdateRiverLayers.gif" width="90%"> 
    <div style="padding: 1px;"> 增加/更新地形编辑层 (河流层) </div>
    <br>
</center>


## 洞穴

<center>
    <img src="LandscapeCave.png" width="90%"> 
    <div style="padding: 1px;"> 地形洞穴 </div>
    <br>
</center>

在**地形模式**中可以通过绘制**Visibility**层控制地形上特定位置的可视性和碰撞。具体操作如下：

1. 在地形基材质中，将**Landscape Visibility Mask**节点接入材质属性的**Opacity Mask**。  
   （材质的混合模式不需要设置为**Masked**）

   <center>
       <img src="LandscapeVisibilityMask.png" width="60%"> 
       <div style="padding: 1px;"> Landscape Visibility Mask </div>
       <br>
   </center>

2. 在地形模式（Sculpt）中绘制**Visibility**遮罩

   <center>
       <img src="PaintVisibility.png" width="40%"> 
       <div style="padding: 1px;"> 绘制地形洞遮罩 </div>
       <br>
   </center>

**Tip**：如果洞穴的范围覆盖了一个地形块，可以给目标地形块指定一个独立的洞穴材质。如果没有指定洞穴材质，默认使用地形材质，但会将洞穴所在的地形组件的材质混合模式改为Masked，性能开销只影响当前组件。

<center>
    <img src="SelectHoleMaterial.png" width="55%"> 
    <div style="padding: 1px;"> 指定地形洞穴材质 </div>
    <br>
</center>

<center>
    <img src="CaveShaderComplexity.png" width="55%"> 
    <div style="padding: 1px;"> 材质开销 </div>
    <br>
</center>
## 参考

[地形技术指南](https://docs.unrealengine.com/5.1/zh-CN/landscape-technical-guide-in-unreal-engine/)
[Houdini Engine for Unreal](https://www.sidefx.com/docs/unreal/_landscapes.html)
