---
title: 使用WorldCreator更新UE地形
date: 2023-02-07 17:48:45
mathjax: true
tags: [UE, Landscape, World Creator]
typora-root-url: 使用WorldCreator更新UE地形
---

UE的地形可以通过导出高度图进WorldCreator制作地貌细节，完成后再从WorldCreator输出高度图给UE更新地形。

<!--More-->

## 存在的问题

### 尺寸规范问题

#### 地形大小不同

**WorldCreator的地形尺寸是2的n次幂（512, 1024, 2048 ... ），而UE的地形尺寸一般为奇数**（[参考官方文档的地形尺寸建议](https://docs.unrealengine.com/5.1/zh-CN/landscape-technical-guide-in-unreal-engine/)）。

<center>
    <img src="UELandscapeInfo.png" width="40%"> 
    <div style="padding: 15px;"> UE Landscape Info </div>
    <br>
</center>
以上图的UE地形为例，对于1017x1017的尺寸，WorldCreator只能输出1024x1024的高度图。在UE使用1024x1024高度图更新1017x1017的地形会导致边缘高度丢失（迭代多次后，边缘高度信息将大量丢失）

<center>
    <img src="edgeCrop.png" width="70%"> 
    <div style="padding: 15px;"> 地形边缘高度丢失 </div>
    <br>
</center>
#### WorldCreator不支持长宽不等的地形

在WorldCreator中，地形的长宽尺寸是一致的。如果输入的高度图是一个长方形的地形，会被压缩成正方形，导致地形比例失真。

<center>
    <img src="WC_RemapTerrainSize.png" width="80%"> 
    <div style="padding: 15px;"> 长宽不等的地形在WorldCreator中被压缩成正方形 </div>
    <br>
</center>

#### 尺寸匹配的解决方案

**UE输出高度图时匹配WorldCreator的地形尺寸**。   

1. 根据WorldCreator高度图的裁剪方式（以左上角为枢轴点，裁剪右下方的区域），将UE的地形扩展至2的n次幂（延X轴和Y轴的正方向），匹配WorldCreator的地形尺寸

1. 在WorldCreator处理完地形后，导出高度图时将其裁剪为UE地形的尺寸。

<center>
    <img src="WC_HeightmapCrop.png" width="50%"> 
    <div style="padding: 15px;"> WorldCreator高度图裁剪 </div>
    <br>
</center>

<center>
    <img src="SizeResize.png" width="99%"> 
    <div style="padding: 15px;"> 地形扩展与裁剪 </div>
    <br>
</center>


### 高度缩放问题

<center>
    <img src="Heightmap01.png" width="40%"> 
    <div style="padding: 15px;"> Heightmap （16bit, Single Channel） </div>
    <br>
</center>
#### 高度比例不同

对于一张16位的灰度图（在默认缩放值下），UE地形的高度范围是-256米到256米，可以描述高度差在512米以内的地形。WorldCreator的地形范围是-106到10。6米。因此，在默认设置下，WorldCreator不能通过高度图准确地还原UE地形。

<center>
    <img src="HeightRangeDifference.png" width="95%"> 
    <div style="padding: 15px;"> Height Range Difference </div>
    <br>
</center>
<center>
    <img src="HeightErrorInUE.png" width="95%"> 
    <div style="padding: 15px;"> 默认高度比例下UE和World Creator地形高度比较 </div>
    <br>
</center>


#### 视口显示比例不一致

UE在视口中的地形与其尺寸是成正比的，而WorldCreator无法在视口中设置地块的尺寸，只能通过地形精度（Precision）修改输出的高度图尺寸。对于灰度范围相同但尺寸不同的高度图， WC生成的地形是一样的。

<center>
    <img src="ViewPortUE_1K_4k_1.gif" width="90%"> 
    <div style="padding: 15px;"> UE Viewport (1k ~ 4k) </div>
    <br>
</center>

<center>
    <img src="ViewPortWC_4K_1K.gif" width="90%"> 
    <div style="padding: 15px;"> WorldCreator Viewport (1k ~ 4k) </div>
    <br>
</center>

视口比例不一致导致在正确的高度缩放值下，WorldCreator所呈现的地形高度被压缩或拉伸，如下图所示：

<center>
    <img src="WC_DifferentSizeinViewport.png" width="75%"> 
    <div style="padding: 15px;"> 地形尺寸对WorldCreator视口高度的影响 </div>
    <br>
</center>

#### 高度缩放问题的解决方案  

1. 在WorldCreator导入高度图时，根据下面公式修改高度缩放值（Height Scale）还原UE地形的高度比例。
   $$
   HeightScale = 256 / 106 * (1024 / Size) \tag{1} \label{eq1}
   $$
	表1 WorldCreator不同地形尺寸的高度缩放值
   
   | Landscape Size | Height Scale |
   | :------------: | :----------: |
   |  8192 x 8192   |    0.3019    |
   |  4096 x 4096   |    0.6038    |
   |  2048 x 2048   |    1.2075    |
   |  1024 x 1024   |    2.4151    |
   |   512 x 512    |    4.8302    |
   |   256 x 256    |    9.6604    |
   <center>
       <img src="HeightScaleMismatch.png" width="99%"> 
       <div style="padding: 15px;"> 高度缩放匹配地形尺寸 </div>
       <br>
   </center>
   
2. WorldCreator编辑完成后，还原正确的高度缩放值（将高度缩放值修改为2.4151）

3. 在WorldCreator导出高度图时，开启自动归一化选项，将地形高度映射为0~255的灰度范围。

   <center>
       <img src="WC_AutoNormalizeHeightmap.png" width="50%"> 
       <div style="padding: 15px;"> WorldCreator高度图归一化设置 </div>
       <br>
   </center>

6. 在UE导入高度图时，根据WorldCreator的地形高度范围设置Landscape的高度缩放和偏移值。

   <center>
       <img src="WC_GPUInfo.png" width="50%"> 
       <div style="padding: 15px;"> WorldCreator高度范围 </div>
       <br>
   </center>
   高度缩放和偏移的计算公式如下：
   
   $$
   \begin{align}
   ScaleZ =& (MaxHeight-MinHeight) * 100 / 512 \tag{2} \label{eq2}\\
   offsetZ =& ((MaxHeight-MinHeight)/2 + MinHeight) * 100 \tag{3} \label{eq3}
   \end{align}
   $$



## 流程细节

由于UE和WorldCreator的地形存在上述问题，在导入和导出高度图的过程中需要计算地形在不同平台的高度缩放值和偏移值。这些计算可以通过Houdini Engine读取UE地形在后台完成并将结果显示在HDA（heightmap_worldcreator_yh_SOP）的参数面板中，如下图所示：

<center>
    <img src="HDAParameters.png" width="60%"> 
    <div style="padding: 15px;"> HDA参数面板 </div>
    <br>
</center>
* Min Height / Max Height：地形的高度范围
* UE Landscape
  * Offset Z：UE地形的高度偏移值
  * Scale Z：UE地形的高度缩放值
  * Heighmap File：UE高度图的输出路径
  * Save Heightmap：输出UE高度图
* World Creator Terrain
  * Height Scale：WorldCreator 的高度缩放值
  * Precision (Meter)：World Creator 的地形精度

### UE导出高度图

  1. 将HDA（heightmap_worldcreator_yh_SOP）拖入场景中，将Transform数据需要归零。

     <center>
         <img src="HDATransform.png" width="70%"> 
         <div style="padding: 15px;"> HDA Transform </div>
         <br>
     </center>

  2. 设置HDA的输入类型（Landscape Input）并选择目标地形（不能选择地形分块）。

     <center>
         <img src="HDAInput.png" width="50%"> 
         <div style="padding: 15px;"> HDA输入 </div>
         <br>
     </center>

  3. 点击HDA参数面板的**Read Landscape**获取地形数据。

4. 设置HDA参数面板的高度图输出路径**Heightmap File**，然后点击**Save Heightmap**输出高度图。

   **注意**：HDA输出时是以[-256, 256]的高度区间进行归一化灰度图。如果地形的Z轴高度缩放值超过100，即地形高度范围超出-256~256米的范围，则输出高度图时需要重新归一化高度差（避免超过的部分被钳制）。相应地，在World Creator中的高度缩放值（Height Scale）需要根据以下公式重新计算
   $$
   \begin{align}
   HeightScale =& (MaxHeight-MinHeight)  / 212 \tag{4} \label{eq4}
   \end{align}
   $$
   

### WorldCreator导入高度图

<center>
    <img src="WC_CustomShape.png" width="45%"> 
    <div style="padding: 15px;"> WorldCreator导入高度图 </div>
    <br>
</center>
1. 将地形的基础噪声强度设置为0（**GLOBAL** → **BIOMES**** → **BASE SHAPE**** → **Noise Height**）

2. 根据地形尺寸设置地形精度（**GLOBAL** → **Precision**），对应HDA参数面板中的**Precision (Meter)**。

   表2 WorldCreator地形精度与UE Landscape的尺寸关系

   | Landscape Size | Precision   |
   | :------------: |:----------: |
   | 8912 x 8912    | 1 / 8 Meter |
   | 4096 x 4096    | 1 / 4 Meter |
   | 2048 x 2048    | 1 / 2 Meter |
   | 1024 x 1024    | 1 Meter     |
   | 512 x 512      | 2 Meter     |
   | 256 x 256      | 4 Meter     |

3. 将网格细分设置为10（**CUSTOM BASE SHAPE** → **Subdivision**）。

4. 设置高度缩放值（**CUSTOM BASE SHAPE** → **Height Scale**），参考[表1](#高度缩放问题的解决方案)。

5. 导入UE输出的高度图（**CUSTOM BASE SHAPE** → **Import**）。

### WorldCreator导出高度图

参数设置参考下图

* 16 Bit, Single Channel
* Auto Normalization
* 裁剪尺寸（Export Area）

<center>
    <img src="WC_HeightmapOutput.png" width="45%"> 
    <div style="padding: 15px;"> WorldCreator导出高度图 </div>
    <br>
</center>

### UE更新地形

<center>
    <img src="ImportHeightmap_UE.png" width="50%"> 
    <div style="padding: 15px;"> UE导入WorldCreator高度图 </div>
    <br>
</center>
1. 在地形模式下（**Landscape → Manage → Import**），选取WorldCreator输出的高度图，点击**Import**更新地形.

2. 根据HDA参数面板的**Scale Z**和**Offset Z**设置地形的高度缩放值和位移
   * 将地形的缩放值（Transform→Scale.z）设置为**Scale Z**

   * 将地形的Z轴位移（Transform→Location.z）加上**Offset Z**


   **注意**：如果WorldCreator编辑后改变了地形的高度范围，在设置高度缩放和偏移值时不能直接使用HDA的参数。具体计算方式参考公式 \eqref{eq2} 

   <center>
       <img src="UpdateLandscapeTransform.png" width="70%"> 
       <div style="padding: 15px;"> 设置地形的Transform </div>
       <br>
   </center>

## 测试结果

<center>
    <img src="LandscapeUpdated1.png" width="80%"> 
    <div style="padding: 15px;"> 地形测试（1017x1017） </div>
    <br>
</center>
<center>
    <img src="LandscapeUpdated2.png" width="70%"> 
    <div style="padding: 15px;"> 地形测试（1891x1261） </div>
    <br>
</center>



**相关的软件/插件版本**： 

* UE - 5.1
* HoudiniEngine - H19.5.432 - Version 2.0
* World Creator 2022.2

## 参考

1. [World Creator Tutorials](https://www.world-creator.com/tutorials.html)
2. [World Creator Features](https://www.world-creator.com/features.html)
3. [地形技术指南](https://docs.unrealengine.com/5.1/zh-CN/landscape-technical-guide-in-unreal-engine/)
