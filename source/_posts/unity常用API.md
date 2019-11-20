---
title: unity3d常用API
date: 2017-09-25 14:55:45
tags: Unity3D
categories: unity3dAPI
---


挑选了Unity引擎里一些核心API类例如 Object、GameObject、Rigidbody、Transform、Camera、Quaternion、Vector3等进行了详细的功能注解，注解内容包括API的使用方法、算法分析、边界条件、参数间的制约关系及注意事项等，特别是对很多功能相近或使用方法相似的API进行了较为详细的比较说明


<!--more-->

##### 函数调用顺序：

##### Awake：最开始调用，做一些初始化工作。建议少用，此刻物体可能还没有实例化出来，会影响程序执行顺序。

##### Start：不是很紧急的初始化，一般放在Start里面来做。仅在Update函数第一次被调用前调用。

##### Reset：用户点击检视面板的Reset按钮或者首次添加该组件时被调用。此函数只在编辑模式下被调用。Reset最常用于在检视面板中给定一个最常用的默认值。

##### Update：每一帧调用一次，帧间隔时间有可能改变。

##### FixedUpdate：以相同时间间隔调用，用在力学更新效果中。执行在Update之前。

##### LateUpdate：在Update和FixedUpdate调用之后调用。一般人物的移动放在Update中，而摄像机的跟进变化放到FixedUpdate中。确保两个独立。

##### On开头的方法，是由其他事件触发调用的。

##### OnDestory：物体被删除时调用。

##### OnEnable：物体启用时被调用。

##### OnDisable：物体被禁用时调用。

##### OnGUI：这个函数会每帧调用好几次（每个事件一次），GUI显示函数只能在OnGUI中调用

有关函数调用顺序可以参考官方的文档：

![](http://yp.guohaonan.cn/unity/blog%E8%B0%83%E7%94%A8.png)

## time类

---

> ##### unity3D的time类在游戏中十分重要，几乎所有的游戏都要用到time类，使用场景有：技能的释放时间，限时关卡的限时，场景的加载时间等等。

#### 方法：

##### timeSinceLevelLoad :加载场景完成之后所花费的时间（场景开始，开始计时）

##### fixedDeltaTime:每一帧所花费的时间

##### frameCount:一共运行了多少帧

##### Time.realtimeSinceStarup:从游戏开始到现在所运行的时间（当游戏暂停时和在后台运行时这个时间会一直增加

##### Time.fixedTime:游戏运行的时间

##### Time.fixeDeltaTime:游戏运行时间

##### Time.time:游戏运行时间

##### Time.smoothDeltaTime：平滑的时间变化

##### Time.deltaTime:每一帧的时间间隔（变化较大）

##### Time.timeScale:设置时间比例（可以暂停运动的物体，可以放慢动作

##### 测试性能：（开始时记录时间，结束时记录时间）

```
float time1 = Time.realtimeSinceStartup;
for (int i = 0; i < runCount; i++)
{
Method1();
}
float time2 = Time.realtimeSinceStartup;
Debug.Log(time2 - time1);
```

##### 会动的物体：

```
cube.Translate(Vector3.forward * Time.deltaTime); //每帧向前运动1m乘以没一帧时间间隔
```



## GameObject类

---

> ##### 所有游戏物体与场景的基类

##### 创建物体的三种方法：

##### 一.new一个

```
GameObject go = new GameObject("cube");
```

二.根据prefab新建\(相当于克隆）（游戏物体都可以）

```
GameObject.Instantiate(profab);
```

三.创建原始的几何体

```
GameObject.CreatePrimitive(PrimitiveType.cube);
```

##### 给一个物体添加组件

```
GameObject.CreatePrimitive(PrimitiveType.cube);
GameObject go = GameObject.CreatePrimitive(PrimitiveType.Cube);
go.AddComponent<Rigibody>() //添加刚体
go.AddComponent<API01EventFunction>() //添加脚本
```

### 变量：

##### activeInHerarchy: 禁用父类但是不禁用子类\(判断游戏物体是否激活）

##### activeSelf：禁用子类与父类（把一些不用的物体设置，可以节约性能；

##### tag: 用于区别不同物体

##### name:获取物体的名字

```
Debug.Log(go.name);
Debug.Log(go.GetComponent<Transform>().name)//获取组件的名字其实获取的是物体的名字
```

##### layer:可有获取游戏所在的层

##### scene:可以获取游戏所在的场景

### public Functions:

##### AddComponent:为一个物体添加组件

##### CompareTag:比较两个物体的标签是否一样

##### SetActive:用于激活和禁用一个物体

##### BroadcastMessage:广播一个消息（所有物体都能接收）\(子类也会调用）

```
target.BroadcastMessage("Attack",null,SendMessageOptions.DontRequireReceiver);//如果有接受者则发送，如果没有不报错
void Attack() //当一个物体的脚本里拥有此方法则会被调用（子类也会调用）
{
Debug.Log(this.gameObject + "正在攻击");
}
```

##### SendMessage:发送一个消息（不会调用子类）

##### SendMessageUpwards: 发送消息（当前物体与父类都也会调用）

### Static Functions:

##### CreatePrimitive:创建某个基本的几何体

##### Find:根据游戏名字查找（性能; 会遍历所有游戏物体，慎用）

```
GameObject go =GameObject.Find("Main Camera");
go.SetActive(false);
```

##### FindGameObjectsWithTag:根据游戏标签查找\(会查找所有的游戏物体）（效率高）

```
GameObject[] gos =GameObject.FindGameObjectsWithTag("MainCamera");//返回的是一个数组
gos[0].SetActive(false);
```

##### FindWithTag:根据游戏标签 查找（只会查找第一个）（查找到物体后一般要做判断，不为空的时候在进行操作）

## Object类

---

> ##### 游戏中所有对象的基类，主要用于对对象的销毁，查找

##### Destroy:被销毁，但是不会立刻被回收，会将要销毁的物体集中到一起，真的不用了再销毁

```
Destroy(this);(一般是脚本）（可以销毁物体也可以销毁组件）
Destroy(gameObject,5); //(5秒后销毁）
```

##### DestroyImmediate:立刻将游戏物体从场景中删除（不建议用，有可能会造成空指针\)

##### DontDestroyOnLoad:不要销毁某个物体（一般从某个场景转到下一个场景，会销毁所有的物体）会带到下一个场景，始终存在。设置共享的游戏物体

##### FindObjectOfType:查找某个游戏物体（根据组件类型查找游戏物体，找到第一个就返回，不管其他的）

##### FindObjectsOfType:会查找所有符合的组件（不查找未激活，被禁用的）

```
Light light =FindObjectOfType<Light>(); //获取所有的灯
light.enabled = false;
```

##### Injstantizte:实例化一个对象

## MonoBehaviour

---

> ##### 所有游戏脚本的基类，用于对游戏脚本进行操作

##### monoBehaviour中很多方法都与GameObject中的方法一致，在对MonoBehaviour进行操作就是对游戏物体进行操作

##### GetComponent:获得一个组件

```
Cube cube = target.GetComponent<Cube>(); //类
Transform t = target.GetComponent<Transform>();
```

##### GetComponents: 获得所有的相同组件

```
Cube[] cubes = target.GetComponents<Cube>();
cubes = target.GetComponentsInChildren<Cube>();
foreach(Cube c in cubes)
{
Debug.Log(c);
}
```

##### ExecuteInEditMode: 一个特性（把此方法放在类上可以使类在编辑模式下就被调用）

```
[ExecuteInEditMoe]
public class PrintAwake : MonoBehaciour
{
}
```

##### invoke: 调用某个方法（判断方法是否被调用，如果没有被调用，就会在2秒后被调用）

```
Invoke("Attack", 3);
void Update () {
bool res = IsInvoking("Attack");
print(res);
}
void Attack()
{
Debug.Log("正在攻击目标");
}
```

##### InvokeRepeating :重复调用某个方法

```
InvokeRepeating("Attack", 4, 2);//第四秒开始调用第一次，之后每2秒调用一次
```

##### CancleInvoke: 取消调用某个方法（取消所有的调用）

```
CancelInvoke("Attack");
```

##### gameObject: 可以获取游戏脚本所在的物体

##### IsActiveAndEnabled:可以判断当前组件是否启用

##### enabel: 可以启用与禁用某个组件（禁用的updata方法）

##### print:输出（只能在MonoBehaviour\)中使用

## 协程

---

> 在脚本运行过程中，需要额外的执行一些其他的代码，这个时候就可以将“其他的代码”以协程的形式来运行
>
> 相当于多开了一个线程是一个能暂停执行，暂停后立即返回，直到中断指令完成后继续执行的函数。
>
> 它类似一个子线程单独出来处理一些问题，性能开销较小。

##### 调用函数的时候不会等函数方法运行完才继续执行

返回值是IEnmerator;

返回参数的时候使用yield return;

协程方法的调用StartCoroutine\(method\(\)\);

##### 利用协程实现物体改变颜色

```
StartCoroutine(ChangeColor());

IEnumerator ChangeColor()
{
print("hahaColor");
cube.GetComponent<MeshRenderer>().material.color = Color.blue;
print("hahaColor");
yield return null;
}
```

##### 利用协程实现物体颜色改变的动画效果

```
void Update () {
if (Input.GetKeyDown(KeyCode.Space))
{
StartCoroutine(Fade()); //协程可以任何时候开启
}

}
IEnumerator Fade()
{
for (float i = 0; i <= 1; i += 0.1f)
{
cube.GetComponent<MeshRenderer>().material.color = new Color(i, i, i, i);//渐变色
yield return new WaitForSeconds(0.1f);
}
}
//
IEnumerator Fade()
{
for ( ;;)
{
//cube.GetComponent<MeshRenderer>().material.color = new Color(i, i, i, i);//渐变色
Color color = cube.GetComponent<MeshRenderer>().material.color;
Color newColor = Color.Lerp(color, Color.red,0.02f);
cube.GetComponent<MeshRenderer>().material.color = newColor;
yield return new WaitForSeconds(0.02f);
if (Mathf.Abs(Color.red.g - newColor.g) <= 0.01f)
{
break;
}
}
}
```
![](http://yp.guohaonan.cn/unity/blog%E6%B8%90%E5%8F%98%E8%89%B2.gif)


##### startCoroutines:开始协程

##### StopCoroutine: 停止指定的协程

##### StopAllCoroutines: 停止所有的协程

```
public IEnumerator ie;
void Update () {
if (Input.GetKeyDown(KeyCode.Space))
{
ie = Fade();
StartCoroutine(ie); //协程可以任何时候开启
}
if (Input.GetKeyDown(KeyCode.E))
{
StopCoroutine(ie);
}

}
```

### Message

##### OnMouseDown ：监听鼠标是否点击这个物体

##### OnMouseDrag ：监听鼠标按下的过程中拖拽的时候

##### OnMouseEnter : 鼠标移上去的时候（跟鼠标按不按下没有关系）

##### OnMouseExit : 鼠标移开的时候

##### OnmouseOver : 鼠标在物体上的时候

##### OnMouseUp ：鼠标点击完抬起的时候

##### OnMouseUpAsButton:鼠标按下与抬起实在同一个物体的时候（点击）

```
void OnMouseDown()
{
print("Down");
}
void OnMouseUp()
{
print("up");
}
void OnMouseExit()
{
print("Exit");
}
void OnMouseDrag()
{
print("Drag");
}
void OnMouseEnter()
{
print("Enter");
}
void OnMouseOver()
{
print("Over");
}
void OnMouseUpAsButton()
{
print("Button");
}
```

## Mathf

---

> ### 有关于数学公式的函数

### Sttic Variables:

##### PI:3.1415926.....

##### Deg2Rd:把读数变为弧度

##### Red2Deg:把弧度变为读数

##### epsilon ：无限小的小数

##### Infinity : 无限大的数字

##### NegativeInfinlty : 无线大的数字

### Static Functions

---

##### Abs:取绝对值

##### Ceil: 向上取整（负数也可以）

##### CeilToInt: 向上取整返回整数

##### Clamp :夹紧（当小于min，取min,当大于max,取max\)

```
transform.position = new Vector3(Mathf.Clamp(Time,time,1.0F,3.0),0,0) //将一物体从1移到3后停止
hp = Mathf.Clamp(hp,0, 100); //限制某个数值在什么范围内
```

##### Clamp01: 把一个值限定在（0,1）之间

##### DeltaAngle : 算出两个角度的最短的差

##### Exp: e的多少次方

##### ClosePowerOfTwo :取得离二得次方最近的数

```
print(Mathf.ClosestPowerOfTwo(2)); //取得离二得次方最近的数
print(Mathf.ClosestPowerOfTwo(3));
```

##### floor: 向下取整

##### FloorToInt：向下取整返回int

##### Max　取得最大值

```
print(Mathf.Max(1, 2));
print(Mathf.Max(1, 2, 5, 3, 10));
```

##### Min: 取得最小值

```
print(Mathf.Min(1, 2));
print(Mathf.Min(1, 2, 5, 3, 10));
```

##### Pow\(f,p\) 取得ｐ的f次方

```
print(Mathf.Pow(4, 3)); //4的3次方
```

##### Sqrt : 取得参数的平方根

```
print(Mathf.Sqrt(3));
```

##### Lerp: 差值运算：\(可以控制角色的移动，也可以控制动画）

```
cube.position = new Vector3(0, 0, 0);
float x = cube.position.x;
//float newX = Mathf.Lerp(x, 10, Time.deltaTime); //一秒后到达位置每帧调用每帧所花的时间间隔
float newX = Mathf.Lerp(x, 10, 0.1f); //每一帧都会用新组成的数进行差值运算
cube.position = new Vector3(newX, 0, 0); //先快后慢
```

##### LerpAngle: 角度的差值运算

##### MoveTowarrds : 想某个方向移动

```
float newX = Mathf.MoveTowards(x, 10, 0.02f); //匀速运动
```

##### PingPong: 从0到最大值来回移动（不能设置最小值）只能设置长度（当超出长度的时候就开始减小）

```
print(Mathf.PingPong(t,30));
//print(Mathf.PingPong(t,30));
cube.position = new Vector3(Mathf.PingPong(Time.time, 10), 0, 0);
```

![](http://yp.guohaonan.cn/unity/blog%E6%9D%A5%E5%9B%9E%E7%A7%BB%E5%8A%A8.gif)

## Input

---

> ##### 有关于键盘鼠标等输入的函数与设置

##### GetKey: 监测按键按下（按着会一直返回ture\)

```
if(Input.GetKeyDown("left shift"))
{
print("left shfit");
}
```

##### GetkeyDown : 监测按键按下（只有在按下时会返回ture）

##### GetKeyUp: 监测按键抬起

##### GetButton: 监测虚拟按键

##### GetMouseButton: 监测鼠标的按键（0，左键1，右键2，中键）（按下的过程中会一直抬起）

##### GetAxis: 控制运动（运动具有渐变效果）

```
cube.Translate(Vector3.right * Time.deltaTime * Input.GetAxis("Horizontal"));
```

##### GetAxisRaw: 运动没有渐变效果（缓冲效果）

##### GetTouch: 监测触摸

#### Static Variables

##### acceleration: 监测重力

##### gyo : 监测陀螺仪

##### anyKeyDown: 检测任何键的按下（会一直返回ture\)\(鼠标，按键）

```
if(Input.anyKeyDown)
{

}
```

##### mousePosition:　检测鼠标在屏幕的位置

```
print(Input.mousePosition）;
```

## vector2

---

> #### 2d模式下有关向量与位置的操作

#### Static Variables

##### down,left,right,up: 上下左右

##### one,zero: 坐标轴为（1,1）（0,0）的向量

#### Variables

##### magnitude: 获取向量的长度

##### normalized: 把向量单位化（把一个不管长度为多少的向量化为1）

##### sqrMagnitude：向量长度还没有被平方根的长度（节省性能）

##### x,y :获取x,y

```
ptrint(Vector2.down);
Vector2 a= new Vector2(2,2);
Vecotr2 b = new Vector(3,5);
print(a.magnitude);
print(a.sqrMagnitude);
```

#### Constructors\(构造方法）

##### vector2: 直接new一个

##### 修改位置：向量是结构体，是值类型，要整体赋值。

`transform.position = new Vector3(3,3,3); //不能直接修改`

`Vector3 pos = transform.position;`

`pos.x = 10;`

`transform.position = pos;`

#### Public Functions

##### Equals: 判断两个向量是否相等

##### normalized: 把自身单位化

##### Set: 进行赋值

##### ToString：转换为String;

#### StaticFunction

##### Angle: 取得两个向量的夹角

##### ClampMagnitude：对向量的长度进行限定

##### Distance: 取向量之间的距离

##### Dot: 点乘

##### LerP: 对ｘ，ｙ进行两差值运算。

##### LerpUnclamped :差值运算不限定最大值

##### MoveTowards:从当前目标向目标运动

```
Public Vector2 a = new Vector(2,2);
public Vector2 target new Vector(10,3);
a = Vector2.moveTowards(a,target,Time.deltaTime).
```

## Vector3

---

> #### 3d模式下有关向量与位置的操作

##### Class:　两个向量进行叉乘运算

##### OrthoNormalize: 让两个向量的夹角为９０度

##### Project: 一个向量在另一个向量上的投影

##### Reflect: 将向量反射

##### Slerp: 求两个向量的差值（按照角度）

##### operator - \* / + ==对向量的运算（判断是否相等）

## Random

---

> ##### 随机生成数，在游戏中有着广泛的应用，用于增加游戏的不确定性

##### Range: 返回一个随机数\(不包含最大值）

```
print(Random.Range(4,10)）;
print(Random.Range(4,10f);
```

##### InitState: 设置随机数生成的种子

```
Random.InitState(0);
```

##### ColorHSY:随机生成一个颜色

##### value: 生成一个0到１的小数（包括０，１）

##### State:获取一个种子

##### rotation: 随机获取一个方向（朝向）

##### InsideUnitCirle:在一个圆内随机生成坐标（可以用于生成敌人）\(二维）

```
cube.position = Randon.insideUnitCircle*5;
```

![](http://yp.guohaonan.cn/unity/blog%E9%9A%8F%E6%9C%BA.gif)

##### InsideUnitsphere: 在一个圆内随机生成

## Quaernion\(四元数）

---

> 控制物体旋转

##### eulerAngles: 把四元数转换为欧拉角\(常用）

```
cube.eulerAngles = new Vetor3(45,45,45);
```

##### Euler:把欧拉角转换为四元数

```
cude.rotatiion = Quaternion.Euler(new Vector(45,45,45));
```

##### LookRotation:会使物体旋转到目标方向（望向目标）

```
if (Input.GetKeyDown("left shift"))
{
Vector3 dir = enemy.position - player.position;
／／dir.y = 0; 会忽略ｙ的高度
player.rotation = Quaternion.LookRotation(dir);
}
```

![](http://yp.guohaonan.cn/unity/blog%E8%BD%AC%E5%90%91.gif)

使转动更加平滑：

```
Quaternion target = Quaternion.LookRotation(dir);
player.rotation = Quaternion.Slerp(player.rotation, target, Time.deltaTime);
```

## Rigidbody

---

> ##### 刚体组件，使物体受力的作用

##### freezeRotation：冻结旋转

##### position: 控制位置\(比用transform块）

```
playerRgd.position = playerRgd.transform.position +
Vector3.forward * Time.deltaTime;
```

##### MovePositon: 持续的运动用着个（用差值的方式使运动更

加平滑）

##### rotation: 控制旋转（比用Transform快）

##### AddForce:　给物体施加力

```
playerRgd.AddForce(Vector3.forward * force);
```

##### MoveRotation: 控制持续的旋转

```
playerRgd.position = playerRgd.transform.position +
Vector3.forward * Time.deltaTime;
```

## Cameera

---

> ##### 摄像头，用于控制视角

##### main:直接获取mainCamera

`Camer.main`

##### 通过Camer把屏幕上的点转为射线，用于检测鼠标是否移动

到物体上

##### ScreenPointToRay: 把屏幕坐标转换为射线

鼠标移动输出在鼠标上的物体：

```
Ray ray = mainCamera.ScreenPointToRay

(Input.mousePosition);

RaycastHit hit;



bool isCollider = Physics.Raycast(ray, out hit);



if(isCollider)

{

Debug.Log(hit.collider);

}
```

##### 从屏幕上射出一条射线

```
Ray ray = mainCamera.ScreenPointToRay(new Vector3
(200,200,0));
Debug.DrawRay(ray.origin, ray.direction * 10,
Color.yellow);
```

## Application

---

##### dataPath: 数据路径

##### Streaming Assets: 资源文件\(创建后不会被打包）

##### identifier: 包的名字

##### isFocused: 判断是否是焦点

##### isMobile： 是否在移动平台

##### isPlaying: 编辑模式下载运行会返回ture

##### 在编辑器模式下退出游戏

```
UnityEditor.EditorApplication.isPlaying = false;
```

##### Plaatform: 可以控制在某个平台下运行

##### identifier:标识名

##### companyName: 公司名

##### prodctName: 游戏名字

##### runlnBackGround: 是否在后台运行

##### unityVersion: 判断unity的版本

##### OpenURL: 可以打开页面

## sceneManager（场景管理器）

---

> #### 场景管理器用于管理场景的切换与销毁，加载

#### Static Variables

##### sceneCount 加载的场景数量

##### sceneCountlnBuildSettingss 加载列表中的场合

##### LoadScene 加载场景

##### LoadSceneMode 加载场景有两种模式Single \(单独的）：把所有的物体摧毁加载新的场景 Addditive:　不改变原来的场景下，加载新的场景

```
using UnityEngine.seneManagement;
if(Input.GetKeyDown(KeyCode.Space))
{
SceneManager.LoadScene(1);
}
```

##### LoadSceneAync 异步加载场景\(h会返回一个AsynOperation的对象，利用AsyncOperation的progress对象会获得0到一的值，1代表完成，isDone,判断是否完成\)

##### GetActiveScene：获得当前场景

```
SceneManager.GetActiveScene().name;
```

##### GetSceneByName: 用名字获取场景

##### GetSceneByPath: 用名字来获取场景

##### SetActiveScene: 激活某个场景，用来激活某个场景。

##### acticeSceneChanged:　加载某个场景的时候

##### sceneLoadded: 场景加载完成的时候触发

```
void OnActiveSceneChanged(Scene a,Scene b)
{
print(a.name);
print(b.name);
}
void OnSceneLoaded(Scene a,LoadSceneMode)
{
print(a.name + " " +mode);
}
```

##### sceneUnloaded: 场景被销毁的时候

##### GetSceneByBuildIndex: 获取已加载的场景的信息

##### Quit:退出游戏

##### CaptureScreenshot： 游戏截图


