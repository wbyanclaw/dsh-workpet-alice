# Artwork source / 素材来源

这些文件是 `dsh-workpet-alice` 的**原始素材**，不是插件运行时文件。

These are the **source assets** for `dsh-workpet-alice`. The plugin itself embeds downscaled
WebP cut-outs in `client.js`; these sheets are kept here so the artwork can be reproduced,
re-cut or re-used.

## 内容 / Contents

```
sprites/
  idle.png  sleeping.png  recording.png  thinking.png
  done.png  error.png     walking.png    dragging.png
```

8 张雪碧图 / 8 sprite sheets：

| 项目 | 规格 |
|---|---|
| 格式 | PNG-32（RGBA，透明背景） |
| 尺寸 | 1024 × 1024 |
| 网格 | 4 × 4 = 16 帧 |
| 单帧 | 256 × 256 |
| 排列 | 行优先，从左到右、从上到下 |
| 构图 | 半身（到上胸），角色占满单帧 |

对应 [character-pack](https://github.com/yazelin/mori-desktop) 规范，可直接作为 Mori
角色包的 `sprites/` 内容使用。

## 来源 / Provenance

- **角色**：`alice`，作者原创角色。原始形象来自作者自有素材
  `live2d-models/alice/artwork/alice.mp4`（Open-LLM-VTuber 模型目录中的角色演示视频）。
- **生成流程**：以该视频首帧为参考图，经 codex-image-service（OpenAI Codex `$imagegen`）
  重绘为 4 × 4 雪碧；随后绿幕抠图、按 alpha 包围盒跨状态统一对齐、缩放输出 1024 × 1024。
- **AI 声明**：画面由图像生成模型产出并经人工筛选；同角色在不同状态下存在轻微笔触差异，
  属于该流程的已知特性。
- **未使用第三方素材**：仓库中不包含任何第三方绘画、贴图或字体资源。

- **Character**: `alice`, an original character by the repository owner. The reference is the
  owner's own asset `live2d-models/alice/artwork/alice.mp4` (character showcase video in the
  Open-LLM-VTuber model directory).
- **Pipeline**: the first video frame was used as an image reference; codex-image-service
  (OpenAI Codex `$imagegen`) redrew it into 4 × 4 sprite sheets; sheets were chroma-keyed and
  cross-state aligned by alpha bounding box, then exported at 1024 × 1024.
- **AI disclosure**: frames are model-generated and human-curated; slight stylistic drift
  between states is inherent to this pipeline.
- **No third-party assets**: no third-party drawings, textures or fonts are included.

## 许可 / License

MIT，© 2026 Yu Dong。与仓库根目录的 [LICENSE](../LICENSE) 相同——代码与素材采用同一许可，
可自由使用、修改、再分发，保留版权与许可声明即可。

MIT, © 2026 Yu Dong. Same terms as the repository [LICENSE](../LICENSE): code and artwork are
licensed together — use, modify and redistribute freely, keep the copyright and permission
notice.

## 从素材重建插件 / Rebuilding the plugin frames

插件里每状态只用第 1 帧（左上角那格），流程：

1. 切成 4 × 4 格；
2. 按 alpha 包围盒裁边；
3. 缩到 160 px 高；
4. 编码 WebP（q88）；
5. base64 后写进 `client.js` 的 `ART`，并按裁边后的宽 ÷ 高更新 `RATIO`。

单张控制在 120 KB 以内——所有宠物共处一页，`client.js` 是一次性加载的。
