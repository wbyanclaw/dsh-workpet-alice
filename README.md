# dsh-workpet-alice

给 DeepSeek Harness 的 **Work Pet**（`dsh-workpet`）加一个 **alice** 形象：戴圆框眼镜、深棕短卷发、棕色西装的半身少女，素材来自 Mori 的 `alice_half` 雪碧包。

## 依赖

必须先装 `dsh-workpet` 本体（它提供 `ctx.workpet`，并自带默认小猫）：

```text
install_bundle dsh-workpet
```

## 安装本插件（本地路径）

在 DeepSeek Harness 会话里让 agent 执行：

```text
install_bundle d:/POC/chatbot/work/dsh-workpet-alice
```

CLI 管理的 profile 也可以：

```sh
dsh plugin --profile <profile> add link:d:/POC/chatbot/work/dsh-workpet-alice
```

装完刷新 Harness 窗口，宠物出现在右下角。`link:` 安装指向本目录，之后改 `client.js` 不需要重装。

## 验证

**Plugins → Work Pet** 页面里，alice 的卡片会把 13 个动作都演一遍（不需要等真实会话睡着或失败）。

命令行语法检查：

```sh
npm run check
```

## 动作映射

| DSH activity | 用的雪碧状态 | 动效 |
|---|---|---|
| absent | sleeping | 慢呼吸 |
| idle | idle | 呼吸 |
| sleeping | sleeping | 慢呼吸 |
| thinking | thinking | 轻微摇晃 |
| writing | recording | 上下小幅弹动 |
| tool | walking | 左右摆动 |
| retrying | error | 抖动 |
| awaiting | recording | 上下小幅弹动 |
| delegating | idle | 环顾 |
| done | done | 跳跃 |
| error | error | 抖动 |
| petted | done | 轻弹 |
| dragged | dragging | 摆荡 |

## 结构

- `client.js` — 全部内容：8 张 WebP（base64 内嵌）、13 个动作映射、CSS 关键帧、`ctx.workpet.register` 注册块（id 为 `alice`）
- `index.js` — 空的 Host half
- `cordis.patch.yml` — 安装时插入的 Loader 行
- `locale/*.json`、`icon.svg` — 插件管理器里显示的名字/描述/图标

## 换素材

图片是从 `d:\POC\chatbot\work\mori\half\sprites\<state>.png` 每张图集的第 1 格裁出来的，缩放成 160px 高、WebP q88。换形象时重新跑一遍生成脚本即可（脚本会重写 `client.js` 的 ART 段）。

## 发布到 npm（可选）

本地能用之后如需公开：确认 `package.json` 的 `name` 未被占用 → `npm login` → `npm publish` → 之后别人 `install_bundle dsh-workpet-alice` 就能装。正式发布建议照 `dsh-workpet-parrot` 的 Trusted Publisher 流程走。
