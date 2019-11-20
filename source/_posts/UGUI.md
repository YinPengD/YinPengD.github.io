---
title: Unity UGUI界面
date: 2017-10-02 20:58:56
tags: Unity3D
categories:  unit UGUI界面
---


不属于3d环境中的时刻显示在屏幕上的，用于游戏的开始菜单，RPG游戏的菜单栏，侧边栏与功能栏比如背包系统，任务列表，设计用来控制移动的虚拟杆和攻击的攻击按钮

<!--more-->
---

## 创建一个公告的文本列表

>  游戏内的公告板用于对玩家的提示，或者是任务的了解

##### 1.创建一个UI的image

##### 2.只能上下拖动就要取消勾选以下按钮：

![](http://yp.guohaonan.cn/%E5%8F%96%E6%B6%88%E5%8B%BE%E9%80%89.png)

##### 3. 在刚创建的image下创建一个text

![](http://yp.guohaonan.cn/%E5%88%9B%E5%BB%BATxt.png)

##### 4。为了只显示在框内要添加Scoll Rect组件

![](http://yp.guohaonan.cn/%E6%B7%BB%E5%8A%A0ScollRect.png)

##### 5.为了消除白板要添加Mash组件

![](http://yp.guohaonan.cn/%E6%B7%BB%E5%8A%A0Mash.png)

##### 6.添加Scrollbar滑动组件

##### 7.把Scrollbar组件添加到被滑动的Image上

![](http://yp.guohaonan.cn/%E6%B7%BB%E5%8A%A0Scrollbar.png)

##### 最终实现的效果：

![](http://yp.guohaonan.cn/note%E6%95%88%E6%9E%9C.gif)

---
## 监控UI界面的按钮点击事件（点击开始按钮开始游戏）

>  用于游戏中场景的切换

##### 1.按钮上的点击事件：

![](http://yp.guohaonan.cn/%E7%82%B9%E5%87%BB%E4%BA%8B%E4%BB%B6.png)

##### 2.脚本代码：

```
using UnityEngine.SceneManagement;

public class StartGame : MonoBehaviour {

public void OnClick(string sceneName)
{
SceneManager.LoadScene(sceneName);
}
}
```

##### 实现：

![](http://yp.guohaonan.cn/%E7%82%B9%E5%87%BB%E5%AE%9E%E7%8E%B0.gif)



---
## 制作一个显示血条的UI:

>  血条的增减是游戏中的基本



##### 1.添加一个slider的组件：

![](http://yp.guohaonan.cn/%E6%B7%BB%E5%8A%A0slider.png)

##### 2.修改图片的类型:

![](http://yp.guohaonan.cn/%E4%BF%AE%E6%94%B9%E5%9B%BE%E7%89%87.png)

##### 实现：

![](http://yp.guohaonan.cn/%E8%A1%80%E6%9D%A1%E5%AE%9E%E7%8E%B0.gif)



---
## 制做一个技能

>  对于一个游戏角色来说，多变的技能必不可少，这里主要简单的实现了技能UI的检测，与触发

##### 1.创建一个鼠标点击事件和检测按钮按下脚本

![](http://yp.guohaonan.cn/%E5%88%9B%E5%BB%BA%E8%84%9A%E6%9C%AC.png)

##### 2.创建被技能加载的图片

![](http://yp.guohaonan.cn/%E5%88%9B%E5%BB%BA%E5%8A%A0%E8%BD%BD%E8%84%9A%E6%9C%AC.png)

##### 3.脚本代码：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI; //添加UI

public class skill1 : MonoBehaviour {
public float timer = 0;
public bool isStartTimer = false;
public float coldTimer = 2;
public Image Imageskill1;
public KeyCode keyCode1;
// Use this for initialization
void Start () {
Imageskill1 = transform.Find("load").GetComponent<Image>(); //寻找图片下的Image组件
}

// Update is called once per frame
void Update () {
if (Input.GetKeyDown(keyCode1)) //监控按钮时间
{
isStartTimer = true;
}
if (isStartTimer)
{
timer += Time.deltaTime; //开始计时
Imageskill1.fillAmount = (coldTimer - timer) / coldTimer; //把得到的时间的比例传递个Image组件
if(timer >= coldTimer)
{
timer = 0;
isStartTimer = false;
Imageskill1.fillAmount = 0;
}
}

}
public void OnClick() //监控点击事件
{
isStartTimer = true;
}
}
```

##### 4.开启监控：

![](http://yp.guohaonan.cn/%E5%BC%80%E5%90%AF%E7%9B%91%E6%8E%A7.png)

##### 最终实现：

![](http://yp.guohaonan.cn/%E6%8A%80%E8%83%BD%E5%AE%9E%E7%8E%B0.gif)

---
## 制作一个角色的物品栏面板

> 游戏中的物品栏面板用于玩家对自己已获得物品的了解

##### 1.为不同的物品栏面板添加toggel选项卡组件

![](http://yp.guohaonan.cn/%E6%B7%BB%E5%8A%A0Toggel.png)

##### 2.添加Togger Group组件使选项卡只能选择一个：

![](http://yp.guohaonan.cn/t%E6%B7%BB%E5%8A%A0Group.png)

##### 3.为面板添加自动布局

![](http://yp.guohaonan.cn/%E6%B7%BB%E5%8A%A0%E8%87%AA%E5%8A%A8%E5%B8%83%E5%B1%80.png)

##### 4.为了防止物体图片被自动布局影响，创建一个空的中间物体

![](http://yp.guohaonan.cn/%E4%B8%AD%E9%97%B4%E7%89%A9%E4%BD%93.png)

### 实现：

![](http://yp.guohaonan.cn/%E7%89%A9%E5%93%81%E9%9D%A2%E6%9D%BF%E6%A0%8F.gif)

---
## 制作一个关卡选择界面\(实现关卡的拖动与选择）

> 便于玩家选择关卡

##### 1.点击按钮实现关卡选择界面的切换：

![](http://yp.guohaonan.cn/%E7%82%B9%E5%87%BB%E6%8C%89%E9%92%AE.png)

##### 2.拖动与点击跟随触发代码：

```
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems; //实现接口所需要引用的
using UnityEngine.UI;

public class LevelButtonScroll : MonoBehaviour,IBeginDragHandler,IEndDragHandler { //实现接口
private ScrollRect scrollRect;
private float[] pageArray = new float []{ 0, 0.5f, 1 };
private float smoothing = 3; //滑动速度
private float targetPosition;
private bool isDragStart = false;
public Toggle[] toggler; //创建跟随触发的toggle数组
// Use this for initialization
void Start () {
scrollRect = GetComponent< ScrollRect > (); //获得ScrollRect组件

}

// Update is called once per frame
void Update () {
if(isDragStart ==false) //当开始拖动的时候才用这个方法赋予scrollRect值
scrollRect.horizontalNormalizedPosition = Mathf.Lerp(scrollRect.horizontalNormalizedPosition, targetPosition, Time.deltaTime * smoothing);
//缓慢的移动
}
public void　OnBeginDrag(PointerEventData eventDate)
{
isDragStart = true;
}

public void OnEndDrag(PointerEventData eventData)

{
isDragStart = false;
float posX = scrollRect.horizontalNormalizedPosition; //获取
int index = 0;
float offset = Mathf.Abs(pageArray[index] - posX); //初始化偏移量
for(int i = 1; i <pageArray.Length;i++)
{
float offsetTemp = Mathf.Abs(pageArray[i] - posX); //拖动的位置更与不同页的位置差
if (offsetTemp < offset) // 比较偏移的值更接近哪个页数的值
{
index = i;
offset = offsetTemp;
}
}
targetPosition = pageArray[index];
//scrollRect.horizontalNormalizedPosition = pageArray[index]; //返回接近的位置
toggler[index].isOn = true; //跟随触发
}


/*************设置按钮开关的调用方法**********************/
public void MoveTOPasage1( bool isOn) //当调用此方法时改变位置
{
if (isOn)
{
targetPosition = pageArray[0];
}
}
public void MoveTOPasage2(bool isOn)
{
if (isOn)
{
targetPosition = pageArray[1];
}
}
public void MoveTOPasage3(bool isOn)
{
if (isOn)
{
targetPosition = pageArray[2];
}
}
}
```

#### 实现：

![](http://yp.guohaonan.cn/%E5%85%B3%E5%8D%A1%E9%80%89%E6%8B%A9%E5%AE%9E%E7%8E%B0.gif)

---
## 制作一个设置界面（关于自定义Toggle组件）

> 设置界面便于玩家对游戏进行一些调整，这里重点实现了一个自定义Toggle开关的实现

##### 1.创建一个Toggle组件：

![](http://yp.guohaonan.cn/Toggle%E7%BB%84%E4%BB%B6.png)

##### 2.重写Toggle调用代码

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class MyToggle : MonoBehaviour {
public GameObject switch1; //用于显示不同开关
public GameObject switch2;
private Toggle toggle;
// Use this for initialization
void Start () {
toggle = GetComponent<Toggle>(); //得到物体上的TOggle组件
OnVlueChange(toggle.isOn); //初始化开关
}
// Update is called once per frame
void Update () {
}
public void OnVlueChange(bool isOn) //当按钮被点击时调用
{
switch1.SetActive(isOn);
switch2.SetActive(!isOn);
}
}
```

#### 实现：

![](http://yp.guohaonan.cn/%E5%AE%9E%E7%8E%B0%E8%AE%BE%E8%AE%A1%E9%9D%A2%E6%9D%BF.gif)
