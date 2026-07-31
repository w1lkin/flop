# 翻牌消消乐（Flop）

纯前端单机小游戏：翻开卡牌、配对消除，限时内清空棋盘即获胜。

## 单机版特性

- **纯静态**：仅 `index.html`（内联 CSS/JS），零依赖、无构建步骤。
- **无需联网**：所有逻辑在浏览器本地运行，不依赖后端或账号。
- **数据本地**：最高分 / 进度仅保存在本机 `localStorage`。
- **即开即玩**：双击 `index.html` 即可运行（分享卡二维码需联网，但游戏本体离线可用）。

## 本地运行

```sh
cd flop
python3 -m http.server 8000
# 浏览器打开 http://localhost:8000
```

或直接用浏览器打开 `index.html`。

## 文件结构

```
flop/
└── index.html   # 单文件，含全部 HTML/CSS/JS
```

## 部署

可一键部署到 Cloudflare Pages（根目录即 `index.html`）。

## 版本

当前分支：`release/1.0.0`
