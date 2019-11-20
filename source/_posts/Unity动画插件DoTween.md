---
title: Unity 动画插件DoTween
date: 2017-12-08 9:22:54
tags: [Unity3D,动画插件]
categories: Unity DoTween动画插件
---


DoTween Pro是一款untiy插件，是untiy中最好用的tween插件，实现脚本和视觉脚本的新功能，支持包括移动，淡出，颜色，旋转，缩放，打孔，摇动，文本，相机属性等，DoTween与其他动画插件相比，它的效率是最高，支持支持可视化编程。
<!--more-->

## 1.DoTween中数值的曲线变化

> 数值从一个固定值变化到另一个固定值，变化速度先快后

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using DG.Tweening; //引入标准库

public class move : MonoBehaviour {
/** 接收物体 */
public Transform cubeTransform;
/** 创建一个位置信息 */
public Vector3 Value = new Vector3(0, 0, 0);
/** 创建一个面板信息 */
public RectTransform taskpanelTransform;

void Start () {
/**
* 用动画的方式改变变量的值(通过差值的方式去修改一个值的变化)：变化成曲线进行，先快后慢
* @param ()=> - 指定一个目标
* @param x=> - 给x的系统值赋值
* @param Vector3 - 目标位置
* @param 2 -移动到目标位置需要的时间
*/
DOTween.To(() => Value, x => Value = x, new Vector3(10, 10, 10), 2);
}

void Update () {
//把cubeValue的值赋给物体
cubeTransform.position = Value;
taskpanelTransform.localPosition = Value;
}
}
```

#### 能实现的功能：

##### 1.游戏物体的移动

![移动物体](http://yp.guohaonan.cn/%E7%A7%BB%E5%8A%A8.gif)

#### 2.UI面板的移动

![面板移动](http://yp.guohaonan.cn/%E7%A7%BB%E5%8A%A8.gif)

UI面板移动代码优化：

```
public RectTransform panelTransform;
public void OnClick()
{
panelTransform.DOLocalMove(new Vector3(0, 0, 0), 0.5f);
}
```

#### 3.动画的倒放

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using DG.Tweening;
public class ButtonClick : MonoBehaviour {

public bool isIn = false;
public RectTransform panelTransform;
void Start()
{
Tweener tweener = panelTransform.DOLocalMove(new Vector3(0, 0, 0), 0.5f);//默认播放完动画就销毁了
/** 创建一个Tweener对象保存这个动画的信息 */
tweener.SetAutoKill(false); // 把自动销毁动画设置为false
tweener.Pause();
}

public void OnClick()
{
if (isIn == false)
{
panelTransform.DOPlayForward(); // 前放（与back是一对)
isIn = true;
}
else
{
/** 使面板离开视野 */
panelTransform.DOPlayBackwards(); //倒放之前的动画
isIn = false;
}
}
}
```

实现:
![倒放](http://yp.guohaonan.cn/%E5%80%92%E6%94%BE.gif)

---

## 2.常规设置

##### 1.设置动画播放曲线

```
Tween tweener = transform.DOLocalMoveX(0, 5);
// -------设置动画的播放曲线
tweener.SetEase(Ease.InBounce);
```

##### 不同的曲线类型（缓动函数）： ![缓动函数](http://yp.guohaonan.cn/unity/DOTween%E7%BC%93%E5%8A%A8.bmp)2.设置动画的循环次数

```
tweener.SetLoops(1);
```

##### 3.设置动画事件函数

```
tweener.OnComplete(OnTweenComplete);

void OnTweenComplete()
{
Debug.Log("完成");
}
```

动画事件函数有：


**OnComplete(TweenCallback callback) 动画播放完成时调用
OnKill(TweenCallback callback) 动画被销毁时调用
OnPlay(TweenCallback callback) 动画播放时调用（可被调用多次）
OnPause(TweenCallback callback) 动画暂停时被调用
OnRewind(TweenCallback callback) 动画被重置时调用
OnStart(TweenCallback callback) 动画刚开始播放时被调用(只会被调用一次)
OnStepComplete(TweenCallback callback) 动画完成一个循环时被调用
OnUpdate(TweenCallback callback) 动画被更新时被调用
OnWaypointChange(TweenCallback<int> callback 动画改变时被调用**

#### 文字的播放

```
private Text text;
void Start () {
text = this.GetComponent<Text>();
text.DOText("恭喜你挑战成功，我们即将开始下一个篇章的战斗!", 3);
}
```

演示：

### ![文字的播放](http://yp.guohaonan.cn/%E6%92%AD%E6%94%BE%E6%96%87%E5%AD%97.gif)

#### 摄像机的震动（增强打击效果）

```
void Start () {
//对摄像机进行抖动，1，震动时间，Vector3:震动范围
transform.DOShakePosition(1, new Vector3(3, 3, 0));
}
```

##### 效果： ![震荡](http://otxqcugty.bkt.clouddn.com/unity/DOTween震动.gif)

#### 颜色的渐变与透明度渐变

```
text = GetComponent<Text>();
//慢慢变成红色
text.DOColor(Color.red, 3);
//慢慢的显示出来
text.DOFade(1, 3);
```

显示：

![颜色渐变](http://yp.guohaonan.cn/unity/DOTween%E9%9C%87%E5%8A%A8.gif)

## 3.DoTween的可视化编辑

![可视化](http://yp.guohaonan.cn/unity/DOTween%E6%B8%90%E5%8F%98%E9%A2%9C%E8%89%B2.gif)

## 4.DoTweend的路径编辑器

![路径](http://yp.guohaonan.cn/unity/DOTween%E8%B7%AF%E5%BE%84.png)

### 1.基本介绍：


主要功能就是：

a.路径点的创建和删除。

b.路径的可视化。

c.路径动画的控制。


### 2.Path路径动画的创建：


静态创建：

在需要添加Path动画的物体上挂上 DOTweenPath 组件.

a. Shift + Ctrl ： 添加路径点

b. Shift + Alt ： 删除路径点
动态创建：

transform.DOPath\(vector3\[\] waypoints,float duration\);

a. waypoints ： 路径点

b. duration ： 动画时间

c. pathtype ： 路径类型，路径类型分为线性或者利用CatmullRom插值算法形成的曲线。\(默认参数\)

d. pathmode ： 路径模式，主要是用于对物体三个方向上的旋转的限制。\(默认参数\)

e. resolution ：CatmullRom算法的参数,数值越大曲线越精细,一般5足以，默认为10。\(默认参数\)

f. gizmoColor ：辅助线的颜色,只会在动画Running时在Secene面板上可见。 （默认参数）


### 3.Path路径动画的属性：

一般动画有的属性Path都有，几个独特的属性：

a. Ease ：动画类型

b. ClosePath ：封闭路径，如果勾选此属性路径将会形成一个封闭环。

c. LocalMovement ：局部移动，如果勾选此属性将会按照局部坐标移动。

d. Orientation ：运动朝向，分为ToPath朝向路线LookAtTran朝向Tran和LookatPos朝向点.

e. LookAhead ：朝向前瞻性，数值越大朝向约向靠近更前方的点。

f. Relative ：点相对，表示路径点是否与物体为相对的。


### 4.Path路径动画的相关方法：


一般动画有的方法Path都有,几个独特的方法：

a. SetOptions\(bool closePath,AxisConstraint lockPos,AxisConstraint lockRota\);

i.是否为封闭路径

ii.路径上三个维度的位置限制，给的参数为AxisConstraint.X,那么路径在X上位置不会变化。

ii.路径上三个维度的方向限制，给的参数为AxisConstraint.X,那么路径在X上方向不会变化。

b. SetLookAt\(\) 设置Path动画 Orientation 属性的。

c. PathLength\(\) 返回路径长度。

d. PathGetPoint\(float pathPecentage\); 参数为0~1小数,返回路径上小数百分比对应的点。

e. PathGetDrawPoint\(float pathPecentage\);参数为返回构成路径点的个数。

注意d.e两个方法，如果返回为Vector3.zero或者null.表示路径无效、路径尚未初始化或者这不是一个路径动画.封闭路径
##### 设置效果：

![路径](http://yp.guohaonan.cn/%E5%BE%AA%E7%8E%AF.gif)
