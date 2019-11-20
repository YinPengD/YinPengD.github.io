---
title: UE4插件开发
date: 2018-12-27 16:33:21
tags: [UE4,插件,XML]
categories:  UE4
---

很多虚幻引擎中的子系统都被设计为具有可扩展的能力，能够在不直接修改引擎代码的前提下，插件为引擎添加完整独立的新功能， 或者修改引擎中内建的功能。可以为引擎创建新的文件类型，为编辑器添加新的菜单项或者工具栏按钮， 甚至增加一个完整的全新功能模块或者编辑器的子模块

<!--more-->



##### 文件目录

| 插件类型 | 搜索路径                                   |
| -------- | ------------------------------------------ |
| 引擎插件 | /UE4 root/Engine/Plugins/My Engine Plugin/ |
| 游戏插件 | /My project/Plugins/My Game Plugin/        |

虚幻引擎通过查找 .uplugin 文件来定位插件。我们把这些文件称为插件描述器，这些文件是文本文件，提供了插件的基础信息。 虚幻引擎、编辑器以及编译工具（UBT）在运行时，会自动发现并加载插件描述器。

如果插件具有 Source 目录的模块（以及 *.Build.cs 文件），插件的代码会自动添加到生成的 C++ 项目中，这样便能更容易的一边开发游戏一边开发插件。 每当编译游戏项目是，任何带有源码的插件也会根据游戏的依赖关系一并被编译。

引擎插件有一个特殊的要求：引擎的代码模块必须不能够有任何静态链接依赖到引擎插件的模块库上。 也就是说，引擎的插件必须要保持不被引擎依赖的独立性——插件模块必须不能够成为“被引擎依赖的模块”。这是一个设计哲学上的选择， 这样能够让引擎在这些插件不可用的时候仍然能够工作正常。

##### 插件描述器配置‘

```
{
	"FileVersion": 3,
	"Version": 1,
	"VersionName": "1.0",
	"FriendlyName": "ReadXML",
	"Description": "ToReaderXML",
	"Category": "Other",
	"CreatedBy": "RocYing",
	"CreatedByURL": "",
	"DocsURL": "",
	"MarketplaceURL": "",
	"SupportURL": "",
	"CanContainContent": true,
	"IsBetaVersion": false,
	"Installed": false,
	"Modules": [
		{
			"Name": "ReadXML",
			"Type": "Runtime",
			"LoadingPhase": "PreLoadingScreen"     //插件在什么时候会被加载
		}
	]
}
```

##### Type这里的类型设置决定了该插件适合于那种类型的应用程序加载,有以下几项：

Runtime 的模块在无论何时都会被加载，哪怕是在最终发行的游戏版本中也会。

Developer 的模块只会在 Development 运行时或者编辑器版本中才会被加载，并不在在最终发行版本中加载。Editor 模块只会随着 Editor 的启动被加载。

插件也可以使用几种不同类型的组合来达到所需要的目的。

##### 示例：XML解析插件

```
bool UReadXMLBPLibrary::ReadXML(FString& MaxFPS, FString& SetRes, FString& ScreenPercentage)
{
	if (FPlatformFileManager::Get().GetPlatformFile().FileExists(*(FPaths::GamePluginsDir()/TEXT("ReadXML/XmlFiles/Setting.xml"))))  //判断是否有此文件
	{
		FXmlFile* file = new FXmlFile(FPaths::GamePluginsDir() + "ReadXML/XMLFiles/Setting.xml");     // 获取文件地址
		FXmlNode* RootNode = file->GetRootNode();    	//获取文件根节点
		FXmlNode *MaxFPSNode = RootNode->FindChildNode("MaxFPS");   // 寻找数据子节点
		MaxFPS = *MaxFPSNode->GetContent();         // 获取其内容
		// 获取Setres字段内容
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

然后就能在所有蓝图中调用了

#### 注意：

1.在插件中使用了什么模块在在.Build.cs包含此模块

```
PublicDependencyModuleNams.AddRange(
new string[]
{
    "Core",
    "XmlParser"
}
)
```

2.插件系统是独立于本体的系统

3.在游戏中创建了SCompondWidget类将其移动插件中时需要将API模块说明符删除 “模块名”_API