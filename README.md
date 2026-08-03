# 翻牌消消乐（Flop）

纯前端单机记忆配对游戏：翻开卡牌、配对消除，限时内清空棋盘即获胜。

## 特性

- **纯静态**：单文件 `index.html`（内联 CSS/JS），零依赖、无构建步骤。
- **无需联网**：游戏逻辑全部在浏览器本地运行。
- **数据本地**：最高分保存在本机 `localStorage`。
- **移动端适配**：针对触屏与微信 webview 优化。
- **分享卡片**：游戏内可生成 1200×1600 分享图（二维码需联网）。

## 本地运行

```sh
cd flop
python3 -m http.server 8000
# 浏览器打开 http://localhost:8000
```

> 分享卡片的二维码依赖 `api.qrserver.com`，必须经 `http(s)` 来源加载，请用本地服务器方式打开，不要直接 `file://` 打开。

## 文件结构

```
flop/
└── index.html   # 单文件，含全部 HTML/CSS/JS
```

## 部署

已部署至 Cloudflare Pages：`flop-39t.pages.dev`
