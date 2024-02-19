---
title: 使用HoudiniEngine更新UE5地形
date: 2023-01-11 14:05:20
tags:
description: 使用Houdini Engine修改UE5地形
typora-root-url: 使用HoudiniEngine更新UE5地形
---

Houdini Engine for Unreal 可以通过HDA在UE中更新目标地形的编辑层或者创建一个新的地形。    
相关的软件/插件版本： 

* UE - 5.1
* HoudiniEngine - H19.5.432 - Version 2.0

<!--more-->

## 编辑地形层 — Editable Landscape Layers

### HDA输入


HDA无法读取地形编辑层（Landscape Edit Layers），只能将结果输出到指定的地形编辑层 。可以使用Landscape inputs（地形输入）或World Outliner Inputs（世界大纲输入）将地形导入 Houdini  。

* 使用World Outliner Input时，如果选择的是LandscapeStreamingProxy对象，虽然支持多选，但只有一个地块能生效。选择Landscape对象时，整个地块都能生效
* 使用Landscape Input 时， 只能选择一个LandscapeStreamingProxy对象或Landscape对象。

   <center>
       <img src="InputMode.png" width="90%"> 
       <div style="padding: 1px;"> HDA的输入UE地形，Landscape inputs（a）,World Outliner Inputs（b） </div>
       <br>
   </center>

HDA 输入带有地形编辑层的地形时 ，会将最终的高度场和各个编辑层一起导入HoudiniEngine（编辑层以 "Landscapelayer_ 层名字"的方式命名）。在Houdini Engine中提取这些编辑层时，需先将其转成高度场（Height）。
<center>
    <img src="LandscapeToHeightfield.png" width="75%"> 
    <div style="padding: 1px;"> UE Landscape → Houdini Heightfield </div>
    <br>
</center>



### HDA输出

Houdini Engine输出的高度场可以绘制到现有地形上的可编辑地形层，输出方式由下表中的属性控制。

| Attribute Name                   | Owner | Type   | Description                                       |
| :------------------------------- | ----- | ------ | ------------------------------------------------- |
| unreal_landscape_output_mode     | any   | int    | 设置输出模式：生成新的地形 (0) 或者修改编辑层 (1) |
| unreal_landscape_editlayer_name  | any   | string | 目标编辑层的名称                                  |
| unreal_landscape_editlayer_after | any   | string | 该输出层应放置在目标地形的哪个层之后              |

&emsp;&emsp;**unreal_landscape_output_mode**建议设置为1，只修改输入地形的目标编辑层，而非生成一个新的地形。   
&emsp;&emsp;目标编辑图层的名称使用**unreal_landscape_editlayer_name **属性进行设置。如果目标地形编辑图层不存在，则添加一个新的编辑图层（位于顶部）。还可以通过使用**unreal_landscape_editlayer_after**属性指定一个编辑层，将输出层放置在该层之后（上方）。

<center>
    <img src="UpdateEditLayers.png" width="100%"> 
    <div style="padding: 1px;"> 通过HDA修改UE地形的编辑层 </div>
    <br>
</center>
## 遇到的问题

* **HDA添加或者修改目标编辑层后导致地形高度拉伸问题**  
从HDA导入的地形高度场是所有编辑层叠加后的结果，经过HoudiniEngine计算后，如果直接将高度场更新至目标编辑层，则相当于叠加了两个UE地形，导致地形高度被拉伸成两倍基础层的高度。  
**解决方法**：HDA内部处理完目标编辑层后，需要将当前高度场减去基础层，再输出给UE的目标编辑层。

<center>
    <img src="HeightDeform.png" width="95%"> 
    <div style="padding: 1px;"> 解决高度拉伸的问题 </div>
    <br>
</center>

* **修改地形基础层后导致其他编辑层的高度不正确的问题**   

  修改地形基础层后，如果其他编辑层是通过HDA创建或修改的，会导致该层的高度叠加了基础层的修改。
  **解决方法**：HDA输入UE地形后，先清除目标编辑层的高度信息，再进行后续的高度计算。

  <center>
      <img src="UpdateBaseLayerAndRecookHDA.png" width="80%"> 
      <div style="padding: 1px;"> 修改基础层导致其他层高度不正确的问题 </div>
      <br>
  </center>

* **HDA输出的地形边缘丢失高度信息**     

  当HDA Actor的位移或者旋转数据不为零时，会导致输出的地形在Transform偏移的范围内丢失高度信息。

  **解决方法**：关卡中与地形编辑相关的HDA Actor的Transform数值需要归零。

  <center>
      <img src="ResetTransform.png" width="95%"> 
      <div style="padding: 1px;"> HDA Actor的Transform导致地形更新错误 </div>
      <br>
  </center>

## 导致UE崩溃的操作

* **在关卡中使用HDA读取geo缓存生成地形**

  * 地形分块或未分块均会崩溃

  * UE5.0不会

* **没有指定地形的编辑层，直接修改输入的地形**

* **unreal_landscape_output_mode 属性如果设置为0（默认1），HDA Cook后会生成新的地形导致UE崩溃**  

  这些问题应该是UE5.1版本的Bug，经测试UE5.0是能正常使用的。崩溃的原因都是直接修改或创建新的Landscape导致。

