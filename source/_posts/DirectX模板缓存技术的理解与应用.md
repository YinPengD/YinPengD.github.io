---
title: DriectX模板缓存技术的理解与应用
date: 2019-06-3 16:33:21
tags: [图形学,DirectX,模板缓存]
categories:  图形学
---

模板缓存用于获得某种特效的离屏缓存，它允许我们动态的有针对性的将某种像素写入后台缓存中，这使得我们可以实现一些特殊的效果，例如游戏中的镜子，水面反射，战争迷雾等，被应用于游戏中的方方面面。

<!--more-->

### 开启模板缓存

 由于模板缓存与深度缓存共享一个离屏的表面缓存，通常是创建的32位缓存，其中24位给深度缓存，8位给模板缓存，有些图形卡可能不支持8位的模板缓存，所以首先要查询设备是否能支持模板缓存，然后将其启用

```
Device->SetRenderState(D3DRS_STENCJLENABLE, true);
```

- 开启了模板缓存之后，我们要知道哪些区域是能够被绘制的镜面区域，防止将镜面成像绘制到了非镜面区域（比如墙壁上）
- 简单来说，模板缓存是用来控制哪些像素会被加载到后台缓存然后被渲染
- 那么如何对不同的像素（镜面区域）进行判断呢？
- 这就使用了模板测试对其进行判断

### 模板测试的底层原理

 其中标记的表达式为

**（ref & mask) ComparisonOperation (value & mask)** **// ref:模板参考值 mask：模板掩码 value: 模板值**

我对这个公式的理解为

1. 初始化时先用模板参考值 1
2. 对镜面像素进行标记,然后之后对镜面绘制时，如果这个像素为镜面，那么它的模板值就为 1 ，
3. 然后这个此像素的模板测试就是成功的，
4. 模板掩码是为了在特定情况下减少对比的数字而存在的。

##### 所以我们要对所有的像素进行镜面标记，就比较运算函数设置为模板测试总是会成功

```
Device->SetRenderState(D3DRS_STENCILFUNC, D3DCMP_ALWAYS);
```

##### 将镜面的所有像素都设置为1，对其进行标记

```
Device->SetRenderState(D3DRS_STENCILREF, 0x1); // 模板参考值设置为1：所有的模板缓存中的像素都设置为0x1,
```

模板掩码，用于屏蔽变量的某些位，默认不屏蔽

```
Device->SetRenderState(D3DRS_STENCILMASK, 0xffffffff);
```

写掩码的值，可以屏蔽写入的模板缓存，默认不屏蔽值

```
Device->SetRenderState(D3DRS_STENCILWRITEMASK, 0xffffffff);
```

根据像素深度测试与像素模板测试的成功与失败，我们将定义一下3种模板缓存值的更新方式

1.当像素模板测试失败时

```
Device->SetRenderState(D3DRS_STENCILFAIL, D3DSTENCILOP_KEEP);
```

2.当像素深度测试失败时

```
Device->SetRenderState(D3DRS_STENCILZFAIL, D3DSTENCILOP_KEEP);
```

3.当像素模板测试与像素深度测试都成功时

```
Device->SetRenderState(D3DRS_STENCILPASS, D3DSTENCILOP_REPLACE);
```

D3DSTENCTILOP的预定义常量有以下

- D3DSTENCILOP_KEEP 不史新模板缓存中的值（即，保留当前值）
- D3DSTENCILOP_ZERO 将模板缓存中的值设为0。
- D3DSTENCILOP_REPLACE 用模板参考值替代模板缓存中的对应值
- D3DSTENCILOP_INCRSAT 增加模板缓存中的对应数值，如果超过最大值，取最大值
- D3DSTENCILOP_DECRSAT 减小模板缓存中的对应数值，如果小千最小值，取最小值
- D3DSTENCILOP_INVERT 模板缓存中的对应值按位取反
- D3DSTENCILOP_INCR 增加模板缓存中的对应数值，如果超过最大值，则取0
- D3DSTENCILOP_DECR 减小模板缓存中的对应数值，如果小千0,则取最大值

### 应用模板缓存实现镜面效果

 因为镜面效果的本质是对像素进行融合预算的结果，所以我们要开启融合运算并对其进行设置

```
Device->SetRenderState(D3DRS_ALPHABLENDENABLE, true);  //开启融合运算
Device->SetRenderState(D3DRS_SRCBLEND, D3DBLEND_ZERO); //设置对象融合因子
Device->SetRenderState(D3DRS_DESTBLEND, D3DBLEND_ONE); //设置目标融合因子
```

我们设置了模板测试的底层，但是还没有定义在什么情况下模板测试成功

```
Device->SetRenderState(D3DRS_STENCILFUNC, D3DCMP_EQUAL); //设置比较运算函数，为LHS=RHS，就模板测试成功
```

之后还要设置模板测试成功之后要如何处理缓存

```
Device->SetRenderState(D3DRS_STENCILPASS, D3DSTENCILOP_KEEP); // 测试成功，就保留模板中缓存中的值
```

 经过以上对模板缓存的处理我们就可以保证镜面反射的信息只会在镜面中绘制了,这时候我们就要开始对反射的物体进行绘制了,根据镜面成像的数学原理：

[![mark](http://yp.guohaonan.cn/MPic/20190603/aGpsqMT1ky3y.png?imageslim)](http://yp.guohaonan.cn/MPic/20190603/aGpsqMT1ky3y.png?imageslim)

在DirectX中已经帮我们封装好了对矩阵的转换方法，我们只用设定图像的转换平面就行了

```
D3DXMATRIX W, T, R;
D3DXPLANE plane(0.0f, 0.0f, 1.0f, 0.0f); // xy plane
D3DXMatrixReflect(&R, &plane);   // 先对其进行对应xy轴进行平移
D3DXMatrixTranslation(&T,     // 平移完成之后应该对其进行镜像变换
	TeapotPosition.x,
	TeapotPosition.y,
	TeapotPosition.z);
```

在对图像进行了镜面转换后，由于图像转换后的深度大于镜面，所以我们要清除转换后的图像的深度缓存

然后对镜面图像进行融合运算

```
Device->SetRenderState(D3DRS_SRCBLEND, D3DBLEND_DESTCOLOR);    // 设置源融合因子
Device->SetRenderState(D3DRS_DESTBLEND, D3DBLEND_ZERO);  // 设置目标融合因子
```

最后还有一个很容易被遗忘的点就是，因为镜面的观察方向与实际物体的观察方向相反，所以我们看到的应该是物体的背面，所以，我们需要对物体的正面的进行背面消隐

```
Device->SetRenderState(D3DRS_CULLMODE, D3DCULL_CW);
```

最后将其绘制

```
Teapot->DrawSubset(0);
```

绘制完毕之后，包绘制是更改的绘制设定，改为正常的方式

```
Device->SetRenderState(D3DRS_ALPHABLENDENABLE, false);    // 禁用融合
Device->SetRenderState(D3DRS_STENCILENABLE, false);   // 禁用对模板的操作
Device->SetRenderState(D3DRS_CULLMODE, D3DCULL_CCW);    // 对背面的三角形进行消隐
```

### 镜面绘制实例的完整代码

```
void RenderMirror()
{
	// 对相关的绘制状态进行设置
	Device->SetRenderState(D3DRS_STENCILENABLE, true);   //开启镜面缓存
	Device->SetRenderState(D3DRS_STENCILFUNC, D3DCMP_ALWAYS);  //设置比较运算函数，模板缓存总会成功
	Device->SetRenderState(D3DRS_STENCILREF, 0x1);  // 模板参考值设置为1：所有的模板缓存中的像素都设置为0x1,即对镜面要绘制的部分进行标记
	Device->SetRenderState(D3DRS_STENCILMASK, 0xffffffff);  // 设置模板掩码，用于屏蔽变量的某些位，默认不屏蔽
	Device->SetRenderState(D3DRS_STENCILWRITEMASK, 0xffffffff);// 设定写掩码的值，可以屏蔽写入的模板缓存，默认不屏蔽值
	Device->SetRenderState(D3DRS_STENCILZFAIL, D3DSTENCILOP_KEEP); // FAIL:像素深度更新失败， KEEP:不更新模板缓存中的值（即保留当前值）
	Device->SetRenderState(D3DRS_STENCILFAIL, D3DSTENCILOP_KEEP); //  FAIL:像素模板更新失败， KEEP:不更新模板缓存中的值（即保留当前值）
	Device->SetRenderState(D3DRS_STENCILPASS, D3DSTENCILOP_REPLACE); // 设置模板缓存的操作方式，用模板参考值代替模板缓存中的对应值

	Device->SetRenderState(D3DRS_ZWRITEENABLE, false); //设置绘制状态，阻止对深度缓存进行写入操作
	Device->SetRenderState(D3DRS_ALPHABLENDENABLE, true);   //开启融合运算
	Device->SetRenderState(D3DRS_SRCBLEND, D3DBLEND_ZERO);  //设置融合因子对象
	Device->SetRenderState(D3DRS_DESTBLEND, D3DBLEND_ONE);  //设置目标融合因子

	// draw the mirror to the stencil buffer
	Device->SetStreamSource(0, VB, 0, sizeof(Vertex));
	Device->SetFVF(Vertex::FVF);
	Device->SetMaterial(&MirrorMtrl);
	Device->SetTexture(0, MirrorTex);
	D3DXMATRIX I;
	D3DXMatrixIdentity(&I);   // 创建一个单位矩阵
	Device->SetTransform(D3DTS_WORLD, &I);   // 转换为世界坐标
	Device->DrawPrimitive(D3DPT_TRIANGLELIST, 18, 2);   //绘制的图元有18个顶点，2个图元

	Device->SetRenderState(D3DRS_ZWRITEENABLE, true);  // 设置绘制状态可以对深度缓存进行写入操作

	// 只在镜子中绘制反射信息
	Device->SetRenderState(D3DRS_STENCILFUNC, D3DCMP_EQUAL);   //设置比较运算函数，为LHS=RHS，就模板测试成功
	Device->SetRenderState(D3DRS_STENCILPASS, D3DSTENCILOP_KEEP);  // 测试成功，就保留模板中缓存中的值
	// 对位置进行反射
	D3DXMATRIX W, T, R;
	D3DXPLANE plane(0.0f, 0.0f, 1.0f, 0.0f); // xy plane
	D3DXMatrixReflect(&R, &plane);   // 先对其进行对应xy轴进行平移
	D3DXMatrixTranslation(&T,     // 平移完成之后应该对其进行镜像变换
		TeapotPosition.x,
		TeapotPosition.y,
		TeapotPosition.z);
	W = T * R;
	Device->Clear(0, 0, D3DCLEAR_ZBUFFER, 0, 1.0f, 0);   // 清除深度缓存，因为茶壶深度大于镜面
	Device->SetRenderState(D3DRS_SRCBLEND, D3DBLEND_DESTCOLOR);    // 设置源融合因子
	Device->SetRenderState(D3DRS_DESTBLEND, D3DBLEND_ZERO);  // 设置目标融合因子
	// 绘制茶壶
	Device->SetTransform(D3DTS_WORLD, &W);   // 转换为世界坐标
	Device->SetMaterial(&TeapotMtrl);    //设置材质
	Device->SetTexture(0, 0);      // 设置纹理
	// 反置模式
	Device->SetRenderState(D3DRS_CULLMODE, D3DCULL_CW);  // 对正面的三角形进行背面消隐，因为反过来了
	Teapot->DrawSubset(0);   // 绘制
	// 绘制完成要恢复原来的状态
	Device->SetRenderState(D3DRS_ALPHABLENDENABLE, false);    // 禁用融合
	Device->SetRenderState(D3DRS_STENCILENABLE, false);   // 禁用对模板的操作
	Device->SetRenderState(D3DRS_CULLMODE, D3DCULL_CCW);    // 对背面的三角形进行消隐 
}
```

### 效果

> 如果不开启模板缓存就绘制图形，那么在镜面之外的墙面中也会出现茶壶的倒影

[![mark](http://yp.guohaonan.cn/MPic/20190603/WpERPCVF5Ipt.png?imageslim)](http://yp.guohaonan.cn/MPic/20190603/WpERPCVF5Ipt.png?imageslim)

