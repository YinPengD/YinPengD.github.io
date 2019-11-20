---
title: 打砖块Demo的代码与演示
date: 2017-07-31 16:18:32
tags: Unity3D
categories: Unity3D
---

“打砖块”Demo主要内容：创建一面墙壁，通过鼠标点击发射子弹，击中墙壁，使其坍塌，涉及的知识：砖块的批量产生，人物的移动，射击子弹的触发，主要用与Unity的入门了解。

<!--more-->

#### 创建一面墙砖块

```
public class BrickWarGame : MonoBehaviour {

public GameObject brick;
private int columnNum = 10;
private int rowNum = 5;

void Start () {    
//Instantiate（ brick );       创建一个brick的实例对象
//Instantiate(brick,new Vector3(0,10f,0),Quaternion.idntity );    在(0,10f,0)创建一个brick的实例对象

for (int rowIndey = 0; rowIndey < 10; rowIndey++){
for (int columnIndex = 0; columnIndex < 10; columnIndex++){
// （被创建的物体 ， 创建的位置（0，5的时候刚好能露出来），是否旋转）
Instantiate(brick, new Vector3(columnIndex - 5,0.5f + rowIndey, 0), Quaternion.identity);
}
}
}
```

实例图：

#### [![img](http://yp.guohaonan.cn/unity/demo1brick.png)](http://yp.guohaonan.cn/unity/demo1brick.png)

#### 创建一个发射停止5秒会销毁的球

```
public class Ball : MonoBehaviour {

// Use this for initialization
void Start () {
Destroy(this.gameObject, 2f);
}
```

#### 创造一个会移动的发射者

```
public class Shooter : MonoBehaviour {
public GameObject shootPos;
public float force = 1000;
public Rigidbody ballPrefab;
public float moveSpeed = 10f;
// Use this for initializatio

// Update is called once per frame
void Update () {
//Instantiate(创建变量，从哪创建，创建的状态）
if (Input.GetButtonDown("Fire1") )
{
//当转化的类型出错的时候，unity就会为空，就会报错
Rigidbody ball = Instantiate(ballPrefab, shootPos.transform.position, shootPos.transform.rotation) as Rigidbody;
ball.AddForce(force * shootPos.transform.forward);
}
float h = Input.GetAxis("Horizontal") * moveSpeed * Time.deltaTime;
float V = Input.GetAxis("Vertical") * moveSpeed * Time.deltaTime;
transform.Translate(h, V, 0f);
}
}
```

实例图：

![img](http://yp.guohaonan.cn/unity/demo1shooter.png)
效果演示：

![e](http://yp.guohaonan.cn/unity/demo1%E6%89%93%E7%A0%96%E5%9D%973.gif)