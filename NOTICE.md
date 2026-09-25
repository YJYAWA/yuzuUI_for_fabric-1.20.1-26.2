# 版权与归属声明

## 上游仓库

本仓库（`yuzuUI_for_fabric-1.20.1-26.2`）是以下两个仓库的**非官方前向移植**：

| 仓库 | 作者 | 本仓库对它的使用 |
|---|---|---|
| [ming-sc/YuZuUI-Forge](https://github.com/ming-sc/YuZuUI-Forge) | [ming-sc](https://github.com/ming-sc) | 原作。界面设计、素材、动画参数、实现思路的来源 |
| [ming-sc/YuZuUI-Fabric](https://github.com/ming-sc/YuZuUI-Fabric) | [ming-sc](https://github.com/ming-sc) | 作者自己的 Fabric 1.20.1 版。本仓库 `yuzu-fabric-1.20.1/` 的 `src/` 与它逐字节相同 |

**上游版权归原作者所有。** `ming-sc/YuZuUI-Forge` 的 `gradle.properties` 中声明：

```properties
mod_license=All Rights Reserved
```

`ming-sc/YuZuUI-Fabric` 中虽有一份 `LICENSE`（CC0 1.0），但那是 Fabric 官方示例模组模板
（`fabric-example-mod`）自带的默认文件，该仓库的 `gradle.properties` 中并未填写 `mod_license`，
因此**不构成作者对本项目的一次明确授权**。

**本仓库未获得原作者许可。** 在取得许可之前请勿分发本仓库或其构建产物。
若原作者提出要求，本仓库应立即删除。详见 [README.md](README.md) 的授权声明一节。

---

## 素材归属

`src/main/resources/assets/yuzu/` 下的全部素材**原样取自上游，未经任何修改**
（未裁剪、未调色、未重编码）。已核对数量：

### 音频（9 个 OGG，`assets/yuzu/sounds/`）

```
yuzu_title_music.ogg                     标题背景音乐
yuzu_title_senren.ogg                    立绘登场音
yuzu_title_button_on.ogg                 按钮悬停音
yuzu_title_button_click.ogg              按钮点击音
yuzu_title_button_new_game.ogg           新游戏
yuzu_title_button_select_world.ogg       选择世界
yuzu_title_button_realms.ogg             Realms
yuzu_title_button_options.ogg            选项
yuzu_title_button_quit_game.ogg          退出游戏
```

播放与注册方式见 `InitSounds.java`。这些音频的著作权归原作者 / 原始权利人所有。

### 图像（20 个 PNG）

`assets/yuzu/icon.png` —— 模组图标（1 个）

`assets/yuzu/textures/gui/` —— 标题界面素材（19 个）

```
background.png                           背景
title_yoshino.png / title_murasame.png   人物立绘
title_mako.png    / title_lena.png
title_head.png                           左下方人物
title_logo.png                           标题 LOGO
title_new_game_button_normal.png         按钮（normal / on 各一张）
title_new_game_button_on.png
title_select_world_button_normal.png
title_select_world_button_on.png
title_continue_button_normal.png
title_continue_button_on.png
title_realms_button_normal.png
title_realms_button_on.png
title_options_button_normal.png
title_options_button_on.png
title_quit_game_button_normal.png
title_quit_game_button_on.png
```

### 角色与作品

界面中的四张立绘取自《千恋＊万花》的女主角（按文件名对应，实际画面以素材本身为准）：

| 素材 | 角色 |
|---|---|
| `title_yoshino.png` | 朝武 芳乃（ともり よしの） |
| `title_murasame.png` | 叢雨（むらさめ） |
| `title_mako.png` | 常陸 茉子（ひたち まこ） |
| `title_lena.png` | レナ（Lena Liechtenstein） |

《千恋＊万花》（千恋＊万花 / Senren＊Banka）由 **ゆずソフト（Yuzusoft）** 开发，
作品及角色的著作权归 Yuzusoft 及相关权利人所有。本仓库与 Yuzusoft 无任何关联，
亦未获得其授权。素材仅作为同人性质的界面替换素材使用。

---

## 第三方组件

| 组件 | 许可 |
|---|---|
| [Fabric Loader](https://github.com/FabricMC/fabric-loader) | Apache-2.0 |
| [Fabric API](https://github.com/FabricMC/fabric-api) | Apache-2.0 |
| [Fabric Loom](https://github.com/FabricMC/fabric-loom) | MIT |
| [SpongePowered Mixin](https://github.com/SpongePowered/Mixin) | MIT |
| [Gradle](https://gradle.org/) | Apache-2.0 |
| Minecraft | Mojang AB 的最终用户许可协议（EULA） |

Fabric 示例模组模板（`fabric-example-mod`）的 CC0 1.0 声明仅覆盖模板本身产生的
脚手架代码（`ExampleMixin`、`build.gradle` 骨架等），不覆盖本项目的界面实现与素材。
本仓库中 `src/main/java/com/img/mixin/ExampleMixin.java` 这一模板遗留文件已在
1.21.8、26.2 两个版本中删除（见 [MODIFICATIONS.md](MODIFICATIONS.md)）。

---

## 本仓库自身的改动

移植过程中新增 / 改写的代码（例如自绘的 `YuZuTitleScreen`、`TitleButton`、`Layer`
以及与各版本 API 对接的 mixin）由本仓库作者撰写，但**作为衍生作品，其分发同样受上游
`All Rights Reserved` 声明约束** —— 在上游许可未明确之前，本仓库整体不构成一个可自由
再分发的作品。

逐版本的详细改动清单见 [MODIFICATIONS.md](MODIFICATIONS.md)。
