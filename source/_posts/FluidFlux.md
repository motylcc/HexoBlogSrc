---
title: FluidFlux
date: 2023-03-22 15:24:08
tags:
description: FluidFlux
typora-root-url: FluidFlux
---

# M_FluxSurfaceOver

<center>
    <img src="M_FluxSurfaceOver.png" width="100%"> 
    <div> M_FluxSurfaceOver </div>
</center>

## MF_SurfaceLayer

<center>
    <img src="MF_SurfaceLayer.png" width="100%"> 
    <div> MF_SurfaceLayer </div>
</center>

输出材质属性和透明度（Opacity Mask）

### MF_SurfaceFlux_Mesh

<center>
    <img src="MF_SurfaceFlux_Mesh.png" width="100%"> 
    <div> MF_SurfaceFlux_Mesh </div>
</center>

读取顶点和UV数据，设置材质属性

* R+G：Flow (Velocity)    
* Alpha：Foam Mask

#### MF_SurfaceFluxSet

<center>
    <img src="MF_SurfaceFluxSet.png" width="100%"> 
    <div> MF_SurfaceFluxSet </div>
</center>

#### MF_SurfaceFluxGet

<center>
    <img src="MF_SurfaceFluxGet.png" width="100%"> 
    <div> MF_SurfaceFluxGet </div>
</center>

### MF_WaterlineCorrection

<center>
    <img src="MF_WaterlineCorrection.png" width="100%"> 
    <div> MF_WaterlineCorrection </div>
</center>

修改WPO：将与相机的距离做为Mask，乘以WPO  
效果：当相机靠近水面时，WPO效果减弱

### MF_SurfaceFluxInteraction

<center>
    <img src="MF_SurfaceFluxInteraction.png" width="100%"> 
    <div> MF_SurfaceFluxInteraction </div>
</center>

通过采样一张贴图（法线+高度）叠加到输入的法线和速度方向  


* 修改Flow (Velocity) ：加上**MF_FluxDetailInteraction**输出的法线（N * 0.1)   
* 修改Normal：加上***MF_FluxDetailInteraction**输出的法线（N.z = 0)  
  **MF_FluxDetailInteraction**默认输出的N = (0.5, 0.5, 0.5)

#### MF_FluxDetailInteraction

<center>
    <img src="MF_FluxDetailInteraction.png" width="100%"> 
    <div> MF_FluxDetailInteraction </div>
</center>

输入：World Position (R, G)  ，贴图（InteractionNormalHeight）
输出：  

* UV：WorldPosition * 0.01  (未启用)

* 法线：Tex.RG

* Height：Tex.B

* Foam：Tex.A

  贴图默认为4x4，RGB为灰色，A为1（在材质实例中使用该默认贴图)

##### MF_FluxWorldToInteractionUV

<center>
    <img src="MF_FluxWorldToInteractionUV.png" width="100%"> 
    <div> MF_FluxWorldToInteractionUV </div>
</center>

输入：World Position (R, G)  
输出：

* UV Position (V2)：WorldPosition * 0.01
* UV Size (S) ：未启用

###### MF_FluxInteractionUV

<center>
    <img src="MF_FluxInteractionUV.png" width="100%"> 
    <div> MF_FluxInteractionUV </div>
</center>

输出：  

* Location (V2)：(0.0, 0.0)（default）
* Scale (S)：100（default）
* ScaleInv(S)：0.01（default）

### MF_SurfaceWind

<center>
    <img src="MF_SurfaceWind.png" width="100%"> 
    <div> MF_SurfaceWind </div>
</center>

根据输入的风向更新Flow

### MF_SurfaceFlux_State

<center>
    <img src="MF_SurfaceFlux_State.png" width="100%"> 
    <div> MF_SurfaceFlux_State </div>
</center>

有三个输出（Mask, SurfaceLayer, UV），主要关注Mask （Opacity Mask）  


#### MF_FluxWorldToSimulationUV

<center>
    <img src="MF_FluxWorldToSimulationUV.png" width="100%"> 
    <div> MF_FluxWorldToSimulationUV </div>
</center>

输出UV和Offset (未启用)

UV = (WorldPosition - (1, 0)) / (0, 0) ???  
*MF_FluxSimulationUV的参数WorldToSimulationUV可能是流体模拟时传入*？

##### MF_FluxSimulationUV

<center>
    <img src="MF_FluxSimulationUV.png" width="100%"> 
    <div> MF_FluxSimulationUV </div>
</center>

输出：

* Scale (V2)：默认值为 (0, 0)
* Location (V2)：默认值为 (1, 0)
* Height (S) ：默认值为 0.0
* Resolution (V2)：（未启用）
* TexelSize (V2)： 默认值 (1, 0)

##### MF_FluxVelocity

<center>
    <img src="MF_FluxVelocity.png" width="100%"> 
    <div> MF_FluxVelocity </div>
</center>

主要关注Mask输出（默认由多个MF_FluxSample输出的volume值取最大值，UseExtraMasking参数为False时，Mask值等于一个不加偏移值的MF_FluxSample输出的volume）

###### MF_FluxSample

<center>
    <img src="MF_FluxSample.png" width="100%"> 
    <div> MF_FluxSample </div>
</center>

主要关注Volume输出（FluidVelocityHeightFoam贴图的蓝色通道）

## MF_FluidDetailWave

<center>
    <img src="MF_FluidDetailWave.png" width="100%"> 
    <div> MF_FluidDetailWave </div>
</center>

更新法线和Scattering (Roughness)  
增加法线细节（Tex：DetailWaveNormal）

MF_FluidDetailWaveSample

<center>
    <img src="MF_FluidDetailWaveSample.png" width="100%"> 
    <div> MF_FluidDetailWaveSample </div>
</center>

采样一张细节法线贴图（Tex：DetailWaveNormal），输出法线数据
