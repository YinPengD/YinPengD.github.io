---
title: UE4碰撞解析
date: 2018-11-16 16:33:21
tags: [UE4,碰撞,Collision,Physics]
categories:  UE4
---

在游戏中常见的带有物理的物体一般有5种，胶囊体一类、静态网格物体StaticMesh、骨骼网格物体SkeletalMesh、Landscape地形以及PhysicsVolume（BrushComponent），UE4继承了PhysX物理引擎，来模拟刚体的物理行为，包括碰撞响应，由于碰撞检测所出现的BUG不在少数，我也是在解决类似绝地求生经常有的碰撞之后飞天的BUG中加深对物体碰撞的理解的。

<!--more-->

### 一.任意两个物体碰撞的条件有：

1.两个物体都拥有Collision组件

2.其中有一个物体开启了simulate Physics

### Collision组件

> Collision实际是对碰撞进行监测追踪的组件

#### 属性：

**Simulation Generates Hit Events**（模拟生成碰撞事件）：是否会调用Event Hit(碰撞事件)和On Component Hit(组件碰撞)事件

**Generate Overlap Events**(生成重叠事件)：会调用重叠事件，比如Event Actor Begin Overlap(Actor开始重叠事件)或Event Actor End Overlap(Actor结束处重叠事件)

**Can Character Setp up on** :玩家撞到物体后是否能够走上去

**Collision Presets** :碰撞的预设类型

```
No Collision（无碰撞） - 不同该对象发生碰撞，无论是踪迹碰撞还是物理碰撞。
No Physics Collision（无物理碰撞） - 该刚体仅用于射线投射、扫射及重叠。（4.9之后改为Query Only)
Collision Enabled（启用碰撞） - 该刚体用于物理模拟和碰撞查询。
```

碰撞类型通道有（CollisionChannel)：

```
Camera:摄像机能否穿过次物体
world Static : 静态物体
World Dynamic : 动态物体
Pawn ： 控制器
PhysicsBody : 刚体
Vehicle : 交通工具（只会和WorldStatic、WorldDynamic发生碰撞）
Destructible : 可破坏纹理物体
Projectile : 子弹，抛射物
Spectator只和WorldStatic发生碰撞（因为是观战者，不能干扰比赛，但也不能陷到地底下去啊） 
Trigger只和WorldDynamic发生“碰撞”
```

**USe CCD**：是否针对该对象应用 **Continuous Collision Detection（连续碰撞检测）** 。增加碰撞检测的准确度。

**Always Create Physics State（总是创建物理状态）**： 通过需要时才去创建物理状态（碰撞属性）来减少计算对象的物理状态来提高性能。

**Multi Body Overlap（多个刚体重叠）**：为同一物体的多个刚体生成不同的重叠事件。

**Check Async Scene on Move（移动时检查异步场景）**：如果该项设置为 **真** ，那么组件将在两个物理场景（同步和异步）中都查找碰撞。异步场景主要由可破坏网格物体的破碎块使用。

**Trace Complex on Move（移动时跟踪复杂碰撞）**：如果该项设置为 **真**, 扫过该组件的对象将在运动时跟踪复杂碰撞。复杂碰撞简而言之就是基于每个面的碰撞，而简单碰撞则是您的球体、胶囊体、盒体及生成的凸面体形状。

**Return Material on Move（移动时返回材质）**：设置该项为 **真** 将返回物理材质到 **Hit Info（碰撞信息）** 中

### Physics组件

> 碰撞的发生需要靠Physics simulating(物理模拟)，如果其中一方没有设置物理模拟，那么就算怎么设置Collision也不会发生碰撞的，Collision起到的作用是去监测碰撞的发生。

#### 属性

**Simulate Physics模拟物理效果**：开启物理模拟，碰撞的必要条件之一

**Mass in KG质量 KG为单位**：物体的重量

**Angular Damping角度阻尼**：阻塞旋转的力的大小

**Linear Damping线性运动阻尼**：阻塞移动的力的大小

**Enable Gravity是否启用重力**：启用重力

**Lock Position（锁定位置）**: 锁定某个方向不发生位移

**Lock Rotation（锁定旋转）**：锁定不向某个方向旋转

**Mode（模式）**：不同的限制移动方向

**Auto Weld（自动连接为整体）**: 将会与父控件组成一个整体进行物理模拟

**Start Awake开始时唤醒**：实体是在游戏开始时就唤醒物体模拟，或是一开始处于待唤醒状态

**Center Of Mass Offset重心偏移**：从计算后的位置再增加一个特定的重心偏移

**Mass Scale重力的缩放比例**：每个实例的重力缩放比例

**Max Angular Velocity最大角速度**：实例的最大角速度

**Use Async Scene使用异步的场景**：如果勾选的话，实体将会放在异步的物理场景中。如果不勾选，他就会放到同步的物理场景。如果实体是一个静态实体，无论bUseAsyncScene是什么值，他都会直接被放在两个场景

**Override Max Depenetration Velocity覆盖最大的重叠移动速度**：这个实体实例是否有自己的最大重叠移开速度（脱离碰撞的速度）

**Max Depenetration Velocity最大脱离重叠速度**: 最大脱离重叠速度

**Walkable Slope Override重写在该实体上可行走的坡度**：自定义的可行走坡度

**Walkable Slope Behavior可行走的坡度行为**：表面的行为（就是 是否影响行可走的坡度）

**Walkable Slope Angle可行走的角度**：覆盖可以行走的斜度

**Sleep Family休眠集**：一个决定集合，决定什么时候让实体静止不动（Normal适合大部分情况，不过在某些弧形移动的转折点或者钟摆的速度非常小的地方会导致物体静止。sensitive即使在这两个情况下也不会停止运动。）

**Position Solver Iteration Count位置计算迭代数**：物理实体计算位置的迭代次数。次数越多，对CPU的消耗越大，但是移动动作会越平稳

**Velocity Solver Iteration Count角度计算迭代数**：物理实体计算角度的迭代次数。次数越多，对CPU的消耗越大，但是旋转动作会越平稳

**Should Update Physics Volume是否更新物理体积**：当组件移动时，重叠的缓存物理体积是否被立即更新

### 小技巧：

1.使用编辑视图的Player Collision可以看到玩家的碰撞世界

2.在项目设置中可以添加对象通道和踪迹响应通道

### 注意：

1.父控件的simulatePhysics开启，那么子组件的Simulating Generates Hit Events开启，必定会响应子组件的OnCompnentHit事件

2.Actor的子组件的OnComponentHit事件响应，那么父Actor的OnActorHit也会响应

3.有些Static Mash开启了simulat physics 但是没有效果，原因是没有在网格编辑界面添加碰撞盒，和在Collision Complexty选项选取 Use Complex As Simple,如果设置了复杂的碰撞盒也同理，设置反了容易无法正确模拟出物理碰撞。

4.Collision使用的碰撞体是继承自Mesh的，在MeshEditor编辑

```
有三种选择，第一种是Simple，第二种是Complex，第三种是Default。
Simple：Mesh的碰撞体全部由简单立体图形组成，比如Sphere，Cube等等，可以手工添加和删除，与Mesh本体的匹配度不是很高，但是只有Simple支持PhysicsSimulate，这个是重点。 
Complex：每一个Ploy都会计算碰撞，计算量相当大，于是不支持PhysicsSimulate，实用Complex碰撞之后，如果再使用物理模拟的话，会穿过底面掉到无穷远去。
```

5、人物运动就是我们人物的运动受控于两个大组件，一个是物理引擎，还有一个是MovementComponent。MovementComponent一定要注意，它很容易被忽视，比如CharacterMovementComponent，它的内部也实现了一个重力移动逻辑，和物理引擎是完全独立的！当你杀死一个人之后，把他的CapsuleComponent设置为Overlap，你就会发现他的CapsuleComponent陷到地下去了，你关了SimulatePhysics，关了GravityEnable都不行，这就是因为他的CharacterMovementComponent内部还是有一个重力逻辑，会使他落下，加上你把他的碰撞关掉了，所以他就陷到地底去了。

6.当使用AddActorWorldOffset对人物进行移动时，人物将不会发生碰撞，因为是强制位移，要使用就要勾选Sweep。

7.有时不可避免的也会出现穿墙现象，是因为检测是按照帧来的，所以当你速度够快或者正好卡在某个帧检测之间碰撞了，是会出现穿墙现象，可以另外写逻辑来判断解决，例如：绝地求生穿到地底或者墙里面的情况。

### 特殊使用场景

##### 1.当我们要创建一个透明的墙，或者做某个复杂Actor的碰撞体积时

使用基础组价的BlockingVolume即可

##### 2.当我们要为一些使用画刷工具创建出来的StaticMash物体时，想要为其添加体积碰撞

在StaticMash物体的StaticMash组件下拉隐藏界面中有一个（CreateBlockingVolume)选项，点击此选项将会将会根据此物体的不同形态常见不同的透明碰撞盒

### 参考博客

physics:<https://blog.csdn.net/u012999985/article/details/51078906>

UE4中的Collision（碰撞）规则梳理 <https://blog.csdn.net/SittingAtThisMoment/article/details/80118748>

碰撞参考指南 <http://api.unrealengine.com/CHN/Engine/Physics/Collision/index.html>

UE4蓝图碰撞检测解析 <https://blog.csdn.net/u012999985/article/details/51113853>

UE4碰撞规则详解(2016.7.12更新) <https://blog.csdn.net/zhangxsv123/a>