---
title: Unity AssetBundle/Addressables 笔记 1 - 环境部署/基础原理
date: 2026-3-2 16:11:50 +0800
tags: [note,unity]
typora-root-url: ..
---

笔记部分内容来自AI生成， 尚未完全校验，仅供参考。



## Phase 0 - 环境部署

* Unity版本：2022.3 LTS
* Addressables版本：1.22.3



**准备环境：**

创建空白项目，安装Addressables包。



**工具：**

Window → Analysis → Profiler （性能检查）

Package Manager → Memory Profiler （内存泄露检查）

Window → Asset Management → Addressable Assets → Analyze （资源分析，自动扫描冗余/重复资源、依赖问题） | [文档](https://docs.unity3d.com/Packages/com.unity.addressables@1.1/manual/AddressableAssetsAnalyze.html)

官方样本仓库 → [Addressables-Sample](https://github.com/Unity-Technologies/Addressables-Sample)



**完成安装后：**

打开Addressables Groups窗口：Window → Asset Management → Addressables → Groups

点击 Create Addressables Settings ，查看基础设置界面。



![](img/miscs/11.png)



## Phase 1 - AB包基础原理



### 基础

[基础介绍 →](https://docs.unity3d.com/2022.3/Documentation/Manual/AssetBundlesIntro.html)



**一句话定义：**

AssetBundle 是一种归档文件（archive file），里面存放的是平台特定的、非代码类资源（例如模型、贴图、预制体、音频片段、甚至整张场景），Unity 可以在运行时动态加载这些资源。



**解决痛点：**

安装包太大、热更新、平台差异、内存爆炸。



**主要用途与优势（为什么游戏要用 AssetBundle）：**

* 做**可下载内容（DLC）**最常见的方式

* 大幅**缩小初始安装包体积**（玩家先下载一个小本体，后续资源按需下载）

* 可以针对不同平台加载**优化过的资源**（例如高配/低配、iOS/Android 不同纹理压缩）

* 有效**减轻运行时内存压力**（不用把所有资源都提前打包进主包）

* 支持**资源之间互相依赖**（比如 Bundle A 里的材质可以引用 Bundle B 里的贴图）



**AssetBundle 里到底装了什么？**

一个 AssetBundle 文件（也叫 AssetBundle archive）内部主要包含两种数据：

* **序列化文件（serialized file）：**把你标记进 Bundle 的所有资源拆成一个个独立对象，全部写进这一个文件。

* **资源文件（resource files）：**某些特殊资源（主要是**纹理**和**音频**）会把原始二进制数据单独拆出来存放，方便 Unity 在子线程高效加载。

另外，“AssetBundle” 这个词在不同语境下有两种含义：

* **文件层面：**磁盘上的 .assetbundle / .unity3d 文件（归档容器）
* **代码层面：**运行时通过 AssetBundle.LoadFrom... 得到的那个 AssetBundle 对象，它内部有一张“路径 → 资源”的映射表



**关于脚本与 ScriptableObject 的重要说明。**

AssetBundle **可以** 包含 ScriptableObject（或其他代码类实例）的**序列化数据**，但**类定义本身**必须已经编译在项目的 Assembly 里。



加载时 Unity 会：

1. 找到匹配的类定义
2. 创建实例
3. 把序列化数据填充到字段里



**这意味着：**只要不修改类的结构（字段、命名空间等），你就可以通过 AssetBundle 持续新增游戏内容（新武器、新道具、新关卡配置等），非常适合长期运营的游戏。



### 快速对比

#### 什么时候适合用 AssetBundle？

| 需求                      | 是否推荐 AssetBundle | 推荐替代方案           |
| ------------------------- | -------------------- | ---------------------- |
| 游戏体积非常大（>几百MB） | 非常推荐             | —                      |
| 需要分章节/赛季更新内容   | 强烈推荐             | Addressables（更现代） |
| 想做平台差异化优化资源    | 推荐                 | —                      |
| 小型独立游戏，资源很少    | 不推荐               | 直接打进主包就好       |
| 需要频繁热更代码逻辑      | 不行                 | 用其他热更新方案       |



### 小结

AB包 = 平台专属（**Build Target**）压缩档案（非代码资源：模型、贴图、Prefab、场景）。

优点：DLC、减小安装包、按需加载、平台优化。

官方推荐：**不要直接用AssetBundle底层API**，改用Addressables（官方推出的AssetBundle的用户友好封装版，它自动帮你管理依赖、打包、加载）。



### QA

#### QA：平台专属压缩档案？

“平台专属压缩档案”指的是 Unity 构建的 AssetBundle（AB包）本身是平台专属（platform-specific）的，而不是某种特殊的“Unity平台压缩格式”。

这里的 “平台” 对应构建窗口的 Build Target 。



**“平台专属压缩档案”** = AssetBundle 是为特定平台（Android/iOS/Windows 等）单独构建的压缩包，里面的内容经过平台优化，跨平台不能通用。

**压缩方式（LZMA/LZ4）**是文件级别的压缩算法，和“平台专属”无关。



#### QA：为什么 AssetBundle 是平台专属的？

AssetBundle 里存的不是原始资源文件，而是 Unity 在特定 **Build Target**（构建目标平台）下 **序列化 + 平台优化后** 的数据。这些数据包含：

- 纹理的平台专用压缩格式（例如 Android 用 ETC2/ASTC，iOS 用 PVRTC/ASTC，PC 用 DXT/BCn 等）
- Shader 的平台特定变体和编译结果
- 网格、动画、音频等资源的平台专用序列化格式
- 一些平台特有的元数据（如 endianness 大小端、特定指令集等）



所以同一个资源（比如一张贴图），在 Windows、Android、iOS 上打包出来的二进制数据 **完全不同** 。

如果你用 Android 打包的 AB 包去 iOS 上加载，几乎 100% 会崩溃、出现粉色材质、或诡异渲染错误。



官方文档：

> AssetBundles are platform specific, meaning at runtime you can only load AssetBundles built for the target platform.



#### QA：压缩格式（LZMA / LZ4 / Uncompressed）

AB 包文件本身的压缩方式：

- **LZMA**：全包压缩，文件最小，但加载时要整包解压（适合远程下载）
- **LZ4**（Chunk-based）：分块压缩，加载更快，文件稍大（推荐本地或缓存使用）
- **Uncompressed**：不压缩，最大最快

这些压缩选项在打包时选，但 **即使压缩方式相同，不同平台的 AB 包仍然不能互用** 。



#### QA：Addressables与AssetBundle的底层特性？

Addressables 底层就是 AssetBundle，所以它打包出来的内容也是平台专属的。



当你在 Addressables Groups 窗口 **Build** 时：

- 当前 Editor 的 **Build Settings → Platform** 决定了这次打包是给哪个平台的
- 你需要为**每个目标平台**（Android、iOS、Windows、WebGL 等）**分别 Build** 一套 Addressables 内容
- 远程服务器/CDN 上通常要放**多份文件夹**（android/、ios/、windows/ 等）



#### QA：实际项目中怎么处理多平台？

常见做法：

- **文件夹命名约定**：
  remote-android/
  remote-ios/
  remote-windows/
  remote-webgl/
- **Profile + RemoteLoadPath**：在 Addressables Profiles 里为不同平台设置不同的 CDN 路径前缀
- **构建流水线**：用脚本或 CI 依次切换平台 → Build Addressables → 上传对应文件夹
- PC/Mac/Linux Standalone 很多时候可以共用一套（因为序列化格式高度相似），但严格来说还是建议分开（尤其是 Shader 变体多时）



#### QA：“平台专属”最直观的表现？

纹理压缩格式差异（最常见导致跨平台崩溃的原因）：

- **Windows/PC (DirectX)**：DXT1 / DXT5 / BC7（现代常用 BC7）
- **Android**：ETC2（主流）、ASTC（高配推荐）、ETC1（老设备）
- **iOS**：PVRTC（老设备）、ASTC（现代设备推荐）
- **WebGL**：通常 ETC2 或 uncompressed（浏览器限制）



如果跨平台加载，常见症状：

* 粉色材质（Shader 变体不匹配）
* 黑块/绿块纹理（压缩格式不兼容）
* 崩溃（序列化数据 endianness 或对齐问题）



#### QA：序列化文件 vs 资源文件 的实际影响？

* 纹理/音频拆成 resource files → 支持**异步子线程解压加载**（LZ4 特别友好）。

* 如果全塞进 serialized file → 加载时主线程压力更大，适合小 bundle。



#### QA：ScriptableObject 在 AB 中的黄金使用场景？

最适合做**配置表、物品数据库、关卡数据**的热更新载体，因为代码逻辑不动，只更新数据实例。



#### QA：压缩格式推荐？

| 场景                        | 推荐压缩     | 为什么推荐                                      | 文件大小 | 加载速度 | 内存峰值 |
| --------------------------- | ------------ | ----------------------------------------------- | -------- | -------- | -------- |
| 远程下载（DLC/热更）        | LZMA         | 体积最小，下载流量最省（但首次加载慢）          | 最小     | 慢       | 中等     |
| 本地缓存 / StreamingAssets  | LZ4          | 平衡最好：分块解压 + 较快随机访问（官方最推荐） | 中等     | 快       | 低       |
| 极致加载速度（关键UI/角色） | Uncompressed | 最快，但包体大（适合 <10MB 的高频 bundle）      | 最大     | 最快     | 最低     |
| 混合使用                    | 默认 LZ4     | Addressables 默认就是 LZ4，绝大多数项目够用     | —        | —        | —        |

**小提醒：**LZMA 在移动端 StreamingAssets 里非常不推荐（官方明确警告），因为解压会产生临时大文件，IO 爆炸。



#### QA：多平台处理实际操作建议？

* **最简单起步方式**（开发期）
  1. 只针对当前 Editor 平台（通常 Windows）Build Addressables。
  2. Play Mode 选 “Use Existing Build (requires built groups)”。

* **真机多平台流程**（生产期）
  1. 写一个 Editor 菜单脚本，循环切换 BuildTarget（Android / iOS / StandaloneWindows64 等）。
  2. 每次切换后 Build Addressables → 输出到不同文件夹（e.g. ServerData/Android, ServerData/iOS）。
  3. 用 **Addressables Profiles** 的变量（如 {Platform}）自动替换 RemoteLoadPath：
     - 示例：https://cdn.example.com/{Platform}/[BuildTarget]
       - Unity 会自动替换成 Android / iOS 等。

* **Shader 变体问题**（最容易踩的坑）

  - 如果项目 Shader 很多，跨平台 bundle 很容易因为 missing variant 出现粉色。

  - 解决方案：
    - 用 **Shader Variant Collection** 提前收集所有用到的变体。
      - 或在打包前强制 strip（但小心过度 stripping 导致运行时丢失）。
      - Addressables 里可以为不同平台建不同 Group（带 platform 标签）。



### 实例



#### 如何执行打包？



**常见写法：**使用 AssetBundleBuild 数组（灵活，支持指定 bundle 名 + 资源路径）

```csharp
using UnityEditor;
using UnityEngine;
using System.IO;

public class BuildAssetBundlesEditor
{
    // 菜单项：Assets → Build AssetBundles (All)
    [MenuItem("Assets/Build AssetBundles (All)", false, 100)]
    static void BuildAllAssetBundles()
    {
        string outputPath = Path.Combine(Application.dataPath, "AssetBundles");
        
        // 确保输出文件夹存在
        if (!Directory.Exists(outputPath))
        {
            Directory.CreateDirectory(outputPath);
        }

        // 打包选项（推荐组合）
        BuildAssetBundleOptions options = BuildAssetBundleOptions.StrictMode | 
                                          BuildAssetBundleOptions.ChunkBasedCompression;  // LZ4 压缩

        // 当前平台（根据 Build Settings 自动取）
        BuildTarget target = EditorUserBuildSettings.activeBuildTarget;

        // 开始打包所有已标记 AssetBundle 的资源
        // 会自动生成 .manifest 文件和总 manifest
        BuildPipeline.BuildAssetBundles(
            outputPath, 
            options, 
            target
        );

        AssetDatabase.Refresh();  // 刷新 Project 窗口
        Debug.Log($"AssetBundles 打包完成！输出路径：{outputPath}");
    }

    // 菜单项：只打包特定 bundle（更可控，适合分包）
    [MenuItem("Assets/Build Specific AssetBundles", false, 101)]
    static void BuildSpecificAssetBundles()
    {
        string outputPath = Path.Combine(Application.dataPath, "AssetBundles");

        if (!Directory.Exists(outputPath))
            Directory.CreateDirectory(outputPath);

        // 示例：手动指定要打包的 bundle 和里面的资源路径
        AssetBundleBuild[] builds = new AssetBundleBuild[3];

        // Bundle 1: 所有角色相关
        builds[0] = new AssetBundleBuild
        {
            assetBundleName = "characters",                  // bundle 文件名 → characters.unity3d
            assetNames = new[]
            {
                "Assets/Prefabs/Characters/Player.prefab",
                "Assets/Prefabs/Characters/Enemy/Skeleton.prefab",
                "Assets/Materials/Character.mat"
            }
        };

        // Bundle 2: UI 资源
        builds[1] = new AssetBundleBuild
        {
            assetBundleName = "ui",
            assetNames = new[]
            {
                "Assets/UI/Prefabs/MainMenu.prefab",
                "Assets/UI/Sprites/atlas_ui.png",
                "Assets/UI/Fonts/YaHei.asset"
            }
        };

        // Bundle 3: 场景（场景也可以打成 bundle）
        builds[2] = new AssetBundleBuild
        {
            assetBundleName = "level1_assets",
            assetNames = new[] { "Assets/Scenes/Level1.unity" }
        };

        BuildAssetBundleOptions options = BuildAssetBundleOptions.ChunkBasedCompression; // LZ4
        BuildTarget target = EditorUserBuildSettings.activeBuildTarget;

        BuildPipeline.BuildAssetBundles(
            outputPath,
            builds,                     // ← 这里传入数组
            options,
            target
        );

        AssetDatabase.Refresh();
        Debug.Log("指定 AssetBundles 打包完成！");
    }
}
```



**旧方法（简单，但不推荐大规模使用）：**不传 AssetBundleBuild 数组

```csharp
[MenuItem("Assets/Build All AssetBundles (Old Way)")]
static void BuildAllOldWay()
{
    string outputPath = "Assets/AssetBundles";  // 相对 Assets 路径也可以

    BuildPipeline.BuildAssetBundles(
        outputPath,
        BuildAssetBundleOptions.None,  // 或 ChunkBasedCompression
        BuildTarget.StandaloneWindows64   // 硬编码平台（不灵活）
    );
}
```



**快速上手步骤：**

1. **准备资源**：
   - 选中几个 Prefab/材质/贴图，在 Inspector 最下方 **AssetBundle** 下拉框选一个名字（如 characters、ui）。
   - 如果用上面第二种方式（AssetBundleBuild 数组），可以不手动标记，直接在代码里指定路径。
2. **创建脚本**：
   - 新建脚本 BuildAssetBundlesEditor.cs，放进 Assets/Editor/ 文件夹。
   - 复制上面任一版本代码。
3. **执行打包**：
   - 菜单：**Assets → Build AssetBundles (All)** 或 **Build Specific AssetBundles**。
   - 打包后在 Assets/AssetBundles/ 文件夹看到：
     - characters.unity3d
     - characters.unity3d.manifest
     - ui.unity3d
     - ……
     - 还有一个总的 **AssetBundles**（或你指定的文件夹名）.manifest 文件（包含所有 bundle 依赖信息）
4. **验证**：
   - 用 Profiler 查看打包是否成功。
   - 后续加载测试：用 AssetBundle.LoadFromFile 或 UnityWebRequestAssetBundle 加载这些 .unity3d 文件。



**常见参数说明：**

| 参数                    | 常用值                                    | 说明                              |
| ----------------------- | ----------------------------------------- | --------------------------------- |
| BuildAssetBundleOptions | ChunkBasedCompression                     | LZ4 分块压缩（推荐）              |
|                         | StrictMode                                | 严格模式，报错更清晰              |
|                         | ForceRebuildAssetBundle                   | 强制全量重建（调试用）            |
| BuildTarget             | EditorUserBuildSettings.activeBuildTarget | 跟随当前 Build Settings（最方便） |
|                         | BuildTarget.Android / iOS 等              | 手动指定平台                      |



**提醒：**

- 每次切换平台（Build Settings → Platform 改成 Android/iOS），都要重新 Build 一套（因为Bundle的平台专属特征）。
- 现在官方强烈建议转向 **Addressables**，但理解底层 BuildPipeline.BuildAssetBundles 对排查问题、自定义打包非常有帮助。



#### 如何读取并使用Bundle内资源？

以下为AB包写法（传统方法，还没有使用Addressables）。

先确保存在 core/shape 包体，且包体内存在 Shape1 预制件。

确保打包生成的包体复制到了 `StreamingAssets/AssetBundles/...` 。（注：无需复制 `.manifest` 。）

按如下方式编写脚本后挂载到场景运行。

```csharp
using System.IO;
using UnityEngine;

public class BundleLoader : MonoBehaviour
{
    void Start()
    {
        string bundlePath = Path.Combine(Application.streamingAssetsPath, "AssetBundles/core/shape");
        AssetBundle bundle = AssetBundle.LoadFromFile(bundlePath);
        GameObject loadedPrefab = bundle.LoadAsset<GameObject>("Shape1");
        Instantiate(loadedPrefab);
        bundle.Unload(false);
        
        // 注：可以通过下方代码获取bundle内所有资源名称，加载时可根据输出的资源名查找。
        // string[] names = bundle.GetAllAssetNames();
		// foreach (var n in names) Debug.Log("Bundle 内资源名: " + n);
    }
}
```

注意加载完成资源后运行 `bundle.Unload(false)` 避免包体残留在内存中造成泄漏。



### 注意事项



#### 关于包体命名

AssetBundle输出后通常没有后缀（如：`characters`）。

使用 `characters.unity3d` / `characters.bundle` 是社区约定，而非官方强制。