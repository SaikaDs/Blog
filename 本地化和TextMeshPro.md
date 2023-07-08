摘要
> 本地化插件(Localization)的基本、拓展使用，以及解决TextMeshPro用一段时间后无法显示新的文字（显示为口口）的问题
# 字库问题
Unity最先进的文字解决方案TextMeshPro能同时满足在UI和场景中（自带一个Mesh）显示文字。它需要从普通字体文件(.ttf)中创建一个SDF文件作为TextMeshPro的字体。有两种创建方式：
1. 右键ttf文件->Create->TextMeshPro->Font Asset
用该方式创建的字体文件只包含项目当前使用过的文字，后续输入新的未使用过的文字则会报错而显示口口。
2. Window->TextMeshPro->Font Asset Creator，在弹出的面板中选择ttf文件，Character Set选择Characters from File，然后置入一个自己编写的字库txt文件，txt里写满自己需要的所有字即可。因为暂时无法预测会用到哪些字，所以从网上下载了常用字库。这样生成出来的字体文件很大，有100多m

能看出TextMeshPro与旧版文字系统不同，它的每个Asset是即时生成了所有文字的贴图，将它们与unicode编码一一对应，并且开放出来，用户可查看每个编码所对应的文字图像，甚至可以修改它们
<center>
<img src="images/%E6%9C%AC%E5%9C%B0%E5%8C%96%E5%92%8CTextMeshPro/2.png" width=400>
<div>文字图形编码对应表</div>
</center>
<center>
<img src="images/%E6%9C%AC%E5%9C%B0%E5%8C%96%E5%92%8CTextMeshPro/3.png" width=400>
<div>生成的贴图</div>
</center>

这个特性在带来强大的自由度的同时，也存在一个不方便的地方，那就是我们需要自行管理游戏中会用到的所有文字。如果用常用字库，未免有些冗余。最理想的办法就是有个记录游戏中会用到的所有文字的功能，并将这些文字导出作为字库。

**本地化插件（Localization）正好有这个功能**

# 本地化插件
本地化插件提供了```多语言表```的功能，可以指定一个key对应的各种语言的文本，在游戏中使用文本的地方填入key而不是文本，运行时插件会根据当前语言设定找到对应语言的文本来显示。此外，该插件还提供了将多语言表导出为字库文件的功能。
Window->Package Manager->Packages: Unity Registry->搜索并安装官方插件Localization

## 基本用法
[参考来源](https://www.yii666.com/blog/377464.html?action=onAll)
1. Edit > Project Settings > Localization
2. Locale Generator中选择要使用的语言
3. Window > Asset Management > Localization Tables->New Table Collection新建一个表格，注意勾选需要用到的语言，然后在Edit Table Collection中编写内容
4. 在任意TextMeshPro组件上右键->Localization，在自动添加的Localize String Event中的String Reference中选择需要用到的文本的key

## 问题：在场景中的TextMeshPro没有Localization这个选项
问题的根源在于TextMeshPro在UI中和场景中应用的类是完全不同的两个类。  
### 在UI中
我们可以在任何一个Canvas上右键->UI->Text - TextMeshPro创建一个对象，可以看到创建出来的对象上的组件叫TextMeshPro - Text (UI)，并同时带有一个Canvas Renderer。
<center>
<img src="images/%E6%9C%AC%E5%9C%B0%E5%8C%96%E5%92%8CTextMeshPro/4.png" width=400>
</center>

并且，该组件实际的类名叫做```TextMeshProUGUI```，在脚本中要获取它也需要用这个类名。  

### 在场景中
我的做法是在一个空物体上直接添加组件TextMeshPro - Text
<center>
<img src="images/%E6%9C%AC%E5%9C%B0%E5%8C%96%E5%92%8CTextMeshPro/5.png" width=250>
</center>

> 至于为什么不添加TextMeshPro - Text (UI)，这是Unity的UI机制问题，Unity中的UI渲染和普通2d精灵的渲染是在完全不同的两个场景中做的，要让UI对象跟随场景中的精灵移动，（比如怪物头上的血条、物体头上的提示文字等），需要自己做好映射才行，比较麻烦。

添加TextMeshPro - Text后，会附带添加一个Mesh Renderer，可以看出文字就是渲染在这个Mesh上，以使它得以出现在场景中。于是我们就可以像对待一个普通场景GameObject一样使用这个文字对象了。
<center>
<img src="images/%E6%9C%AC%E5%9C%B0%E5%8C%96%E5%92%8CTextMeshPro/6.png" width=500>
<div>TextMeshPro - Text</div>
</center>
<center>
<img src="images/%E6%9C%AC%E5%9C%B0%E5%8C%96%E5%92%8CTextMeshPro/7.png" width=500>
<div>TextMeshPro - Text (UI)</div>
</center>

该组件实际的类名叫做```TextMeshPro```，在脚本中要获取它也需要用这个类名。

### 回到问题中来
通过检查本地化插件源码，我们可以看到它只对TextMeshProUGUI做了右键菜单支持
<center>
<img src="images/%E6%9C%AC%E5%9C%B0%E5%8C%96%E5%92%8CTextMeshPro/8.png" width=500>
</center>
场景文本对象中的TextMeshPro并不是TextMeshProUGUI，所以才没有Localization这个选项。  

这其实并不是什么大问题，我们可以手动添加Localize String Event这个组件到场景文本对象中，只是需要多一步操作：在Update String (String)中添加一项，把本Object拖入，下拉菜单中选择TextMeshPro.text。然后多语言文本就会应用到TextMeshPro的text上了。至于右键菜单中的Localization选项，它只是把```添加Localize String Event组件，选择Object，选择目标TextMeshProUGUI.text```这系列操作集成了一下，做成了一键功能。

<center>
<img src="images/%E6%9C%AC%E5%9C%B0%E5%8C%96%E5%92%8CTextMeshPro/9.png" width=500>
</center>

### 如果我嫌这麻烦，想要在TextMeshPro也加上右键菜单的Localize选项呢
很简单，我们只要仿照插件里的右键菜单支持代码也写一个脚本就可以了。插件本身是只读的，我们可以吧LocalizeComponent_TMP复制出来到我们项目源码里，稍微改动一下，把里面的TextMeshProUGUI类偷换成TextMeshPro即可。
代码就贴在这里了
```C#
using TMPro;
using UnityEngine;
using UnityEngine.Events;
using UnityEngine.Localization;
using UnityEngine.Localization.Components;

/// <summary>
/// 给TextMeshPro(非TextMeshProUGUI)加上右键添加Localization菜单按钮
/// </summary>
namespace UnityEditor.Localization.Plugins.TMPro
{
    [InitializeOnLoad]
    internal static class LocalizeComponent_TMPro2
    {
        //static LocalizeComponent_TMPro()  这段不知道什么作用，先注释了，暂时没有影响
        //{
        //    // Register known driven properties
        //    LocalizationPropertyDriver.UnityEventDrivenPropertiesLookup[(typeof(TextMeshProUGUI), "set_text")] = "m_text";
        //}

        [MenuItem("CONTEXT/TextMeshPro/Localize")]  //改动处
        static void LocalizeTMProText(MenuCommand command)
        {
            var target = command.context as TextMeshPro;
            SetupForLocalization(target);
        }

        public static MonoBehaviour SetupForLocalization(TextMeshPro target)  //改动处
        {
            var comp = Undo.AddComponent(target.gameObject, typeof(LocalizeStringEvent)) as LocalizeStringEvent;
            var setStringMethod = target.GetType().GetProperty("text").GetSetMethod();
            var methodDelegate = System.Delegate.CreateDelegate(typeof(UnityAction<string>), target, setStringMethod) as UnityAction<string>;
            Events.UnityEventTools.AddPersistentListener(comp.OnUpdateString, methodDelegate);
            comp.OnUpdateString.SetPersistentListenerState(0, UnityEventCallState.EditorAndRuntime);

            const int kMatchThreshold = 5;
            var foundKey = LocalizationEditorSettings.FindSimilarKey(target.text);
            if (foundKey.collection != null && foundKey.matchDistance < kMatchThreshold)
            {
                comp.StringReference.TableEntryReference = foundKey.entry.Id;
                comp.StringReference.TableReference = foundKey.collection.TableCollectionNameReference;
            }

            return comp;
        }
    }
}

```

### Localize组件之外使用多语言key
我想要配置一个对话列表，想尽可能地在自定义对话脚本中配置多语言中的key而不是直接写文本，这里可以直接把Localize插件中的LocalizeString类抠出来用。
上述String Reference选项的本质是一个LocalizeString类型对象，可以在自定义脚本中设置一个List&#60;LocalizeString&#62;，显示在面板中如下。（组件化做的太好了，非常方便强大）
<center>
<img src="images/%E6%9C%AC%E5%9C%B0%E5%8C%96%E5%92%8CTextMeshPro/10.png" width=350>
</center>

于是就可以愉快地通过下拉菜单选择key了。  
在脚本中，通过以下方式获取LocalizeString对象在当前语言设置下的实际text
```C#
string text = localizeString.GetLocalizedString();
```

- 未解决的问题
用这种方式每次下拉菜单选择key时，编辑器都会报错Unsupported type LocalizedString，暂时没找到原因，但不影响使用。

### 字库导出
等开发到合适的时间点，即可在Localization Tables面板中->右上角三点按钮->Export->Character Set  
导出只用于本项目的精简字库，并用来重新生成字体文件，以降低资源大小。

### 拓展
除了文字，本地化插件还提供了在不同语言设置下展示不同的图片、音效等，可以说是很全面了。等后续探索
