---
title: UE4反射系统与序列化整理
date: 2018-05-23 16:33:21
tags: [UE4,反射, 序列化]
categories:  UE4
---

#### 什么是UE4反射？

在UE4里面，你无时无刻都会看到类似UFUNCTION（）这样的宏。官方文档告诉你，只要在一个函数的前面加上这个宏，然后在括号里面加上BlueprintCallable就可以在编辑器里面调用了。按照他的指示，我们就能让我们的函数实现各种各样特别的功能，那这个效果就是通过UE4的反射系统来实现的。这看起来确实非常棒，不过同时给UE4的反射系统增添了一点神秘感。我们可能一开始尝试着去找一下这个宏的定义，但是翻了几层发现没有头绪，可能也就懒得再去研究他是怎么实现的了。

其实，所谓反射，是程序在运行时进行自检的一种能力，自检什么呢？我认为就是检查自己的C++类，函数，成员变量，结构体等等（对应起来也就是大家在UE4能看到的UCLASS，UFUNCTON，UPROPERTY，USTRUCT后面还会提到）。

<!--more-->

C++本来是不支持反射的，只有一个基本的RTTI（运行时类型信息）特性，仅能在运行时获取对象的类型信息，无法得到成员变量和函数列表信息。如果我们要对成员变量进行序列化（存档/读档），需要自己写很多辅助的读取和写入方法，非常麻烦。**反射就是，运行时获取成员变量，成员函数等的机制**

UE4在C++编译开始前，使用工具`UnrealHeaderTool`，对C++代码进行预处理，收集出类型和成员等信息，并自动生成相关序列化代码。然后再调用真正的C++编译器，将自动生成的代码与原始代码一并进行编译，生成最终的可执行文件。这个过程类似于Qt的qmake预处理机制。

**UCLASS** :类,告诉UE4生成类的反射数据。类必须派生自 UObject，如果你的类继承自UObject,需要在类名上方加

**UFUCTION**: 函数,使 UCLASS 或 USTRUCT 的类方法可用作 UFUNCTION。UFUNCTION 允许类方法从蓝图中被调用，并在其他资源中用作 RPC。(包裹功能)

**UPROPERT**Y: 成员, 使 UCLASS 或 USTRUCT 的成员变量可用作 UPROPERTY。UPROPERTY 用途广泛。它允许变量被复制、被序列化，并可从蓝图中进行访问。垃圾回收器还使用它们来追踪对 UObject 的引用数。（包裹属性）

**USTRUCT**: 变量

**USTRUCT**：结构体,告诉UE4生成结构体的反射数据（包裹结构）

**UENUM()**：告诉UE4生成枚举的反射数据

**GENERATED_BODY()** - UE4 使用它替代为类型生成的所有必需样板文件代码

那检查这些东西做什么呢？最明显的就是支持蓝图和C++的交互功能，说的更通俗一点，就是可以更自由的控制这些结构，让他在我们想出现的地方出现，让他在我们想使用的地方使用。要知道我们在虚幻4中声明的任意一个类，都是继承于UObject类的，所以他远远不是我们所以为的那个普通的C++类。我们可以使用这个类进行网络复制，执行垃圾回收，让他和蓝图交互等等。而这一切原生的C++是并不支持的，也正是因此虚幻4才构建了一个这样的反射系统。

### 反射一般用于哪些地方？

在UE4里面， 基本上所有的游戏工程的类都需要用到。比如，你用编辑器新建一个类，类的前面会自动添加UCLASS()；新建一个结构体，需要使用USTRUCT()；新建一个枚举变量，需要在前面声明UENUM()；在类的里面，也必须要加上GENERATED_UCLASS_BODY()才行。

如果你的类继承自UObject,的的类名上方需要加入UCASS()宏，同时需要在类体的第一行添加GENERATED_UCLASS_BODY宏，或者GENERATED_BODY()宏，

如果是GENERATED_UCLASS_BODY，则需要手动实现一个带有const FObjectInitializer&参数的构造函数，

如果是GENERATED_BODY()则需要手动实现一个无非参数构造函数

如果你想让你的变量能显示在编辑器里面，想让你的函数可以被蓝图调用或者通过让这个函数实现RPC网络通信功能，或者你想让你的变量被系统自动的回收，这些都离不开反射系统以及这些宏定义。

所以，我们这里起码能认识到，在**网络通信，蓝图交互以及垃圾回收**方面，这与反射系统是密不可分的。

## 序列化

> 序列化是将一个对象变为更已保存的形式，写入到持久储存中，反序列化则相反，从持久储存中读取数据，然后还原原先对象

有了反射功能之后，成员变量的序列化也就更方便了。UE4收集了每个类成员的类型信息，这样存档和读档时，根据名称和类型就可以自动完成了，整个过程不需要人工干预。

需要序列化的成员变量，需要在变量声明的时候在前面加上`UPROPERTY()`宏，宏参数有很多，分别表示变量的详细属性，下面列举一些常用的：

| UPROPERTY参数      | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| EditAnywhere       | 表示该属性可从编辑器内的属性窗口编辑。                       |
| Category           | 定义属性的分类。使用方法: Category=CategoryName. （分类=分类名称） |
| Const              | 编辑器中不能修改该值                                         |
| BlueprintReadOnly  | 在蓝图中只读，不可修该。                                     |
| BlueprintReadWrite | 在蓝图中可读写。                                             |
| BlueprintCallable  | 仅能用于Multicast代理。该代理可被蓝图调用。                  |

VisbleDefaultsOnly： 属性只在archetype蓝图中修改

与`UPROPERTY`对应的还有一个用于修饰函数的宏`UFUNCTION`，该宏常用于描述如何从蓝图中访问C++的函数。

| UFUNCTION参数               | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| BlueprintCallable           | 这种类型的函数，只能在C++中实现和重写。可以理解为蓝图“只读”函数。 |
| BlueprintImplementableEvent | 只能在蓝图中实现的函数。类似于C++的纯虚函数                  |
| BuleprintNativeEvent        | 都可以用（需要提供一个“函数名_Implement”为名字的函数实现，放置与.cpp |
| Exec                        | 控制台调用                                                   |
| BlueprintAssignable         | 暴露该属性来在蓝图中进行分配赋值                             |

##### 实例:

```
UFUNCTION(BlueprintCallable,Category = "Sanwu|TestFuncLib")
```

Category是函数的分类，点击鼠标右键会打开上下文菜单，“SanWu|TestFunLib”的意思是指Say Hello函数放在sanwu分类下的Test Func Lib类目下

### 类修饰符（UCLASS）：

 **abstract（抽象）**：

将类声明为抽象基类，这样会阻止用户实例化这个类。

 **advancedclassdisplay（高级显示）**：

强制类的所有属性仅在Details面板中的高级选项中显示，并且默认为隐藏。

 **autocollapsecategories（自动隐藏分类）**：

取消在父类上使用AutoExpandCategories修饰符的列出分类效果。

 **autoexpandcategories（自动展开分类）**：

为这个类的对象指定应该在虚幻编辑器属性窗口中自动展开的一个或多个类别。要自动展开没有声明类别的变量，请使用声明这个变量的类的名称。

 **blueprintable（蓝图可接受）**：

指定该类为创建蓝图的可接受基类。除非被继承，否则默认值为NotBlueprintable。它由子类继承。

 **blueprinttype（蓝图的类型）**：

此类可作为蓝图中的一种变量类型使用。

 **classgroup（类组）**：

表示虚幻编辑器的Actor浏览器在Actor浏览器中启用GroupView的时候应该在指定的GroupName中包括这个类及其所有子类。

 **collapsecategories（类组）**：

表示这个类的属性不应该归类在虚幻编辑器属性窗口的类别中。这个关键字被传递给子类，但子类可以使用DontCollapseCategories关键字覆盖这个标志。

 **config（配置）**：

表示允许这个类在配置文件(.ini)中存储数据。 如果在这个类中有任何可配置的变量（使用 config 或 globalconfig变量修饰符进行声明），这个修饰符会将这些变量存储在 ( 和 ) 内的指定配置文件中。 将这个标志传递给所有子类，而且无法否定这个标志，但是子类可以通过重新声明config 关键字并指定一个不同的文件名来更改这个.ini文件。 将 IniName 的值添加到游戏名称后面 - 减去 “Game” 部分 - 指定要存储数据的.ini文件的名称（例如，在 UDKGame 指定config(Camera)将会使这个类使用UDKCamera.ini 文件）。 还可以将关键字inherit指定为 IniName ，这样做会使得这个类使用与它的父代相同的配置文件。 默认情况下，会有一些 .ini文件，例如：
Config=Engine: 使用 引擎 配置文件，也就是您的游戏名称后面加上Engine.ini。 例如，UDKGame 的引擎配置文件的名称是UDKEngine.ini。
Config=Editor: 使用 编辑器 配置文件，也就是您的游戏名称后面加上Editor.ini。 例如，UDKGame 的编辑器配置文件的名称是 UDKEditor.ini 。
Config=Game: 使用 游戏 配置文件，也就是您的游戏名称后面加上Game.ini。 例如，UDKGame 的游戏配置文件的名称是 UDKGame.ini 。
Config=Input: 使用 输入 配置文件，也就是您的游戏名称后面加上Input.ini。 例如，UDKGame 的引擎配置文件的名称是UDKInput.ini

 **const（常量）**：

本类中的所有属性及函数均为常量，并应作为常量导出。该标识由子类继承。

 **conversionroot（根转换）**：

根转换限制子类转换，使其仅能转换为其上等级的首个根类的子类。

 **customconstructor（自定义构造函数）**：

防止构造函数声明的自动生成。

 **defaulttoinstanced（默认实例化）**：

该类中所有实例都被视为”已进行实例化”。已进行实例化的类（组件）在构建时被复制。该标识由子类继承。

 **dependson（依赖）**：

表示 ClassName 是在这个类之前进行编译的。 ClassName 必须在同一个（或者是以前的）软件包中指定一个类。多个依赖的类可以通过使用一个单独的DependsOn代码行并且类之间通过逗号分界来指定，或者通过为每个类使用单独的DependsOn行来指定。这在类使用一个在其他类中声明的结构体或枚举变量时非常重要，因为编译器只知道在它已经编译的类中都有些什么。

 **deprecated（弃用）**：

该类已被废弃，并且该类的对象在序列化时将不会被保存。该标识由子类继承。

 **dontautocollapsecategories（取消自动隐藏分类）**：

取消从父类继承的特定目录的AutoCollapseCategories关键字。

 **dontcollapsecategories（取消类组）**：

取消从基类继承的CollapseCatogories关键字。

 **editinlinenew（可新建）**：

表示这个类的对象可以通过虚幻编辑器属性窗口进行创建（默认的操作是只引用可以通过属性窗口进行分配的现有对象）。这个标志将被传递给所有子类，子类可以使用 NotEditInlineNew 关键字覆盖这个标志。

 **hidecategories（隐藏组）**：

为这个类的对象指定应该隐藏在虚幻编辑器属性窗口中的一个或多个类别。要隐藏没有声明类别的变量，请使用声明这个变量的类的名称。这个关键字被传递给子类。

 **hidedropdown（在组合框中隐藏）**：

禁止这个类显示在虚幻编辑器属性窗口组合框中。

 **hidefunctions（隐藏函数）**：

将指定函数隐藏在属性视图中。

 **intrinsic（原生）**：

类直接在C++中进行声明，并且不具有由UnrealHeaderTool生成的样板文件，不要在新类上使用此标识。

 **minimalapi（精简）**：

使得类的类型信息由其他模块导出以供使用。这个类可以被投射，但类的函数无法被调用（除了内联方式）。这样可以改善对不需要其所有功能在其它模块进行调用的类的编译时间。

 **noexport（不导出）**：

表示这个类的声明不应该包含在头文件编译器自动生成的 C++ 头文件中。该 C++ 类声明必须在单独的头文件中手动进行定义。只对 native 类有效。

 **nontransient（取消临时）**：

取消从基类继承的Transient关键字。

 **notblueprintable（非蓝图接受）**：

指定该类不是创建蓝图的可接受基类。除非被继承，否则默认值为NotBlueprintable。它由子类继承。

 **notplaceable（不可放置）**：

否定从基类继承的Placeable关键字。表示在虚幻编辑器中不可以将这个类放置到关卡等位置。

 **perobjectconfig（配置每个对象）**：

这个类的配置信息将会根据对象进行存储，其中每个对象在.ini文件中都有一项，它以这个对象的名字命名，格式为[ObjectName ClassName]。此关键字被传递到子类。

 **placeable（可放置）**：

表明该类可在虚幻编辑器内进行创建并被放置在关卡，UI场景，或蓝图内（取决于该类类型）。此标识被传递到所有的子类中，子类可使用 NotPlaceable 关键字来重载该标识。

 **showcategories（显示组）**：

否定从基类继承的特定分类的 HideCategories 关键字。

 **showfunctions（显示函数）**：

在属性视图中显示指定的函数。

 **transient（显示函数）**：

也就是“属于这个类的对象永远不应该保存在磁盘上”。仅在与本身为非持续的native类的特定种类结合使用时有用。这个关键字被传递给子类，子类可以使用NonTransient关键字覆盖这个标志。

 **within（包含）**：

表明此类的对象不能存在于 ClassName 的实例 之外 。 为了创建这个类的对象，您必须将 ClassName 的实例指定为 Outer 对象。

### 函数修饰符（UFUNCTION）：

 **blueprintauthorityonly（蓝图仅有权限执行）**：

如无网络权限，则该函数将不会从蓝图代码中执行。

 **blueprintcallable（蓝图可执行）**：

该函数可在蓝图或关卡蓝图图表内执行。

 **blueprintcosmetic（蓝图修饰）**：

此函数为修饰函数而且无法运行在专属服务器上。

 **blueprintimplementableevent（蓝图重载事件）**：

此函数可以在蓝图或关卡蓝图图表内进行重载。

 **blueprintnativeevent（同时事件）**：

此函数将由蓝图进行重载，但同时也包含native类的执行。提供一个名称为[FunctionName]_Implementation的函数本体而非[FunctionName];自动生成的代码将包含转换程序,此程序在需要时会调用实施方式。

 **blueprintpure（蓝图可执行纯净）**：

此函数不会以任何方式影响其从属对象，并且可在蓝图或关卡蓝图图表中执行。

 **category（分类）**：

当在蓝图编辑工具中显示时，定义函数的分类。

 **client（客户端）**：

此函数仅在该函数从属对象所从属的客户端上执行。提供一个名称为[FunctionName]_Implementation的函数主体，而不是[FunctionName]; 自动生成的代码将包含一个转换程序来在需要时调用实现方法。

 **customthunk（自定义转换）**：

UnrealHeaderTool（虚幻头文件工具）的代码生成器将不会为此函数生成execFoo转换程序; 可由用户来提供。

 **exec（执行）**：

此函数可从游戏中的控制台中执行。Exec命令仅在特定类中声明时才产生作用。

 **netmulticast（网络所有执行）**：

无论Actor的NetOwner为何值，此函数都会在服务器上被本地执行且将被复制到所有的客户端。

 **reliable（可靠）**：

此函数在网络间进行复制，并会忽略带宽或网络错误而被确保送达。仅在与客户端或服务器共同使用时可用。

 **server（服务器）**：

此函数仅在服务器上执行。提供一个名称为[FunctionName]_Implementation的函数主体，而不是[FunctionName]; 自动生成的代码将包含一个转换程序来在需要时调用实现方法。

 **unreliable（不可靠）**：

此函数在网络间复制，但可能会由于带宽限制或网络错误而传送失败。仅在与客户端或服务器一起使用时有效。

### 元数据修饰符（meta=）：

 **blueprintinternaluseonly（仅内部实现）**：

此函数为内部实现细节，被用来实现另一个函数或节点。它从不在图表中直接展现。

 **blueprintprotected（受保护）**：

此函数仅能在蓝图中针对‘此’实例进行调用。无法针对另一个实例进行调用。

 **blueprintspawnablecomponent（蓝图可生成）**：

如有该类修饰符，此组件类可由蓝图来生成。

 **connotimplementinterfaceinblueprint（蓝图不可实现接口）**：

该接口无法通过蓝图来实现（比如，它仅有不显示的C++成员方式）。

 **deprecatedfunction（启用的函数）**：

该函数被废弃，任何引用它的蓝图都会产生一个编译警告。

 **deprecationmessage（启用信息）**：

提供废弃函数的自定义信息。

 **unsafeduringactorconstruction（构造函数调用不安全）**：

### 属性修饰符（UPROPERTY）：

 **advanceddisplay（高级选项）**：

属性被显示在细节面板的高级下拉框中。

 **assetregistrysearchable（资源可注册）**：

表明此属性及其值将会为任意将其作为成员变量而包含的资源类示例被自动添加到资源注册中。不可用于结构体属性或参数。

 **blueprintassignable（蓝图分配）**：

仅能用于Multicast代理。应显示该属性，以供在蓝图中分配。

 **blueprintcallable（蓝图可调用）**：

仅能用于Multicast代理。应显示该属性，以在蓝图代码中调用。

 **blueprintreadonly（蓝图仅可读）**：

蓝图该属性仅可读取。

 **blueprintreadwrite（蓝图可读写）**：

蓝图该属性仅可读写。

 **category（分类）**：

定义属性的分类。

 **config（可配置）**：

表示该变量将会成为可配置状态。当前值可被保存到ini文件中，并且将在创建时被载入。无法在默认属性中被赋值。只读。

 **const（常量）**：

定义属性为常量。

 **duplicatetransient（复制重置）**：

表示变量值应在任意类型的重复过程中（复制/粘贴，二进制文件复制等）被重置为类默认值。

 **editanywhere（可编辑）**：

表示该属性可从编辑器内的属性窗口编辑。

 **editdefaultsonly（仅对原型编辑）**：

表示该属性可通过属性窗口来编辑，但仅能对原型编辑。

 **editfixedsize（固定大小）**：

仅限于动态数组。这使得用户不能通过UnrealEdtor属性窗口来变更数组的长度。

 **editinline（可编辑引用）**：

通过此修饰符使得用户可编辑UnrealEd的属性查看器中的变量所引用的对象属性。（仅对对象引用可用，包括对象引用数组）。

 **editinstanceonly（仅对实例编辑）**：

表示该属性可通过属性窗口来编辑，但仅能对实例而非原型进行编辑。

 **export（导出）**：

仅对对象属性（或对象数组）有效。表示当对象被复制（复制/粘贴）或导出到T3D时，被分配给该属性的对象应完全作为子对象区块来导出，而不是仅仅输出对象引用本身。

 **globalconfig（全局配置）**：

类似于config修饰符，区别是您不能在子类中重载它。无法在默认属性中被赋值。 只读。

 **instanced（实例化）**：

仅能用于对象属性。当此类的实例被创建时，它会被赋予一个默认分配给此变量的对象的独特拷贝。用于对在类默认属性中定义的子对象进行实例化。类似EditInline和Export修饰符。

 **interp（演出）**：

表示该值可由Matinee的浮点或向量属性轨迹来随时间驱动。

 **localized（本地）**：

此变量的值将定义本地值。最常用于字符串。只读。

 **native（原生）**：

属性为native:C++代码负责对其序列化并显示给GC。

 **noclear（非空）**：

防止该对象引用在编辑器中被设置为None.隐藏编辑器的清除（以及浏览）按钮。

 **noexport（不可导出）**：

仅对native类有效。此变量不应被包含在自动生成的类声明中。

 **nontransactional（不可撤销重做）**：

表示变更为此变量值将不会被包含在编辑器的撤消/重做历史中。

 **ref（参考）**：

该值在函数调用后被复制出来。仅在函数参数声明中有效。

 **replicated（网络复制）**：

此变量应通过网络进行复制。

 **replicatedusing（网络复制执行）**：

此变量应通过网络进行复制，在其接受到 Callback 函数后执行。

 **repretry（网络重复复制）**：

仅用于结构体属性。如无法被完全发送，请重试复制此属性（例如，对象引用尚无法通过节点网络来进行序列化）。对于简单引用来说，这是一个默认值，但对结构体来说，由于带宽消耗，很多情况下我们不需要。所以除非此标识被定义，否则其会被禁用。

 **savegame（保存游戏）**：

 **serializetext（序列化为文本）**：

Native属性应以文本形式进行序列化（导入文本，导出文本）。

 **simpledisplay（显示）**：

使属性在细节面板中默认为可见。

 **transient（临时）**：

该属性为临时属性。不应被保存，在载入时会被填零。

 **visibleanywhere（全可见）**：

表示该属性在属性窗口中可见，但根本无法被编辑。

 **visibledifaultsonly（默认中可见）**：

表示该属性仅在原型的属性窗口中可见，但无法被编辑。

 **visibleinstanceonly（仅实例可见）**：

表示该属性仅在实例的属性窗口中可见，但对原型则不行，并且无法被编辑。

### 结构体修饰符（USTRUCT）：

 **atomic（单元）**：

意味着这个struct要一直作为一个单独的单元进行序列化。

 **blueprinttype（蓝图类型）**：

将此结构体作为用于蓝图中变量的类型。

 **immutable（不可变）**：

仅可在Object.h中使用，而且正逐步淘汰，请不要在新的结构体上使用！。

 **noexport（不导出）**：

不会为该类创建自动生成的代码；标头只用于解析元数据。