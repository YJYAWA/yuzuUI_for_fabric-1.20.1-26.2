# 修改声明

本仓库是对以下两个上游仓库的**非官方移植 / 衍生作品**：

| 上游 | 用途 |
|---|---|
| [ming-sc/YuZuUI-Forge](https://github.com/ming-sc/YuZuUI-Forge) | 原作（Forge 版）。界面布局、动画参数、素材全部来自这里 |
| [ming-sc/YuZuUI-Fabric](https://github.com/ming-sc/YuZuUI-Fabric) | 作者自己的 Fabric 1.20.1 版。本仓库 1.20.1 目标即基于它 |

原作者为 **IMG (ming-sc)**。上游声明为 `All Rights Reserved`，**本仓库未获授权**，详见
[README.md](README.md) 的授权声明与 [NOTICE.md](NOTICE.md)。

下面逐版本列出改动。所有 API 结论均通过 `javap` 反汇编对应版本的真实 Minecraft jar 取得，
不依赖网络上的二手资料。

---

## 通用事实

### 源码层面

- 包名、类名、`MOD_ID`（`yuzu`）、声音注册逻辑、以及 `src/main/resources/assets/yuzu/` 下的
  **全部素材（9 个 OGG + 20 个 PNG）** 均未改动，直接沿用上游。
- 界面元素的坐标、尺寸、动画参数（`delay` / `duration` / 各种 `AnimationFunction` 的系数
  `0.06 / 0.05 / 0.1`）逐字沿用上游，没有做视觉上的改动。

### 构建配置层面（四个版本共有）

- 每个版本的 `gradle.properties` 中 `org.gradle.java.home` 被注释掉：原值是本机的绝对路径，
  直接提交到仓库会让其他人在自己的机器上构建失败。改为注释 + 说明所需 JDK 版本。
- 补齐了 `gradlew` / `gradlew.bat`（上游 `.gitignore` 第 42–43 行把这两个文件排除了，
  所以上游仓库里没有它们）。`gradle/wrapper/gradle-wrapper.properties` 里
  Gradle 版本号沿用上游，下载地址使用官方 `services.gradle.org`。
- `build()` 相关的其余配置（`archives_base_name`、`processResources`、`publishing`）未改。

### 许可文件层面（四个版本共有）

- 四个子工程原本都带着一份 120 行的 `LICENSE`（CC0 1.0 全文）。它是 Fabric 官方示例模组模板
  `fabric-example-mod` 自带的默认文件，**并非原作者的授权声明** —— 上游 `gradle.properties`
  里根本没有填 `mod_license`。把它留在仓库里会让 GitHub 在仓库页顶部挂上 "CC0-1.0" 的许可标识，
  与「上游 All Rights Reserved、本仓库未获授权」的事实直接矛盾。
- 因此**替换**为一份说明「All Rights Reserved / NO LICENSE IS GRANTED」的文件，
  保留文件名 `LICENSE` 以便 `build.gradle` 中 `jar { from("LICENSE") ... }` 继续有效。

---

## 1.20.1 —— 源码零改动

`yuzu-fabric-1.20.1/src/` 与上游 `ming-sc/YuZuUI-Fabric` 的 `src/` **逐字节相同**
（`diff -rq` 无任何输出）。

改动仅在构建配置：

| 文件 | 改动 |
|---|---|
| `gradle.properties` | `org.gradle.jvmargs` 由 `-Xmx1G` 改为 `-Xmx2G`；`org.gradle.java.home` 注释掉 |

架构沿用上游：`@Mixin(TitleScreen.class)` 取消原版 `render`，配 `loom:injected_interfaces`
给 `DrawContext` / `MinecraftClient` 注入方法。

---

## 1.21.1 —— 在上游代码上前向修复

7 个文件有改动，均为 API 断裂导致的必要修改，无功能性改动。

### `src/client/java/com/img/mixin/client/DrawContextMixin.java`（改动最大）

1.20.5 移除了 `Tessellator.getInstance().getBuffer()` 与 `BufferBuilder.begin(...)` / `.next()`。
缓冲区绘制改写为 `Tessellator.getInstance().begin(...)`：

```java
// 1.20.1
BufferBuilder bufferBuilder = Tessellator.getInstance().getBuffer();
bufferBuilder.begin(VertexFormat.DrawMode.QUADS, VertexFormats.POSITION_TEXTURE);
bufferBuilder.vertex(matrix4f, x1, y1, z).texture(u1, v1).next();
...
BufferRenderer.drawWithGlobalProgram(bufferBuilder.end());

// 1.21.1
BufferBuilder bufferBuilder = Tessellator.getInstance().begin(VertexFormat.DrawMode.QUADS, VertexFormats.POSITION_TEXTURE);
bufferBuilder.vertex(matrix4f, x1, y1, z).texture(u1, v1);
...
BufferRenderer.drawWithGlobalProgram(bufferBuilder.end());
```

同时删除了已无法解析的 `@Shadow` 字段/方法：`vertexConsumers`、`tryDraw()`。

### `src/client/java/com/img/DrawContextMixinInterface.java`

删除 `fillGradientVertical(...)` 方法（原版 `DrawContext.fillGradient(x1,y1,x2,y2,z,c1,c2)` 语义相同，
不再需要自定义实现），并移除随之无用的 `VertexConsumer` import。

### `src/client/java/com/img/mixin/client/TitleClientMixin.java`

- 原先调用自定义的 `fillGradientVertical(...)`，改为直接调用原版 `context.fillGradient(...)`。
- 删除 `this.client.setConnectedToRealms(false);` —— 1.21.1 移除了该方法，
  原版 `TitleScreen.init()` 也不再调用它。

### `src/client/java/com/img/mixin/client/MinecraftClientMixin.java`
### `src/client/java/com/img/mixin/client/MusicTypeMixin.java`

`net.minecraft.client.sound.MusicType` → `net.minecraft.sound.MusicType`（该类在 1.21.x 搬了包）。

### `src/client/java/com/img/gui/TitleButtonWidget.java`

`mouseScrolled` 签名跟随原版接口变化：

```java
// 1.20.1
public boolean mouseScrolled(double mouseX, double mouseY, double amount)
// 1.21.1
public boolean mouseScrolled(double mouseX, double mouseY, double horizontalAmount, double verticalAmount)
```

### 构建配置

| 文件 | 改动 |
|---|---|
| `build.gradle` | Loom `1.8-SNAPSHOT` → `1.9.2`；`release` 与 `sourceCompatibility` / `targetCompatibility` 由 17 → 21；删除上游遗留的空 `repositories {}` 块 |
| `src/main/resources/fabric.mod.json` | `depends.minecraft` → `~1.21.1`，`depends.java` → `>=21` |
| `gradle.properties` | 见上文通用改动 |

---

## 1.21.8 —— 架构重写

1.21.6 起 GUI 改为 **RenderPipeline + GuiRenderState 两阶段渲染**，1.21.8 又把 `DrawContext`
整体改写。上游那套「mixin 原版 `TitleScreen` + 取消 `render` + 给 `DrawContext` 注入方法」的做法
在这里已经无法修补：

- `initWidgetsNormal(int,int)` 被改名为 `addNormalWidgets(int,int)`，`@Inject` 目标失效；
- `DrawContext.matrices` / `vertexConsumers` / `tryDraw` 全部消失，`@Shadow` 无法解析；
- 1.21.8 的 `DrawContext` 里 `drawTexture` 已是基于 `RenderPipeline` 的重载，
  上游注入的浮点坐标版本与之冲突。

因此**改为自绘架构**，与上游实现思路不同。

### 删除的文件

```
src/client/java/com/img/DrawContextMixinInterface.java
src/client/java/com/img/MinecraftClientMixinInterface.java
src/client/java/com/img/gui/TitleButtonWidget.java
src/client/java/com/img/mixin/client/DrawContextMixin.java
src/client/java/com/img/mixin/client/TitleClientMixin.java
src/client/java/com/img/mixin/client/MusicTypeMixin.java
src/client/java/com/img/mixin/client/SplashOverlayMixinAccessor.java
src/main/java/com/img/mixin/ExampleMixin.java      ← Fabric 模板遗留的空 mixin
```

同时从 `fabric.mod.json` 移除 `custom.loom:injected_interfaces` 块。

### 新增 / 重写的文件

| 文件 | 说明 |
|---|---|
| `gui/YuZuTitleScreen.java` | 新增。`extends Screen` 的**完全自绘**标题界面，取代上游对 `TitleScreen` 的 mixin |
| `gui/TitleButton.java` | 新增。取代 `TitleButtonWidget`，一个自绘 + 自命中测试的普通按钮对象，不再实现原版控件接口 |
| `gui/Layer.java` | 重写渲染部分以适配 `RenderPipeline` |
| `mixin/client/MinecraftClientMixin.java` | 重写为在 `MinecraftClient.setScreen` 处替换原版标题界面 |
| `yuzu.client.mixins.json` | 只保留 `MinecraftClientMixin` |

### 关键实现要点

**1. 透明度机制更换。** `RenderSystem.setShaderColor(...)` 在 1.21.6+ 已不再影响 GUI。
透明度改为通过 `DrawContext.drawTexture(..., int color)` 末尾的 **ARGB** 参数传入，
新增 `Layer.toColor(float alpha)` 做 `0~1` → ARGB 白色乘算：

```java
public static int toColor(float alpha) {
    int a = (int) (MathHelper.clamp(alpha, 0.0F, 1.0F) * 255.0F);
    return (a << 24) | 0xFFFFFF;
}
```

**2. 拦截点。** 不再 mixin `TitleScreen`，而是 mixin `MinecraftClient.setScreen`：

```java
@Inject(method = "setScreen", at = @At("HEAD"), cancellable = true)
private void yuzu$replaceTitleScreen(Screen screen, CallbackInfo ci) { ... }
```

这里**必须用 `@Inject(HEAD) + cancel`，不能用 `@ModifyVariable(argsOnly = true)`**。
反汇编 `MinecraftClient.setScreen` 可见，方法体内对 `screen == null && world == null` 的情况
会把**局部变量**赋值为 `new TitleScreen()`：

```
73: aload_1
74: ifnonnull  95
77: getfield   world
81: ifnonnull  95
84: new        TitleScreen
91: astore_1            ← 赋值给局部变量，发生在参数被读取之后
```

`argsOnly` 修改的是入参，看不到这次内部赋值，会漏掉游戏启动时的初始标题界面。

另外，原版在 `screen == null && disconnecting` 时会抛 `IllegalStateException`，
注入代码保留了这个语义不被拦截。递归调用是安全的：内层传入的是 `YuZuTitleScreen`，
既不为 `null` 也不是 `TitleScreen`，会立即 return。

**3. 标题音乐。** 不再 mixin `MusicType`，改为覆写 `Screen.getMusic()` 返回自建的
`MusicSound(RegistryEntry.of(InitSounds.YUZU_TITLE_MUSIC), 50, 50, true)`。
已确认 `Screen.getMusic()` 默认返回 `null`，且 `MinecraftClient.getMusicInstance()`
会优先取 `currentScreen.getMusic()`。

**4. 按钮行为。** 沿用上游语义，仅按 1.21.8 的 API 改写构造方式：

| 按钮 | 1.21.8 实现 |
|---|---|
| 新游戏 | `CreateWorldScreen.show(this.client, this)` |
| 选择世界 | `new SelectWorldScreen(this)` |
| 多人游戏 | `skipMultiplayerWarning ? new MultiplayerScreen(this) : new MultiplayerWarningScreen(this)` |
| Realms | `new RealmsMainScreen(this)` |
| 选项 | `new OptionsScreen(this, this.client.options)` |
| 退出游戏 | `this.client.scheduleStop()` |

### 构建配置

| 文件 | 改动 |
|---|---|
| `build.gradle` | Loom → `1.11.8`；Java 17 → 21 |
| `gradle.properties` | Minecraft 1.21.8 / Yarn `1.21.8+build.1` / Loader 0.17.2 / Fabric API `0.136.1+1.21.8` |
| `fabric.mod.json` | `depends.minecraft` → `~1.21.8`，`depends.java` → `>=21`；移除 `loom:injected_interfaces` |

---

## 26.2 —— 换到官方命名 + no-remap 构建

架构与 1.21.8 相同（自绘 `Screen` + 拦截 `setScreen`），差异在于 26.2 的 API 命名与构建方式。

### 命名体系变化

26.2 的 Minecraft jar **本身不再混淆**，直接以 `net.minecraft.*` 命名发布
（已用 `unzip -l` 在官方 client jar 中确认存在 `net/minecraft/client/Minecraft.class`、
`net/minecraft/resources/Identifier.class`），并且 Mojang **不再为 26.2 提供 `client_mappings`**
（版本 JSON 的 `downloads` 里只有 `client` 和 `server`）。因此：

- 不存在可用的 Yarn / 官方映射，`loom.officialMojangMappings()` 会直接报
  `Failed to find official mojang mappings for 26.2`。
- 构建改用 Loom 的 no-remap 入口插件：`id 'net.fabricmc.fabric-loom'`（它内部以
  `disableObfuscation = true` 应用 `fabric-loom`），**没有 `mappings` 依赖**。
- 该模式下不生成 RemapConfiguration，因此依赖必须写成普通 `implementation` 而非
  `modImplementation`。

```groovy
plugins {
    id 'net.fabricmc.fabric-loom' version '1.17.21'
}
dependencies {
    minecraft "com.mojang:minecraft:${project.minecraft_version}"
    implementation "net.fabricmc:fabric-loader:${project.loader_version}"
    implementation "net.fabricmc.fabric-api:fabric-api:${project.fabric_version}"
}
```

这样打出的 jar 其 MANIFEST 为 `Fabric-Mapping-Namespace: official`，且**不含 refmap**（无需重映射）。

### API 适配对照

| 上游（1.20.1 / Yarn） | 26.2（官方命名） |
|---|---|
| `net.minecraft.client.gui.DrawContext` | `GuiGraphicsExtractor` |
| `Screen.render(DrawContext,int,int,float)` | `Screen.extractRenderState(GuiGraphicsExtractor,int,int,float)` |
| `DrawContext.drawTexture(...)` | `GuiGraphicsExtractor.blit(...)`，末尾同样接受 ARGB `int color` |
| `MinecraftClient.setScreen(...)` | `Minecraft.gui.setScreen(...)` / `Minecraft.setScreenAndShow(...)` |
| `MouseButtonEvent` 不存在，用 `mouseClicked(double,double,int)` | **有** `MouseButtonEvent`，签名为 `mouseClicked(MouseButtonEvent, boolean)` |
| `Identifier.of(ns, path)` | `Identifier.fromNamespaceAndPath(ns, path)` |
| `ResourceLocation` | `Identifier` |
| `SoundEvent.of(id)` | `SoundEvent.createVariableRangeEvent(id)`（`SoundEvent` 在 26.x 变成 record） |
| `MusicSound` | `Music` |
| `MusicType.MENU` | `Musics.MENU` |
| `MinecraftClient.scheduleStop()` | `Minecraft.stop()` |
| `SplashOverlay` | `LoadingOverlay` |

### 与 1.21.8 的文件差异

| 文件 | 变化 |
|---|---|
| `mixin/client/GuiMixin.java` | **新增**，取代 1.21.8 的 `MinecraftClientMixin`。26.2 的界面切换入口在 `net.minecraft.client.gui.Gui.setScreen(Screen)`，`@Shadow` 的是 `minecraft` 字段与 `clientLevelTeardownInProgress` 布尔 |
| `InitSounds.java` | 改用 `Registry.register(BuiltInRegistries.SOUND_EVENT, id, SoundEvent.createVariableRangeEvent(id))` |
| `gui/Layer.java` | `drawTexture` → `blit`，`MathHelper` → `Mth` |
| `gui/TitleButton.java` | 声音播放由 `PositionedSoundInstance` 改为 `SimpleSoundInstance.forUI(...)` |
| `mixin/client/TitleClientMixin.java` 等 | 不适用（1.21.8 阶段已删除） |
| `yuzu.client.mixins.json` | `compatibilityLevel` → `JAVA_25`，client 列表为 `GuiMixin` |

`GuiMixin` 采用与 1.21.8 完全相同的 `@Inject(HEAD) + cancel` 策略，原因同前：
反汇编 `Gui.setScreen` 同样可见方法体内存在 `new TitleScreen()` 的局部变量赋值，
`@ModifyVariable(argsOnly = true)` 一样拦不住。

### 构建配置

| 文件 | 改动 |
|---|---|
| `build.gradle` | 见上（no-remap 插件、无 mappings、`implementation`）；Java 21 → 25 |
| `gradle.properties` | Minecraft 26.2 / Loader 0.19.3 / Fabric API `0.157.0+26.2`，无 Yarn |
| `fabric.mod.json` | `depends.minecraft` → `~26.2`，`depends.java` → `>=25` |

---

## 未做的事

- **没有跑过 `runClient` 冒烟测试。** 四个版本都只验证到「构建通过 + 产物结构完整」，
  标题界面在真实游戏中的显示 / 按钮响应 / 音乐播放均未实测。
- 没有改动任何界面素材、坐标、动画参数。
- 没有为 1.20.1 之外的其他 MC 版本做 ROM 兼容（每个 jar 只对应表中的一个版本）。
