---
title: Depth in UE Material
date: 2023-01-31 10:03:56
tags: [Unreal, Material, Shader]
description: Depth in Unreal Engine
typora-root-url: Depth-in-UE-Material
---



## **Depth Material Expressions**

### **Pixel Depth**（像素深度）

当前所渲染像素的深度，即该像素与摄像机之间的距离。（>=0）

### **Scene Depth**（场景深度）

类似于 PixelDepth，也是返回当前渲染像素与相机的距离。但有以下几点区别：

1. PixelDepth只能在当前所绘制像素处进行深度取样，而SceneDepth可以在任何位置（UV）进行深度取样。  
2. SceneDepth只能在Translucent或Post-Process材质中使用。
3. SceneDepth的采样会忽略透明物体。如下图所示，SceneDepth的采样绕过了人物网格体，受背景对象的影响，而PixelDepth的采样与背景无关。

<center>
    <img src="PixelDepthVSSceneDepth.png" width="45%"> 
    <div> PixelDepth和SceneDepth </div>
</center>


4. 有遮挡物时，SceneDepth返回遮挡物与相机的距离（其值小于CustomDepth，两者相减可以做为遮挡蒙版）。

   <center>
       <img src="SceneDepthTest.png" width="50%"> 
       <div style="padding: 1px;"> SceneDepth测试 </div>
       <br>
   </center>




## Depth in GBuffer

### **Custom Depth**（自定义深度）

用于后处理（Post Processing）材质，返回物体与相机之间的距离（cm）。通过将某些对象渲染到另一个深度缓冲区（自定义深度缓冲区）来屏蔽它们。 这增加了额外的绘制调用，但不使用更多材质。渲染相当便宜，因为只输出深度。**Custom Depth**可以做一些可视化遮挡对象的效果，如绘制对象轮廓，或仅从特定视角可见。使用该特性需要在网格体的细节面板开启**Render Custom Depth Pass**选项（默认状态下，CustomDepth的返回的是一个正无穷大的数值）

<center>
    <img src="EnableCustomDepth.png" width="45%"> 
    <div style="padding: 1px;"> 启用CustomDepth </div>
    <br>
</center>

**测试案例**

可视化遮挡对象（将被遮挡的网格体设置为蓝色）

<center>
    <img src="CustomDepthTest.png" width="50%"> 
    <div style="padding: 1px;"> Custom Depth测试 — 效果 </div>
    <br>
</center>

<center>
    <img src="CustomDepthGraph.png" width="90%"> 
    <div style="padding: 1px;"> Custom Depth测试 — 材质 </div>
    <br>
</center>

### **Custom Depth Stencil**（自定义深度模具）

用于后处理材质，是Custom Depth的扩展。可以通过**CustomDepth Stencil Value**值在材质中区分不同的网格体对象。  
**使用自定义Custom Depth Stencil的前置设置**   

* 开启项目设置里的**Custom Depth-Stencil Pass**选项
* 开启网格体的**Render Custom Depth Pass**选项
* 设置网格体的**CustomDepth Stencil Value** （支持 0~255的数值）

<center>
    <img src="EnableDepthStencil.png" width="98%"> 
    <div style="padding: 1px;"> 启用Depth Stencil </div>
    <br>
</center>

**测试案例**

通过Depth Stencil值区分不同Actor，左边小球的Depth Stencil值为10（设置为蓝色），右边为20（设置为红色）

<center>
    <img src="DepthStencilTest.png" width="50%"> 
    <div style="padding: 1px;"> Custom Depth Stencil测试 — 效果 </div>
    <br>
</center>

<center>
    <img src="CustomDepthStencilGraph.png" width="90%"> 
    <div style="padding: 1px;"> Custom Depth Stencil测试 — 材质 </div>
    <br>
</center>
### **Custom Scene Depth**

在SceneDepth的值是一样的

## 参考

[Depth Material Expressions](https://docs.unrealengine.com/5.1/en-US/depth-material-expressions-in-unreal-engine/?utm_source=editor&utm_medium=docs&utm_campaign=rich_tooltips#scenedepth)  
[后期处理材质](https://docs.unrealengine.com/5.1/zh-CN/post-process-materials-in-unreal-engine/)    
[Custom Depth and Custom Depth Stencil in UE4](https://superyateam.com/2019/06/17/custom-depth-and-custom-depth-stencil-in-ue4/)  
