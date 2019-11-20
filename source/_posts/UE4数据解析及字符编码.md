---
title: UE4数据解析及字符编码
date: 2019-06-30 15:43:21
tags: [UE4,数据解析,字符编码，XML，JSON]
categories:  UE4
---

最近在实现客户自定义配置数据功能方面，写了许多的数据解析代码，我就将其中的数据解析文件进行了整理，并将数据解析中设计的字符编码深刻的理解了一番

<!--more-->

### 字符编码变换

> 用TinyXml加载一个xml文档时，由于xml文档是UTF-8编码的，而UE4默认使用的是根据文档的编码方式来加载，在操作过程中需要进行编码转换

#### 了解

##### ASCII码

使用1个byte，表示出英文中所有需要的字符（缺点：非英文的字符都无法表示）

##### UNICODE

是一个字符集，规定了不同的字符对应于一个唯一的整数，平时所说的使用UNICODE编码其实说的是UFT16编码（顾名思义就是用16位来表示一个字符，有65536个字符），UTF8、UTF16和UFT32则是基于UNICODE字符集的三种编码方式

##### TCHAR

TCHAR类型就是通过宏对char和wchar_t的封装，将其中的操作进行了统一。可根据当前平台情况选择对应的类型。_T修饰的字符串常量同理，根据是否定义的UNICODE宏，分别表示""或L""

#### 转换

##### char 转 TCHAR

```
TCHAR* outTchar = new TCHAR[iLength + 1];
const char *outTchar1 = displayNameNode->GetText();
MultiByteToWideChar(CP_UTF8, 0, outTchar1, strlen(outTchar1) + 1, outTchar, iLength);
```

##### TCHAR转char

```
int32 iLength = WideCharToMultiByte(CP_UTF8, 0, *_XmlPath, -1, NULL, 0, NULL, NULL);
char* path = new char[iLength + 1];
WideCharToMultiByte(CP_UTF8, 0, *_XmlPath, -1, path, iLength, NULL, NULL);
```

> 注意：使用wideChartoMultiByte函数时，需要包含stringappiset.h头文件，而使用此头文件有需要包含windows.h头文件

##### 而在UE4中有一个封装好的宏用于char与TCHAR的转换

- TCHAR_TO_ANSI - 将引擎字符串（TCHAR*）转换为 ANSI 字符串。
- ANSI_TO_TCHAR - 将 ANSI 字符串转换为引擎字符串（TCHAR*）。

> 注意：这些宏声明的对象的生命周期非常短。它们将被用作函数的参数。无法将一个变量指定到转换后的字符串内容，因为对象将处于作用域之外，字符串将被释放。
>
> 传入的参数必须是一个固有字符串，因为参数被类型转换为指针。如传入的是 TCHAR 而非 TCHAR*，编译后运行时将出现崩溃。

#### 用法：

**SomeApi(TCHAR_TO_ANSI(SomeUnicodeString));**

```
UTF8_TO_TCHAR(outTchar)
注意：
这个宏的声明周期很短所以需要调用完直接赋值
```

##### FString转为TCHAR *(TCHAR与FString基本都能自动隐式转换)

```
Const FString SceneName;
const TCHAR *hc = *SceneName;
```

### 解析json文件

##### 1. json数据格式

```
{
	{
		“Cultrue": "ZH"
	},
	{
        "MusicVolume: 0.3
	},
	{
        "SoundVolume": 0.3
	},
	{
        "ReCordData": [
            {
                "0": "Defaule"
            },
            {
                "1": "Default"
            }
        ]
	}
}
```

##### 解析Json数据示例

```
// 包含Include"Json.h"
void SlAiJsonHandle::RecordDataJsonRead(FStingp& Culture,float& MusicVolume,float& SoundVolume,TArray<FSting>&RecordDataList)
{
RecordDataFileName = FString("RecordData.json")  //要读取的文件名
Relativepath = FString("Res/ConfigData/");  //文件地址
FString JsonValue"    //读取的保存数据
/**将路径指定的文件读取到一个FString类型的二进制的数组中；*/
LoadStringFroFromFile(RecordDataFileName,RelativePath,JsonValue);  
TArray<TSharePtr<FJsonValue>> JsonParsed;  //解析后保存的数组
/**读取为JsonReader格式*/
TShardRef<TJsonReader<TCHAR>> JsonReader = TJsonReaderFactory<TCHAR>::Create(Jsonvalue);
/**解析数据*/
if(FJsonSerializer::Deserialize(JsonReader,JsonParsed)){
//  根据键值对去获取第一个数据
    CulTure = JsonParsed[0]->AsObject(0)o->GetStringField(FString("Culture"));
    //获取数组数据
    TArray<TSharedPtr<FJsonValue>> RecordDataArray = JsonParsed[3]->AsObject()->GetArrayField(FString("RecordData"));
    // 遍历数组数据
    for(int i =  0; i < RecordDataArray.Num();++i){
        Fstring RecordDataName = RecordDataArray[i]->AsObject()->GetStaringField(Fstring::FromInt(i));   //夺取数据
        RecordDataList.Add(RecordDataName); // 保存数据
    }
}else{
    //解析不成功
    UE_LOG(LogSimpleAPP,Warning,TEXT("Culture = %s"),*Culture);
}
}
```

### 解析XML文件

#### 1.使用UE4自带的XmlParser库进行读写

##### XML数据

```
<?xml version="1.0" encoding="UTF-8"?>
<Setting> // 每个<>都是一个节点（node
	<MaxFPS id="wz9"> //id是属性（attribute)
	<Setres>1440x720w</Setres> //一个完整的节点是元素(element)
	<ScreenPercentage>180</ScreenPercentage>
	</MaxFPS>
</Setting>
```

##### 解析

```
//1.模块中包含XmlParser
//2.包含头文件
#include "Runtime/XmlParser/Public/XmlParser.h"
#include "Runtime/XmlParser/Public/XmlFile.h"
#include "Runtime/XmlParser/Public/XmlNode.h"
#include "Runtime/XmlParser/Public/FastXml.h"
bool UReadXMLBPLibrary::ReadXML(FString& MaxFPS, FString& SetRes, FString& ScreenPercentage)
{
	if (FPlatformFileManager::Get().GetPlatformFile().FileExists(*(FPaths::GamePluginsDir()/TEXT("ReadXML/XmlFiles/Setting.xml"))))  //判断是否有此文件
	{
		FXmlFile* file = new FXmlFile(FPaths::GamePluginsDir() + "ReadXML/XMLFiles/Setting.xml");     // 获取文件地址
		FXmlNode* RootNode = file->GetRootNode();    	//获取文件根节点
		FXmlNode *MaxFPSNode = RootNode->FindChildNode("MaxFPS");   // 寻找数据子节点
		MaxFPS = *MaxFPSNode->GetContent();         // 获取其内容

		FXmlNode* SetresNode = RootNode->FindChildNode("Setres");
		SetRes = *SetresNode->GetContent();

		FXmlNode* ScreenpercentageNode = RootNode->FindChildNode("ScreenPercentage");
		ScreenPercentage = *ScreenpercentageNode->GetContent();
		return true;
	}
	else
	{
		return false;
	}

}
```

#### 2.使用C++常用的tinyxml库进行读写

> 在工作中详细的使用了tinyxml库进行了xml读写，所以详细的说明

##### tinyxml使用前的配置步骤

1. 下载tinyxml库
2. 将tinyxml.h、tinystr.h、tinystr.cpp、tinyxml.cpp、tinyxmlerror.cpp、tinyxmlparser.cpp代码拖进同一种

注意：在UE4种使用要讲thinyxml.cpp种#include "thinyxml.h"放在第一个引用，不然会报错

##### 使用了解

节点：一种对文档结构的描述对象
元素：对文档某一个数据块的描述
文本是指没有孩子的节点

##### 节点类可以转换成元素对象。

`TiXmlElement * pElement = pNode->ToElement();`

> 那什么时候需要转换成元素呢？  当你需要元素的一些属性值是就需要转换了。

##### 元素访问孩子的函数：

`FirstChildElement() 返回当前元素的孩子元素`
`NextSiblingElement() 返回当前元素的同级元素`

##### 节点访问节点孩子的函数：

``返回当前节点的孩子节点: FirstChild()`

`返回当前节点的同级下一个节点: NextSibing()`

> 元素访问和节点访问在一般的访问下没有区别，两者都可以访问的孩子,对于一些特殊的情况下才需要区分。比如你要访问属性时，就需要用元素来找到第一个属性值。

##### 链接节点到上一节点

`RootNode->LinkEndChild(SceneNode);`

##### 链接文本到节点上

`MissionNodes->LinkEndChild(MissionsText);`

##### 对于遍历一个xml文档时，思路一般是这样的：

1. 载入一个xml
2. 获得根元素（不是根节点）
3. 循环访问每一个根元素的子元素
4. 对每一个子元素进行解析。
   获取子元素的类型
   循环遍历子元素的下一级元素
5. 递归调用第四步

##### 如果定位一个节点

   唯一确定一个节点的方法是根据节点名，属性名，属性值
   1 根据xml的遍历思想找到与给定节点名一样的节点
   2 如果这个节点有属性并且属性名和值与给定的一致，说明找到了。
  3 如果没有一致的，说明解析失败。
   4 如果没有给定属性名和值，则可以默认只查找与节点名一致的节点。

##### 指针的 new和释放。

TinyXml已经帮我们把指针分配的内存进行了管理，在析构函数中进行了处理，我们不需要处理new出来的指针

##### 读取示例

```
#include "ReadXML.h"
#include "Engine/Engine.h"
#include <windows.h>
#include <stringapiset.h>
#include "Paths.h"
#include "PlatformFilemanager.h"

bool UReadXML::ReadUnitXMLFile(TArray<FString> &str,
{
	// 工程的相对路径+文件名称
	FString _XmlPath = FPaths::GameSourceDir() + "Battle.xml";
	// 将TCHAR转换char 并转UTF-8编码 
	int32 iLength = WideCharToMultiByte(CP_UTF8, 0, *_XmlPath, -1, NULL, 0, NULL, NULL);
	char* path = new char[iLength + 1];
	WideCharToMultiByte(CP_UTF8, 0, *_XmlPath, -1, path, iLength, NULL, NULL);

	TiXmlDocument *myDocument = new TiXmlDocument();
	if (myDocument->LoadFile(path))
	{
		// 获取跟元素
		TiXmlElement *RootElement = myDocument->RootElement();
		// 遍历节点
		TiXmlElement *lpMapNode = NULL;
		for (lpMapNode = RootElement->FirstChildElement(); lpMapNode != NULL; lpMapNode = lpMapNode->NextSiblingElement())
		{
			GEngine->AddOnScreenDebugMessage(-1, 10, FColor::Red, "read failed");
			const char* outchar = lpMapNode->Attribute("id");

			TCHAR* outTchar = new TCHAR[iLength + 1];
			MultiByteToWideChar(CP_UTF8, 0, outchar, strlen(outchar) + 1, outTchar, iLength);
			str.Push(outTchar);
		}
	}
	else
	{
		return false;
	}
	return true;
}

#pragma once
#include "tinystr.h"
#include "tinyxml.h"
#include "CoreMinimal.h"
#include "Kismet/BlueprintFunctionLibrary.h"
#include "ReadXML.generated.h"

UCLASS()
class DCXVEHICLEDEMO_API UReadXML : public UBlueprintFunctionLibrary
{
	GENERATED_BODY()
public:
	UFUNCTION(BlueprintCallable, Category = "YPFUNC")
		static bool ReadUnitXMLFile(TArray<FString> &str, TArray<FString> &displayName, TArray<FString> &mission, TArray<UTexture2D*> &icon, TArray<FString> &inconPrefab, TArray<FString> &category, TArray<FString> &load, TArray<FString> &temIndex, TArray<FString> &valid, TArray<FString> &elements);
	
};
```

#### 注意

> 打包后的文件地址，与在开发时的地址是不同的，所以需要根据所处的状态调整读取xml文件的方法

##### 读入示例

```
bool UReadXML::SaveMapData(const FString SceneName,const FString Mission,const FString Scenes,const FString Missions)
{
	TiXmlDocument *doc = new TiXmlDocument();   // 创建xml文档对象
	// 创建描述
	TiXmlDeclaration *pDeclaration = new TiXmlDeclaration("1.0", "UTF-8", "");
	// 链接到文档
	doc->LinkEndChild(pDeclaration);
	// 创建根节点
	TiXmlElement *RootNode = new TiXmlElement("Numbers");
	doc->LinkEndChild(RootNode);
	// 创建子节点
	TiXmlElement *SceneNameNode = new TiXmlElement("SceneName");
	RootNode->LinkEndChild(SceneNameNode);  //将节点链接到根节点上
	TiXmlText *SceneNameText = new TiXmlText(TCHAR_TO_UTF8(*SceneName)); // 创建Text内容
	SceneNameNode->LinkEndChild(SceneNameText); // 链接内容到Node

	//Mission
	TiXmlElement *MissionNode = new TiXmlElement("Mission");
	RootNode->LinkEndChild(MissionNode);  //将节点链接到根节点上
	TiXmlText *MissionText = new TiXmlText(TCHAR_TO_UTF8(*Mission)); // 创建Text内容
	MissionNode->LinkEndChild(MissionText); // 链接内容到Node

	// SceneNote 
	TiXmlElement *SceneNode = new TiXmlElement("SceneNote");
	RootNode->LinkEndChild(SceneNode);  //将节点链接到根节点上
	TiXmlText *SceneText = new TiXmlText(TCHAR_TO_UTF8(*Scenes)); // 创建Text内容
	SceneNode->LinkEndChild(SceneText); // 链接内容到Node

	// SceneNote 
	TiXmlElement *MissionNodes = new TiXmlElement("MissionNode");
	RootNode->LinkEndChild(MissionNodes);  //将节点链接到根节点上
	TiXmlText *MissionsText = new TiXmlText(TCHAR_TO_UTF8(*Missions)); // 创建Text内容
	MissionNodes->LinkEndChild(MissionsText); // 链接内容到Node

	// list
	TiXmlElement *ListNode = new TiXmlElement("list");
	RootNode->LinkEndChild(ListNode);

	// num
	TiXmlElement *NumNode = new TiXmlElement("num");
	NumNode->SetAttribute("i", "0");

	// id
	TiXmlElement *idNode = new TiXmlElement("id");
	NumNode->LinkEndChild(idNode);
	TiXmlText *idText = new TiXmlText(TCHAR_TO_UTF8(*Missions)); // 创建Text内容
	idNode->LinkEndChild(idText); // 链接内容到Node
	
	FString XmlPath = FPaths::GameDir()+ "SaveMapData";
	int32 iLength = WideCharToMultiByte(CP_UTF8, 0, *XmlPath, -1, NULL, 0, NULL, NULL);
	char* path = new char[iLength + 1];
	//path = _XmlPath;
	WideCharToMultiByte(CP_UTF8, 0, *XmlPath, -1, path, iLength, NULL, NULL);
	doc->SaveFile(path);
	return true; 
}
```

### XML与JSON的区别

#### XML

##### 优点：

1. 格式统一
2. 因为是标准通用标记语言，所以数据共享比较方便
3. 可读性好

##### 缺点：

1. XML文件庞大，文件格式复杂，传输占带宽
2. 服务器端和客户端都需要花费大量代码来解析XML，导致服务器端和客户端代码变得异常复杂且不易维护
3. 客户端不同浏览器之间解析XML的方式不一致，需要重复编写很多代码
4. 服务器端和客户端解析XML花费较多的资源和时间

#### JSON

##### 优点：

1. 数据格式比较简单，易于读写，格式都是压缩的，占用带宽小
2. 易于解析，客户端JavaScript可以简单的通过eval()进行JSON数据的读取
3. 支持多种语言，JSON格式能直接为服务器端代码使用，大大简化了服务器端和客户端的代码开发量，且完成任务不变，并且易于维护

##### 缺点：

1. 通用性不够
2. 使用不够广泛
3. 数据描述性较差

### 相关博客

XML使用：<https://blog.csdn.net/lgstudyvc/article/details/77859919>

TinyXml节点查找及修改:<https://www.cnblogs.com/lyggqm/p/4565749.html>