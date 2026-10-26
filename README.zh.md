# dsh-workpet-alice

给 [DeepSeek Harness](https://github.com/lastorder/dsh-workpet) 的 **Work Pet**（`dsh-workpet`）加一个 **alice** 形象：戴圆框眼镜、深棕短卷发、棕色西装的半身少女。闲置时呼吸，模型输出时小幅弹动，工具调用时左右摆动，出错时抖动，一轮结束时跳跃，被你拖拽时摆荡。

素材从 Mori 的 2D 雪碧包（`alice_half`）转制：每个状态取一帧，内嵌为 WebP，画进宠物固定的 100 × 80 SVG 盒子。

[English](README.md)

## 依赖

- DeepSeek Harness（桌面版，或 CLI 管理的 profile）
- 同一个 profile 里已安装 `dsh-workpet` 本体 —— 它提供 `ctx.workpet`，并自带默认小猫

## 安装

### 方式一：在 Harness 会话里装（桌面版只能用这个）

在任意 session 里让 agent 执行：

```text
install_bundle dsh-workpet
install_bundle <本仓库绝对路径>     # 例如 d:/POC/chatbot/work/dsh-workpet-alice
```

第一条只需执行一次。目标可以是绝对路径（如上），也可以是包名——前提是本包已发布到 npm。

### 方式二：CLI 管理的 profile

```sh
dsh plugin --profile <profile> add link:/absolute/path/to/dsh-workpet-alice
```

`link:` 安装指向本目录，之后改 `client.js` 只要刷新窗口即可生效，无需重装。

安装完成后 **刷新 Harness 窗口**，alice 会和其他宠物一起出现在输入框上方，不需要任何设置。

## 使用

- 她会自动跟随会话状态：idle → 呼吸，thinking → 轻微摇晃，writing → 弹动，工具调用 → 摆动，失败 → 抖动，完成 → 跳跃。
- **点击**她是摸头（会精神一下），**拖拽**可以挪位置，位置按 skin id `alice` 记住。
- 打开 **Plugins → Work Pet** 可以看到形象卡片。alice 卡片下方的动作按钮会把 13 个 activity 逐个演一遍，不用等真实会话睡着或失败就能检查每个姿势。
- 在插件管理器里关掉本 bundle 即可移除她，默认小猫不受影响。

## 动作映射

| `scene.activity` | 触发时机 | 用的雪碧状态 | 动效 |
|---|---|---|---|
| `absent` | 没有打开的会话 | sleeping | 慢呼吸 |
| `idle` | 会话闲置 | idle | 呼吸 |
| `sleeping` | 闲置超过 70 秒 | sleeping | 慢呼吸 |
| `thinking` | 模型在跑、还没吐字 | thinking | 轻微摇晃 |
| `writing` | 模型正在流式输出 | recording | 小幅弹动 |
| `tool` | 有工具调用在跑 | walking | 左右摆动 |
| `retrying` | 等待重试失败请求 | error | 抖动 |
| `awaiting` | 等你审批或回答 | recording | 小幅弹动 |
| `delegating` | 子 agent 还在跑 | idle | 环顾 |
| `done` | 刚完成一轮 | done | 跳跃 |
| `error` | 刚失败 | error | 抖动 |
| `petted` | 你点了她 | done | 轻弹 |
| `dragged` | 你正在拖她 | dragging | 摆荡 |

状态气泡、工具徽标、helper 小同伴，以及 `z`/`!`/`?`/爱心/闪光这些特效都由 Work Pet 自己画 —— 本包不画这些。

## 工作原理

`client.js` 被 Harness **逐字节原样加载**（无构建步骤、无 TypeScript）。里面分三块：

1. **ARTWORK** —— `ART`（状态名 → `data:image/webp;base64,…`）、`RATIO`（每张裁切图的宽高比）、`ACT`（activity → `[状态, css 类]`）。
2. **ANIMATION** —— `CSS` 字符串，`al-` 前缀的类和 `@keyframes`（呼吸、弹动、摇晃、摆动、抖动、跳跃、环顾、轻弹、摆荡），末尾带 `prefers-reduced-motion` 关闭动效的规则。
3. **PLUG-IN** —— `ctx.effect` 接线：注入样式表、注册语言包（`workpetAlice`）、注册皮肤：

```js
ctx.workpet.register({
  id: 'alice',        // 稳定 id；她在屏幕上的位置按这个 id 保存
  order: 10,          // 0 是默认小猫
  title: () => t('title'),
  description: () => t('description'),
  Figure: AliceFigure // ({ scene, compact }) => <svg viewBox="0 0 100 80">
})
```

`AliceFigure` 返回 100 × 80 盒子的纯 SVG：一个柔和阴影椭圆 + 当前状态的 `<image>`，外面套一层带动效类的 `<g>`。`compact`（helper 小同伴）模式下同样渲染，只是不画场景元素 —— 当前素材没有场景，所以两种模式看起来一样。

文件说明：

| 文件 | 作用 |
|---|---|
| `client.js` | 浏览器端全部内容：素材、动效、注册 |
| `index.js` | 空的 Host 半边，Loader 行需要一个模块可加载 |
| `cordis.patch.yml` | 安装 bundle 时插入 `dsh-workpet-alice` 这一行 |
| `package.json` | bundle 清单；`dsh.client.inject: ["dsh-workpet"]` 让 Work Pet 先加载 |
| `locale/{en,zh}.json` | 插件管理器里显示的名称与描述 |
| `icon.svg` | 插件管理器图标 |

## 自定义

### 换素材

当前帧取自 `alice_half` 图集（`1024×1024`，4 × 4 网格，每格 256 px）：每个状态取第 1 格，按 alpha 裁边，缩到 160 px 高，编码为 WebP q88。

换成自己的图：

1. 把图集切成 4 × 4 格，每个状态挑一帧。
2. 按 alpha 包围盒裁边，缩到 160 px 高，编码 WebP，转 base64。
3. 替换 `ART` 里对应项，并按“宽 ÷ 高”更新 `RATIO`。
4. 刷新 Harness 窗口（若还是旧图就重启应用）。

单张图控制在 120 KB 以内：所有宠物同处一个页面，整份 `client.js` 是一次性加载的。

### 改动效

在 `CSS` 里加/改类，并在 `ACT` 里引用。SVG 里有两条规矩：

- CSS 旋转以 `transform-origin` 为轴，单位是 **viewBox 单位**（`style="transform-origin:50px 78px"`）；不写则绕 `(0,0)` 转。
- 不要给带 `transform="…"` 属性的元素再加动效类 —— 用 `<g>` 嵌套，否则 CSS transform 会覆盖该属性。

### 改名字做新宠物

分叉成别的角色时，这些都要一起改：`package.json` 的 `name`、`cordis.patch.yml` 的 `id`/`name`、`window.__ModuleLoader__.load` 的 `id`（必须等于包名）、`ctx.workpet.register` 的 skin `id`、`al-` CSS 前缀（避免同页两个宠物样式冲突）。

## 常见问题

| 现象 | 处理 |
|---|---|
| 什么都不显示 | `dsh-workpet` 必须装在**同一个** profile；在插件管理器确认两个 bundle 都已启用 |
| 刷新后还是旧图 | 重启 DeepSeek Harness，让它重新加载 `client.js` |
| 出现了但不动 | 系统层面开了 `prefers-reduced-motion`，动效会按设计关闭 |
| 只想留 alice | 默认小猫属于 `dsh-workpet` 本体，形象是叠加的；不想看小猫就关掉 `dsh-workpet` 的默认形象或接受两个都在 |
| 重装后位置复原 | 位置按 skin `id` 保存，改 `id` 会重置 |

## 卸载

在插件管理器里关掉，或从 profile 移除：

```text
install_bundle --remove dsh-workpet-alice
```

（CLI 管理的 profile：`dsh plugin --profile <profile> remove dsh-workpet-alice`。）

## 开发

```sh
npm run check     # 对 client.js 与 index.js 做 node --check，无需依赖
```

没有构建步骤：改 `client.js` → 刷新窗口 → 用 **Plugins → Work Pet** 里的动作按钮预览 13 个 activity。

## 发布到 npm

发布后安装可以直接用包名：

```sh
npm login
npm publish
# 然后：install_bundle dsh-workpet-alice
```

仓库沿用了 `dsh-workpet-parrot` 的约定（bundle patch + `dsh.client` 清单），后续可以在 npmjs.com 配 Trusted Publisher 接管发布。

## 致谢与许可

- 插件结构来自 [dsh-workpet-parrot](https://github.com/lastorder/dsh-workpet-parrot)（lastorder，MIT）。
- 素材：为本项目从 `alice_half` Mori 雪碧包转制，见 [LICENSE](LICENSE)。
- 代码：MIT。
