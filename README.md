# Ps2Unreal Releases

Ps2Unreal `0.1.1` 是 Unreal Engine 5 编辑器插件，用于把 PS2UI Photoshop Package 导入为可编辑的 UMG Widget Blueprint。插件只在编辑器运行，生成资源使用 UE/UMG 标准类型，游戏运行时和正式打包不依赖插件。

## 前置：安装 PS2UI

本仓库不包含 Photoshop 插件。先从 [PS2UI Releases](https://github.com/Homer79980/PS2UI-Releases/releases/latest) 安装 `PS2UI-Photoshop-<版本>.ccx`，然后在 Photoshop 中导出包含 `layout.json` 和 `sprites/` 的 Package。

## 下载

[打开最新 Release](https://github.com/Homer79980/Ps2Unreal-Releases/releases/latest)

| 文件 | 用途 |
|---|---|
| `Ps2Unreal-0.1.1-UE5.7-Win64.zip` | UE 5.7 Win64 预编译包 |
| `Ps2Unreal-0.1.1-UE5.8-Win64.zip` | UE 5.8 Win64 预编译包 |
| `Ps2Unreal-0.1.1.zip` | 通用源码包，用于其它 UE5 小版本自行编译 |
| `Ps2Unreal-0.1.1-SHA256.txt` | 三个 ZIP 的 SHA-256 |

预编译包必须与 UE 小版本严格对应，不能把 UE 5.7 二进制直接用于 UE 5.8。源码编译需要 Visual Studio 2022 和目标引擎支持的 MSVC v143/Windows SDK。

## 安装

1. 关闭 Unreal Editor。
2. 下载与引擎版本匹配的 ZIP 并解压。
3. 把完整 `Ps2Unreal` 文件夹复制到 `<项目根目录>/Plugins/Ps2Unreal`。
4. 确认最终路径是 `<项目>/Plugins/Ps2Unreal/Ps2Unreal.uplugin`，不能多套一层同名目录。
5. 打开项目；源码包首次打开时允许 Unreal 编译插件。
6. 进入 `Edit -> Plugins`，搜索 `Ps2Unreal` 并启用，按提示重启编辑器。

正确结构：

```text
项目根目录/
  MyProject.uproject
  Plugins/
    Ps2Unreal/
      Ps2Unreal.uplugin
      Source/
      Binaries/        # 预编译包才有
```

## 导入操作

1. 执行 `Tools -> Ps2Unreal：导入 PS2UI 导出包...`。
2. 选择 Package 根目录，不要选择 `sprites` 子目录。
3. “字体与导入”会列出项目 Font、已记住 UMG 默认字体和待处理字体；可立即绑定，也可暂用默认字体继续。
4. 导入完成后，在 `/Game/Ps2Unreal/<module>/Widgets` 打开生成的 `WBP_*`。
5. 图片资产位于 `/Game/Ps2Unreal/<module>/Textures`。

Schema 1 生成一个 Widget Blueprint，Schema 2/3 为每个可见画板生成一个 Widget Blueprint。普通图片是 `Image`，九宫是 `Brush.DrawAs = Box`，文字是 `Text Block`，按钮保留透明 `Button` 交互层。

## 字体与排版

PS2UI Package 已包含字体身份和排版参数，不需要在导入前额外准备字体 JSON。

- 映射目标是 UE `Font` 资产（Composite Font Family），不是单独 Font Face。
- 字体名称精确且唯一时可自动匹配；同名候选超过一个时必须人工选择。
- 没有项目字体时仍生成完整 Widget，并保留文字矩形、字号、tracking、显式行距、对齐和缩放。
- 映射保存在 `<项目>/Config/Ps2Unreal/font-map.json`，建议随项目版本控制。
- 默认字体字形度量可能与 Photoshop 不同，因此完成映射前换行位置可能不同。

## 项目资源复用

- PS2UI 先归并 Package 内解码像素、尺寸和九宫契约完全相同的图片。
- Ps2Unreal 再扫描 `/Game` 已有 `Texture2D`；像素完全一致时自动复用，不依赖名称或目录。
- 不同尺寸、镜像或高相似候选必须人工确认，决定保存在 `Saved/Ps2Unreal/reuse-decisions.json`。
- 复用已有 Texture2D 时不修改其像素或导入设置。
- 内容指纹未变化时跳过 Widget 保存；插件不注册全局纹理、Cook、构建或自动刷新钩子。

## 九宫拉伸

九宫边界会归一化写入 `FSlateBrush.Margin`。在 UMG 中修改 Slot/Canvas 节点尺寸来拉伸，不要用 Render Scale 代替尺寸，否则固定边角也会缩放。

## 查看版本

- 在 `Edit -> Plugins` 搜索 `Ps2Unreal` 查看 Version。
- 或查看 `<项目>/Plugins/Ps2Unreal/Ps2Unreal.uplugin` 的 `VersionName`。
- Tools 菜单只保留一个中文导入入口。

## 升级、卸载与排错

- 升级：关闭编辑器，用匹配引擎版本的新 `Ps2Unreal` 完整覆盖旧目录，再启动项目。
- 卸载：关闭编辑器并删除 `<项目>/Plugins/Ps2Unreal`；生成的 `/Game/Ps2Unreal` 资源可按项目需要保留。
- Plugins 列表找不到：检查 `.uplugin` 是否直接位于 `Plugins/Ps2Unreal`，并确认没有 `Plugins/Ps2Unreal/Ps2Unreal/Ps2Unreal.uplugin` 的重复嵌套。
- “模块缺失或使用不同引擎版本构建”：换用匹配的小版本预编译包，或安装 C++ 工具链后使用源码包重新编译。
- 重导可能重建生成 Widget，业务蓝图逻辑应放在生成 Widget 的子类或外层页面。

## 校验下载

```powershell
Get-FileHash .\Ps2Unreal-0.1.1-UE5.7-Win64.zip -Algorithm SHA256
Get-FileHash .\Ps2Unreal-0.1.1-UE5.8-Win64.zip -Algorithm SHA256
Get-FileHash .\Ps2Unreal-0.1.1.zip -Algorithm SHA256
```

输出应与 `Ps2Unreal-0.1.1-SHA256.txt` 一致。
