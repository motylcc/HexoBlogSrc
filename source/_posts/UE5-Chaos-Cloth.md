---
title: UE5 Chaos Cloth
date: 2023-01-17 15:55:09
tags:
description: UE5 Chaos Cloth
typora-root-url: UE5-Chaos-Cloth
---

## Chaos Cloth的制作流程

<center>
    <img src="ChaosClothWorkflow.png" width="90%"> 
    <div style="padding: 1px;"> UE5布料制作流程 </div>
    <br>
</center>

1. **制作布料几何体**（Rig→ FBX）

   * 对于有厚度或模型比较复杂的布料几何体，通常是解算代理几何体（布料简模），再去驱动最终的布料几何体。

   * 给代理几何体分配一个与目标布料不同的材质（UE布料是根据材质区分不同的对象）

     <center>
         <img src="ProxyGeo.png" width="60%"> 
         <div style="padding: 1px;"> 布料几何体及其代理 </div>
         <br>
     </center>

2. **UE导入fbx资产**（FBX → Skeletal Mesh + Physis Asset + Skeleton）

3. **创建布料** （进入SkeletalMesh编辑器面板）

   * 通过材质Section隐藏最终的布料几何体，方便选择代理几何体和观察解算效果

   <center>
       <img src="ProxySection.png" width="90%"> 
       <div style="padding: 1px;"> 根据材质Section启用/禁用几何体 </div>
       <br>
   </center>

   * 选择目标几何体创建布料

   <center>
       <img src="CreateCLoth.png" width="90%"> 
       <div style="padding: 1px;"> 创建布料 </div>
       <br>
   </center>

4. **绘制布料的相关遮罩**（Max Distance, Backstop, Backstop Radius等）

5. **设置布料属性**（Stiffness,  Collision, Iteration等）

   关于Lo（最小值） / Hi（最大值）的设置

   <center>
       <img src="AttributeMask.png" width="60%"> 
       <div style="padding: 1px;"> 绘制属性遮罩控制Lo/Hi值的分布 </div>
   <br>    
   </center>

   如果在Masks中添加了布料属性的遮罩并且启用该遮罩，则 Hi 的值才会生效。绘制值为0对应 Lo的数值，绘制值为1对应Hi的数值。否则，默认只有Lo的值生效。通常，通过Lo值已经足够表达布料的物理特性。

6. **使用代理几何体驱动目标布料**

   移除代理几何体的布料数据，然后在目标几何体上应用代理的布料数据

   <center>
       <img src="ProxyDrive.png" width="90%"> 
       <div style="padding: 1px;"> 代理驱动布料几何体 </div>
       <br>
   </center>
   
   

## key concepts

- 当布料物体的细分较大时，需要相应地提高迭代次数。

- 通过**Backstop**可以有效地修复布料与碰撞体的穿插问题。

- 善用 SkeletalMesh 编辑器的可视化功能去调试或排查问题。

  <center>
      <img src="VisualizationOfCloth.png" width="50%"> 
      <div style="padding: 1px;"> 布料的可视化选项 </div>
      <br>
  </center>

- 对于高度运动的物体，可以适当地提高解算器的迭代次数（Iterations），子步数（subdivisions ）和碰撞体的摩擦力系数（Friction Coefficient）。

- 解算器的迭代次数较低时，可能无法获得比较准确的约束（**stiffness）**和阻尼（**damping**）效果

- Convergence - Until you reach convergence keep stiffness settings at max.

  

## 三个重要的遮罩

- **最大距离（Max Distance）**：布料上的点从其动画位置移动的最远距离。

- **逆止半径（Backstop Radius）**：逆止范围与最大距离（Max Distance）相交时，可防止布料上已绘制的点进入该球体区域。

- **逆止距离（Backstop Distance）**：布料上的点与逆止球表面的距离。

  <center>
      <img src="ChaosBackstop.png" width="70%"> 
      <div style="padding: 1px;"> Backstop </div>
      <br>
  </center>


  在每个布料顶点的后面（沿着法线的反方向）都有一个球形的逆止范围（逆止球），布料顶点到逆止球表面的距离为**逆止距离**。当逆止球与**最大距离**的范围相交时，布料顶点在解算过程中会被阻止进入逆止球的范围。当**逆止距离**设置为0时，则布料的顶点只能沿着法线的正方向运动。合理地设置**逆止半径**和**逆止距离**，可以有效地修复布料与碰撞体的穿插问题。

  <center>
      <img src="BackstopTest.gif" width="90%"> 
      <div style="padding: 1px;"> Backstop测试 </div>
      <br>
  </center>



## 相关指令

* **Stat ChaosCloth**：显示布料性能的相关数据

<center>
    <img src="ClothPerformance.png" width="90%"> 
    <div style="padding: 1px;"> 布料解算的开销统计 </div>
    <br>
</center>

* **p.ClothPhysics 0**：关闭布料模拟，布料几何体将保持在最后一帧的状态

* **p.ClothPhysics 1**：激活布料



## 布料配置说明

<center>
    <img src="ChaosClothConfig.png" width="60%"> 
    <div style="padding: 1px;"> 布料配置的相关参数 </div>
    <br>
</center>

### Mass Properties

**Mass Mode**

* Uniform: Every particle’s mass will be set to the value specified in the UniformMass setting.      

* Total Mass: The total mass is distributed equally over all the particles.     

* Density: A constant mass density is used. (Recommended)

### Material Properties

* **Edge Stiffness**         

  The stiffness of the edge constraints only uses lower than 1 values for very stretchy materials.  Increase the iteration count for stiffer materials.

* **Bending Stiffness**      

  The stiffness of the bending constraints.  Increase the iteration count for stiffer materials   

* **Use Bending Elements**  

  Enable for more accurate bending element constraints instead of the faster cross-edge spring constraints used for controlling bending stiffness. Enabling will allow access to the Buckling Ratio/Stiffness parameters.

* **Buckling**

  This is a feature of the Bending Element constraint (and needs the UseBendingElements ticked in the config to be enabled). It allows the Bending Element constraint to weaken or strengthen for a better control of the cloth material’s creases depending on its two parameters: Buckling Ratio and Buckling Stiffness.

* **Buckling Ratio**

  Once the element has bent such that it's folded more than this ratio from its rest angle ("buckled"), it switches to using the Buckling Stiffness instead of the Bending Stiffness. When Buckling Ratio = 0, the Buckling Stiffness will never be used. When BucklingRatio = 1, the Buckling Stiffness will be used as soon as it is bent past its rest configuration.

* **Buckling Stiffness**

  Bending will use this stiffness instead of Bending Stiffness once the cloth has buckled, i.e., bent beyond a certain angle. Typically, Buckling Stiffness is set to be less than Bending Stiffness. Buckling Ratio determines the switch point between using Bending Stiffness and Buckling Stiffness.

* **Area Stiffness**         

  The stiffness of the area preservation constraints.  Increase the iteration count for stiffer materials.

### Long Range Attachment

* **Tether Stiffness**        

  The tether stiffness of the long-range attachment constraints.  The long-range attachment connects each of the cloth particles to its closest fixed point with a spring constraint.  This can be used to compensate for a lack of stretch resistance when the iterations count is kept low for performance reasons.  Can lead to an unnatural, pull-string puppet-like behavior.  Use 0 to disable.

* **Tether Scale**            

  The limit scale of the long-range attachment constraints (aka tether limit).     

* **Use Geodesic Distance**  

  Use geodesic instead of euclidean distance calculations for the Long Range Attachment constraint, which is slower at setup but more accurate at establishing the correct position and length of the tethers, and therefore is less prone to artifacts during the simulation.

### Collision Properties

* **Collision Thickness**      

  The radius of cloth points when considering collisions against collider shapes.   

* **Friction Coefficient**       

  Friction coefficient for cloth - collider interaction.

* **Use CCD**               

  Use continuous collision detection (CCD) to prevent any missed collisions between fast moving particles and colliders.  This has a negative effect on performance compared to when resolving collisions without using CCD.                           

* **Use Self Collisions**

  Toggle to activate Self Collision Thickness, Friction and Self Intersections (below)

* **Self Collision Thickness**  

  The radius of the spheres used in self collision.  

* **Self Collision Friction**

  Friction coefficient for cloth - cloth interaction.

* **Use Self Intersections**

  Enable the self intersection resolution. This will try to fix any cloth intersections that are not handled by collision repulsions.

* **Use Legacy Backstop** (Not recommended)    

  Retained in case prior setups use it - The main difference is the backstop distance is calculated to the center of the backstop radius. New Backstop is calculated to edge of backstop sphere.                        

### Environmental Properties

* **Damping Coefficient**    

  The amount of damping applied to the cloth velocities.  This Damping Coefficient works differently, as it is global and not at spring level.  It reduces the particles' velocities depending on the group motion. It's similar in a way to drag in a uniform non-moving fluid: the higher the coefficient, the more viscous the fluid.

* **Local Damping Coefficient**  

  The amount of damping for individual deviations of particle velocities from the global motion.  It's more expensive than Damping Coefficient, but also provides better results. It's called Local because it acts in relation to the cloth's center of mass, allowing damp instabilities without causing the cloth to slow down as if it was underwater. Set it to 0 to disable it.

* **Use Point-Based Wind Model**   

  Toggle on for legacy NvCloth setups.

* **Drag**                    

  The drag coefficient applying on each triangle.     

* **Lift**                     

  Applied on each triangle perpendicular to airspeed.  Mass has an effect with wind.

* **Gravity Scale**            

  Scale factor applied to the world gravity and also to the clothing simulation. interactor gravity.  Does not affect the gravity if set using the override below.                         

* **Gravity**                 

  The gravitational acceleration vector [cm/s^2].

* **Pressure** 

  Adds a constant pressure force to each triangle of a simulated cloth. The effect can be modulated across the mesh via the Pressure paintable mask. The pressure force is always applied in the normal direction (use a negative value to push in the opposite direction).   

### Animation Properties

* **Anim Drive Stiffness**     

  The strength of the constraint driving the cloth towards the animated goal mesh.   

* **Anim Drive Damping**     

  The damp strength of the constraint driving the cloth towards the animated goal mesh.   

* **Linear Velocity Scale**     

  The amount of linear velocities sent to the local cloth space from the reference bone.   

* **Angular Velocity Scale**   

  The amount of angular velocities sent to the local cloth space from the reference bone.   

* **Fictitious Angular Scale**  

  The portion of the angular velocity that is used to calculate the strength of all fictitious forces (e.g. centrifugal force). This parameter is only having an effect of the portion of the reference bone’s angular velocity that has been removed from the simulation via the Angular Velocity Scale parameter.  Values range from 0 to 2, with 0 showing no centrifugal effect, 1 full centrifugal effect and 2 an overdriven centrifugal effect.          

### Simulation

* **Iteration Count**（解算器的迭代次数） 

  提高迭代次数可以获得更精确的约束效果，但会增加CPU的开销。                       

* **Max Iteration Count**（最大迭代次数）

  当帧率小于60fps时，解算器的迭代次数被限制在**Max Iteration Count**的值。当帧率大于60fps时，该参数无效。  
  With this parameter, IterationCount becomes the target number of iterations at 60fps. So, if your game currently runs at max 30fps, you will have to double the current value you are using. It will also dynamically increase internally as the framerate comes down, eg: 2 iterations set at 60fps internally becomes 4 at 30fps, 6 at 20fps, ...etc. or 1 at 120fps! MaxIterationCount is used to cap the increase in low framerates, so that you don't end up with 60 iterations at 2fps.

* **Subdivision Count**（解算器的子布数）


  可以提高碰撞和约束的精度，但会提高CPU的消耗（提升该值比增加迭代次数的消耗多50%）。对于高度运动的物体，适当增加子布数可以提高解算的稳定性。



## Reference

[Chaos Cloth Tool Properties Reference](https://dev.epicgames.com/community/learning/tutorials/jOrv/unreal-engine-chaos-cloth-tool-properties-reference)  
[Cloth Troubleshooting and Debugging Tips](https://dev.epicgames.com/community/learning/tutorials/eDwk/unreal-engine-cloth-troubleshooting-and-debugging-tips)  
[官方文档 — 布料工具](https://docs.unrealengine.com/5.1/zh-CN/clothing-tool-in-unreal-engine/)  
