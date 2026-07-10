# 翻牌消消乐 实现计划

> **对于自动化工作者：** 请使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 按任务逐步实现此计划。步骤使用复选框 (`- [ ]`) 语法进行跟踪。

**目标：** 构建一个日系清新风格的记忆配对翻牌小游戏，纯 HTML 单文件，零依赖，通关后可生成分享卡片。

**架构：** 单个 `index.html` 文件，内联 `<style>` 和 `<script>`。CSS 变量管理日系清新主题，CSS Grid 响应式棋盘，CSS 3D transform 实现翻牌动画，Canvas 离屏渲染生成 1200×1600 分享卡片（QR 码通过 api.qrserver.com 加载）。

**技术栈：** HTML5 + CSS3 + 原生 JavaScript（零依赖）

## 全局约束

- 纯 HTML 单文件，零外部依赖
- 日系清新主题：主色 `#81C784`，背景渐变 `#e8f5e9` → `#fafafa`
- 三档难度：4×4（简单）、4×6（普通）、6×6（困难）
- Emoji 配对，无计时，无计分，纯休闲
- 翻牌 3D 动画 0.4s，匹配成功缩小标记，匹配失败 0.6s 后翻回
- 通关弹窗含分享按钮
- 分享卡片：1200×1600 Canvas，日系清新主题，含 QR 码

---

### 任务 1：HTML 骨架与主题 CSS

**文件：**
- 创建：`index.html`

**接口：**
- 产出：CSS 变量（`--primary`、`--bg-start`、`--bg-end` 等）、页面布局容器、标题区、难度按钮占位、棋盘容器、底部按钮占位、通关弹窗占位、分享遮罩占位

- [ ] **步骤 1：创建 index.html 骨架与 CSS 变量**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>翻牌消消乐</title>
<style>
/* === CSS 变量：日系清新主题 === */
:root {
  --primary: #81C784;
  --primary-light: #A5D6A7;
  --primary-dark: #66BB6A;
  --bg-start: #e8f5e9;
  --bg-end: #fafafa;
  --card-shadow: rgba(0, 0, 0, 0.08);
  --text-main: #333;
  --text-sub: #888;
  --white: #fff;
}

/* === 基础重置 === */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: -apple-system, "PingFang SC", "Microsoft YaHei", "Hiragino Sans GB", sans-serif;
  background: linear-gradient(135deg, var(--bg-start) 0%, var(--bg-end) 100%);
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 20px 16px 40px;
  color: var(--text-main);
  -webkit-tap-highlight-color: transparent;
  user-select: none;
}

/* === 主容器 === */
.game-container {
  width: 100%;
  max-width: 520px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
}

/* === 标题 === */
.game-title {
  font-size: 32px;
  font-weight: 700;
  color: var(--primary-dark);
  letter-spacing: 4px;
  margin-top: 12px;
}

/* === 难度选择 === */
.difficulty-bar {
  display: flex;
  gap: 12px;
}

.diff-btn {
  padding: 8px 24px;
  border: 2px solid var(--primary-light);
  background: var(--white);
  color: var(--primary);
  border-radius: 24px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.25s;
}

.diff-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(129, 199, 132, 0.2);
}

.diff-btn.active {
  background: var(--primary);
  color: var(--white);
  border-color: var(--primary);
}

/* === 棋盘 === */
.board {
  display: grid;
  gap: 10px;
  width: 100%;
  max-width: 480px;
  perspective: 800px;
}

/* 默认 6 列，JS 动态覆盖 */
.board.cols-4 { grid-template-columns: repeat(4, 1fr); }
.board.cols-6 { grid-template-columns: repeat(6, 1fr); }

/* === 底部按钮 === */
.btn-restart {
  padding: 12px 36px;
  border: 1.5px dashed var(--primary-light);
  background: transparent;
  color: var(--primary);
  border-radius: 24px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.25s;
}

.btn-restart:hover {
  border-color: var(--primary);
  color: var(--primary-dark);
  transform: translateY(-2px);
}

.btn-restart:active {
  transform: scale(0.96);
}

/* === 通关弹窗 === */
.win-overlay {
  display: none;
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  z-index: 100;
  align-items: center;
  justify-content: center;
}

.win-overlay.active {
  display: flex;
}

.win-card {
  background: var(--white);
  border-radius: 24px;
  padding: 40px 32px 32px;
  text-align: center;
  box-shadow: 0 16px 48px rgba(0, 0, 0, 0.12);
  max-width: 340px;
  width: 90%;
  animation: popIn 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

@keyframes popIn {
  from { transform: scale(0.8); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

.win-emoji {
  font-size: 56px;
  margin-bottom: 12px;
}

.win-title {
  font-size: 26px;
  font-weight: 700;
  color: var(--primary-dark);
  margin-bottom: 8px;
}

.win-subtitle {
  font-size: 15px;
  color: var(--text-sub);
  margin-bottom: 24px;
}

/* === 分享按钮 === */
.btn-share {
  background: none;
  color: var(--primary);
  border: 1.5px dashed var(--primary-light);
  padding: 12px 28px;
  border-radius: 24px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  width: 100%;
  margin-bottom: 12px;
}

.btn-share:hover {
  border-color: var(--primary);
  color: var(--primary-dark);
}

.btn-share:active {
  transform: scale(0.96);
}

.btn-replay {
  background: var(--primary);
  color: var(--white);
  border: none;
  padding: 12px 28px;
  border-radius: 24px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  width: 100%;
}

.btn-replay:active {
  transform: scale(0.96);
}

/* === 分享遮罩 === */
.share-overlay {
  display: none;
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.85);
  z-index: 200;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 24px;
}

.share-overlay.active {
  display: flex;
}

.share-card-img {
  max-width: 90vw;
  max-height: 70vh;
  border-radius: 16px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
  object-fit: contain;
}

.share-hint {
  color: #ccc;
  font-size: 14px;
  margin-top: 16px;
  text-align: center;
  opacity: 0.85;
}

.share-close {
  color: #ccc;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.25);
  padding: 10px 32px;
  border-radius: 24px;
  font-size: 15px;
  cursor: pointer;
  margin-top: 16px;
}
</style>
</head>
<body>
<div class="game-container">
  <h1 class="game-title">🍀 翻牌消消乐</h1>

  <div class="difficulty-bar" id="difficulty-bar">
    <button class="diff-btn" data-diff="easy">简单</button>
    <button class="diff-btn active" data-diff="normal">普通</button>
    <button class="diff-btn" data-diff="hard">困难</button>
  </div>

  <div class="board cols-6" id="board"></div>

  <button class="btn-restart" id="btn-restart">🔄 重新开始</button>
</div>

<!-- 通关弹窗 -->
<div class="win-overlay" id="win-overlay">
  <div class="win-card">
    <div class="win-emoji">🎉</div>
    <div class="win-title">恭喜通关！</div>
    <div class="win-subtitle" id="win-subtitle">你成功消除了所有卡片</div>
    <button class="btn-share" id="btn-share">📤 分享给朋友</button>
    <button class="btn-replay" id="btn-replay">🔄 再来一局</button>
  </div>
</div>

<!-- 分享卡片遮罩 -->
<div class="share-overlay" id="share-overlay">
  <img class="share-card-img" id="share-card-img" src="" alt="分享卡片">
  <p class="share-hint">长按图片保存，发到微信群</p>
  <button class="share-close" id="share-close">关闭</button>
</div>

<script>
// JS 将在后续任务中填充
</script>
</body>
</html>
```

- [ ] **步骤 2：浏览器中验证**

浏览器打开 `index.html`。预期：显示标题、三个难度按钮（普通高亮）、空白棋盘区域、重新开始按钮。

- [ ] **步骤 3：提交**

```bash
git add index.html
git commit -m "feat: HTML 骨架与日系清新主题 CSS"
```

---

### 任务 2：卡片 CSS 与 3D 翻牌动画

**文件：**
- 修改：`index.html` — 在 `<style>` 中添加卡片相关样式

**接口：**
- 消费：CSS 变量（`--primary`、`--white` 等）、`.board` 容器
- 产出：`.card`、`.card-inner`、`.card-front`、`.card-back`、`.card.flipped`、`.card.matched` 样式，`@keyframes matchPulse` 动画

- [ ] **步骤 1：在 `</style>` 之前添加卡片 CSS**

在 `index.html` 的 `<style>` 标签中，`/* === 分享遮罩 === */` 之前插入：

```css
/* === 卡片 === */
.card {
  aspect-ratio: 1;
  perspective: 600px;
  cursor: pointer;
}

.card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.4s ease;
  transform-style: preserve-3d;
}

.card.flipped .card-inner {
  transform: rotateY(180deg);
}

.card-front,
.card-back {
  position: absolute;
  inset: 0;
  border-radius: 14px;
  backface-visibility: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 卡片正面（未翻开） */
.card-front {
  background: var(--white);
  border: 2px solid var(--primary-light);
  box-shadow: 0 2px 8px var(--card-shadow);
  font-size: 28px;
  color: var(--primary-light);
  font-weight: 700;
}

.card-front:hover {
  box-shadow: 0 4px 16px rgba(129, 199, 132, 0.2);
  border-color: var(--primary);
}

/* 卡片背面（翻开后显示 emoji） */
.card-back {
  background: var(--white);
  border: 2px solid var(--primary-light);
  transform: rotateY(180deg);
  font-size: 36px;
}

/* 已匹配消除的卡片 */
.card.matched {
  pointer-events: none;
}

.card.matched .card-inner {
  transform: rotateY(180deg) scale(0.85);
  opacity: 0.5;
  transition: transform 0.5s ease, opacity 0.5s ease;
}

/* 匹配成功的脉冲动画 */
@keyframes matchPulse {
  0%   { box-shadow: 0 0 0 0 rgba(129, 199, 132, 0.5); }
  70%  { box-shadow: 0 0 0 16px rgba(129, 199, 132, 0); }
  100% { box-shadow: 0 0 0 0 rgba(129, 199, 132, 0); }
}

.card.match-highlight .card-back {
  animation: matchPulse 0.8s ease;
  border-color: var(--primary);
}

/* 匹配失败短暂震动 */
@keyframes shakeBrief {
  0%, 100% { transform: rotateY(180deg) translateX(0); }
  25%      { transform: rotateY(180deg) translateX(-4px); }
  75%      { transform: rotateY(180deg) translateX(4px); }
}

.card.mismatch .card-inner {
  animation: shakeBrief 0.4s ease;
}
```

- [ ] **步骤 2：在 `<script>` 中添加测试卡片，浏览器验证**

在 `<script>` 中添加临时测试代码：

```javascript
// === 临时测试：渲染静态卡片 ===
(function testCards() {
  const board = document.getElementById('board');
  const testEmojis = ['🍎', '🍊', '🌸', '⭐'];
  board.innerHTML = '';
  testEmojis.forEach(emoji => {
    const card = document.createElement('div');
    card.className = 'card';
    card.innerHTML = `
      <div class="card-inner">
        <div class="card-front">?</div>
        <div class="card-back">${emoji}</div>
      </div>`;
    card.addEventListener('click', () => card.classList.toggle('flipped'));
    board.appendChild(card);
  });
})();
```

浏览器打开 `index.html`。预期：4 张卡片排成一行（6 列 Grid），点击卡片有 3D 翻转动画，正面显示 `?`，背面显示 emoji。

- [ ] **步骤 3：提交**

```bash
git add index.html
git commit -m "feat: 卡片 CSS 与 3D 翻牌动画"
```

---

### 任务 3：游戏状态管理与棋盘渲染

**文件：**
- 修改：`index.html` — 替换 `<script>` 内容

**接口：**
- 消费：CSS 类 `.board.cols-4`、`.board.cols-6`
- 产出：`GAME` 状态对象、`EMOJI_POOL` 常量、`DIFFICULTY` 配置、`shuffle()`、`createCards()`、`renderBoard()` 函数

- [ ] **步骤 1：清除临时测试代码，写入游戏核心 JS**

将 `index.html` 的 `<script>` 标签内容替换为：

```javascript
// === 配置 ===
const SHARE_URL = ''; // 部署后填写实际 URL

const DIFFICULTY = {
  easy:   { label: '简单', cols: 4, rows: 4, pairs: 8 },
  normal: { label: '普通', cols: 6, rows: 4, pairs: 12 },
  hard:   { label: '困难', cols: 6, rows: 6, pairs: 18 },
};

const EMOJI_POOL = [
  '🍎', '🍊', '🍋', '🍇', '🍓', '🌸', '🌻', '🍀',
  '🐶', '🐱', '🐰', '🦊', '🐼', '🐨', '🐸', '🦋',
  '🐝', '🌈', '⭐', '🔥', '🎵', '🍕', '🎸', '🚀',
];

// === 游戏状态 ===
const GAME = {
  difficulty: 'normal',
  cards: [],       // { emoji: string, flipped: boolean, matched: boolean }
  firstCard: null, // 索引
  secondCard: null,
  isLocked: false,
};

// === DOM 引用 ===
const boardEl = document.getElementById('board');
const diffBtns = document.querySelectorAll('.diff-btn');

// === 工具函数：Fisher-Yates 洗牌 ===
function shuffle(arr) {
  const a = [...arr];
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}

// === 创建卡片数组 ===
function createCards() {
  const config = DIFFICULTY[GAME.difficulty];
  // 从 emoji 池随机选取所需配对数
  const picked = shuffle(EMOJI_POOL).slice(0, config.pairs);
  // 每对两个，打乱顺序
  const doubled = shuffle([...picked, ...picked]);
  return doubled.map(emoji => ({
    emoji,
    flipped: false,
    matched: false,
  }));
}

// === 渲染棋盘 ===
function renderBoard() {
  const config = DIFFICULTY[GAME.difficulty];
  boardEl.className = `board cols-${config.cols}`;
  boardEl.innerHTML = '';

  GAME.cards.forEach((card, index) => {
    const cardEl = document.createElement('div');
    cardEl.className = 'card';
    cardEl.dataset.index = index;
    if (card.flipped) cardEl.classList.add('flipped');
    if (card.matched) cardEl.classList.add('matched');

    cardEl.innerHTML = `
      <div class="card-inner">
        <div class="card-front">?</div>
        <div class="card-back">${card.emoji}</div>
      </div>
    `;

    cardEl.addEventListener('click', () => handleCardClick(index, cardEl));
    boardEl.appendChild(cardEl);
  });
}

// === 新游戏 ===
function newGame() {
  GAME.cards = createCards();
  GAME.firstCard = null;
  GAME.secondCard = null;
  GAME.isLocked = false;
  renderBoard();
}

// === 翻牌事件处理（占位，任务 4 填充） ===
function handleCardClick(index, cardEl) {
  // 任务 4 实现
}
```

- [ ] **步骤 2：浏览器验证**

在 `<script>` 末尾添加临时启动代码：
```javascript
newGame();
console.log('卡片数组:', GAME.cards);
console.log('棋盘列数:', DIFFICULTY[GAME.difficulty].cols);
```

浏览器打开 `index.html`，打开控制台。预期：棋盘渲染 24 张卡片（普通模式，6 列 × 4 行），每张显示 `?`，控制台输出卡片数组。

- [ ] **步骤 3：提交**

```bash
git add index.html
git commit -m "feat: 游戏状态管理与棋盘渲染"
```

---

### 任务 4：翻牌交互与配对逻辑

**文件：**
- 修改：`index.html` — 实现 `handleCardClick()`，更新 `renderBoard()` 中的点击绑定

**接口：**
- 消费：`GAME` 状态、`GAME.cards[]`、`GAME.isLocked`
- 产出：`handleCardClick(index, cardEl)` 完整实现、`checkMatch()`、`resetSelection()`

- [ ] **步骤 1：实现完整翻牌逻辑**

将 `index.html` 中的 `handleCardClick` 替换为：

```javascript
// === 翻牌事件处理 ===
function handleCardClick(index, cardEl) {
  const card = GAME.cards[index];

  // 忽略：已匹配、已翻开、正在锁定中
  if (card.matched || card.flipped || GAME.isLocked) return;
  // 忽略：点击同一张牌
  if (GAME.firstCard === index) return;

  // 翻开当前牌
  card.flipped = true;
  cardEl.classList.add('flipped');

  if (GAME.firstCard === null) {
    // 第一张牌
    GAME.firstCard = index;
  } else {
    // 第二张牌
    GAME.secondCard = index;
    GAME.isLocked = true;

    checkMatch();
  }
}

// === 检查匹配 ===
function checkMatch() {
  const first = GAME.cards[GAME.firstCard];
  const second = GAME.cards[GAME.secondCard];

  if (first.emoji === second.emoji) {
    // 匹配成功
    onMatchSuccess();
  } else {
    // 匹配失败
    onMatchFail();
  }
}

// === 匹配成功 ===
function onMatchSuccess() {
  const firstIdx = GAME.firstCard;
  const secondIdx = GAME.secondCard;
  const firstEl = document.querySelector(`.card[data-index="${firstIdx}"]`);
  const secondEl = document.querySelector(`.card[data-index="${secondIdx}"]`);

  GAME.cards[firstIdx].matched = true;
  GAME.cards[secondIdx].matched = true;

  // 高亮动画
  firstEl.classList.add('match-highlight');
  secondEl.classList.add('match-highlight');

  // 动画结束后标记为 matched
  setTimeout(() => {
    firstEl.classList.add('matched');
    secondEl.classList.add('matched');
    firstEl.classList.remove('match-highlight');
    secondEl.classList.remove('match-highlight');
    resetSelection();
    checkWin();
  }, 600);
}

// === 匹配失败 ===
function onMatchFail() {
  const firstIdx = GAME.firstCard;
  const secondIdx = GAME.secondCard;
  const firstEl = document.querySelector(`.card[data-index="${firstIdx}"]`);
  const secondEl = document.querySelector(`.card[data-index="${secondIdx}"]`);

  firstEl.classList.add('mismatch');
  secondEl.classList.add('mismatch');

  // 0.6s 后翻回
  setTimeout(() => {
    GAME.cards[firstIdx].flipped = false;
    GAME.cards[secondIdx].flipped = false;
    firstEl.classList.remove('flipped', 'mismatch');
    secondEl.classList.remove('flipped', 'mismatch');
    resetSelection();
  }, 600);
}

// === 重置选牌 ===
function resetSelection() {
  GAME.firstCard = null;
  GAME.secondCard = null;
  GAME.isLocked = false;
}

// === 检查是否通关（占位） ===
function checkWin() {
  // 任务 6 实现
}

// === 暴露 newGame 到全局，供按钮调用 ===
window.newGame = newGame;
```

- [ ] **步骤 2：绑定重新开始按钮**

在 `<script>` 末尾添加：

```javascript
// === 按钮事件绑定 ===
document.getElementById('btn-restart').addEventListener('click', () => {
  // 关闭通关弹窗（如果打开着）
  document.getElementById('win-overlay').classList.remove('active');
  newGame();
});

// === 启动游戏 ===
newGame();
```

- [ ] **步骤 3：浏览器验证**

浏览器打开 `index.html`。预期：
- 点击卡片有 3D 翻转，显示 emoji
- 点击两张相同 emoji 的牌 → 脉冲高亮 → 缩小半透明标记消除
- 点击两张不同的牌 → 震动动画 → 0.6s 后翻回
- 翻牌期间不能点击其他牌（锁定）
- 点击"重新开始"按钮重洗牌面

- [ ] **步骤 4：提交**

```bash
git add index.html
git commit -m "feat: 翻牌交互与配对逻辑"
```

---

### 任务 5：难度选择器

**文件：**
- 修改：`index.html` — 添加难度按钮事件

**接口：**
- 消费：`DIFFICULTY` 配置、`diffBtns` DOM 引用
- 产出：`switchDifficulty(diff)` 函数

- [ ] **步骤 1：实现难度切换**

在与按钮事件绑定相同的区域，`newGame()` 启动之前添加：

```javascript
// === 难度切换 ===
diffBtns.forEach(btn => {
  btn.addEventListener('click', () => {
    const diff = btn.dataset.diff;
    if (diff === GAME.difficulty) return;

    // 更新 UI
    diffBtns.forEach(b => b.classList.remove('active'));
    btn.classList.add('active');

    // 更新状态并重开
    GAME.difficulty = diff;
    document.getElementById('win-overlay').classList.remove('active');
    newGame();
  });
});
```

- [ ] **步骤 2：浏览器验证**

浏览器打开 `index.html`。预期：
- 点击"简单"→ 棋盘变为 4×4（16 张牌）
- 点击"普通"→ 棋盘变为 6×4（24 张牌，默认）
- 点击"困难"→ 棋盘变为 6×6（36 张牌）
- 切换难度时自动重新发牌
- 选中按钮样式切换正常

- [ ] **步骤 3：提交**

```bash
git add index.html
git commit -m "feat: 难度选择器"
```

---

### 任务 6：通关弹窗

**文件：**
- 修改：`index.html` — 实现 `checkWin()`、通关弹窗交互

**接口：**
- 消费：`#win-overlay`、`#btn-replay` 元素
- 产出：`checkWin()` 完整实现、`showWin()`、通关弹窗的事件绑定

- [ ] **步骤 1：实现通关检测与弹窗**

将 `checkWin()` 占位替换为：

```javascript
// === 检查是否通关 ===
function checkWin() {
  const allMatched = GAME.cards.every(card => card.matched);
  if (!allMatched) return;

  // 延迟弹出，让最后两张牌的消除动画播完
  setTimeout(() => {
    showWin();
  }, 400);
}

// === 显示通关弹窗 ===
function showWin() {
  const config = DIFFICULTY[GAME.difficulty];
  document.getElementById('win-subtitle').textContent =
    `你成功消除了全部 ${config.pairs} 对卡片！`;
  document.getElementById('win-overlay').classList.add('active');
}
```

- [ ] **步骤 2：绑定通关弹窗按钮**

在按钮事件绑定区域添加：

```javascript
// === 通关弹窗按钮 ===
document.getElementById('btn-replay').addEventListener('click', () => {
  document.getElementById('win-overlay').classList.remove('active');
  newGame();
});

// 点击遮罩关闭（但不关闭——弹窗只有按钮退出，防止误触）
// 移除这行即可：通关弹窗无法通过点击背景关闭
```

同时在 `newGame()` 中添加关闭弹窗逻辑（如果弹窗打开着）：

确保 `newGame()` 函数中已有：
```javascript
document.getElementById('win-overlay').classList.remove('active');
```

- [ ] **步骤 3：浏览器验证**

浏览器打开 `index.html`。预期：
- 消除所有配对后，弹窗从屏幕中央弹出（`popIn` 动画）
- 显示"🎉 恭喜通关！"和"你成功消除了全部 X 对卡片！"
- 点击"🔄 再来一局"关闭弹窗并重新发牌
- 点击背景区域不关闭弹窗

- [ ] **步骤 4：提交**

```bash
git add index.html
git commit -m "feat: 通关弹窗"
```

---

### 任务 7：分享卡片生成与分享遮罩

**文件：**
- 修改：`index.html` — 添加 Canvas 分享卡片绘制代码、分享按钮事件、遮罩交互

**接口：**
- 消费：`SHARE_URL`、通关弹窗中的 `#btn-share`
- 产出：`generateShareCard()`、`shareGame()`、`showShareOverlay()`、`hideShareCard()`、`rr()` 辅助函数

- [ ] **步骤 1：添加 Canvas 绘制辅助函数和分享卡片生成**

在 `<script>` 中，`newGame()` 启动之前添加：

```javascript
// === 分享卡片绘制 ===

// 圆角矩形路径辅助
function rr(ctx, x, y, w, h, r) {
  ctx.beginPath();
  ctx.moveTo(x + r, y);
  ctx.lineTo(x + w - r, y);
  ctx.arcTo(x + w, y, x + w, y + r, r);
  ctx.lineTo(x + w, y + h - r);
  ctx.arcTo(x + w, y + h, x + w - r, y + h, r);
  ctx.lineTo(x + r, y + h);
  ctx.arcTo(x, y + h, x, y + h - r, r);
  ctx.lineTo(x, y + r);
  ctx.arcTo(x, y, x + r, y, r);
  ctx.closePath();
}

function generateShareCard() {
  const canvas = document.createElement('canvas');
  canvas.width = 1200;
  canvas.height = 1600;
  const ctx = canvas.getContext('2d');
  const W = 1200, H = 1600;

  // === 日系清新主题色 ===
  const theme = {
    bg: '#E8F5E9',
    bgGradTop: '#E8F5E9',
    bgGradMid: '#FFF9C4',
    bgGradBot: '#FFFDE7',
    cardBg: '#FFF',
    accent: '#81C784',
    accentLight: '#A5D6A7',
    accentShadow: 'rgba(0,0,0,0.08)',
    textTitle: '#81C784',
    textSub: '#999',
    textFeature: '#777',
    textHint: '#AAA',
    dividerColor: '#A5D6A7',
  };

  const config = DIFFICULTY[GAME.difficulty];

  // 背景渐变
  const g = ctx.createLinearGradient(0, 0, 0, H);
  g.addColorStop(0, theme.bgGradTop);
  g.addColorStop(0.4, theme.bgGradMid);
  g.addColorStop(1, theme.bgGradBot);
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, W, H);

  // 装饰圆
  ctx.globalAlpha = 0.12;
  [
    { x: 160, y: 360, r: 200, f: theme.accent },
    { x: 960, y: 280, r: 140, f: theme.accentLight },
    { x: 1040, y: 1120, r: 260, f: '#FFE082' },
    { x: 200, y: 1240, r: 120, f: theme.accent },
    { x: 600, y: 200, r: 90, f: theme.accentLight },
  ].forEach(d => {
    ctx.fillStyle = d.f;
    ctx.beginPath();
    ctx.arc(d.x, d.y, d.r, 0, Math.PI * 2);
    ctx.fill();
  });
  ctx.globalAlpha = 1;

  // 白色内容卡片
  ctx.fillStyle = theme.cardBg;
  ctx.shadowColor = theme.accentShadow;
  ctx.shadowBlur = 80;
  ctx.shadowOffsetY = 12;
  rr(ctx, 90, 240, W - 180, H - 600, 56);
  ctx.fill();
  ctx.shadowColor = 'transparent';
  ctx.shadowBlur = 0;
  ctx.shadowOffsetY = 0;

  // 标题
  ctx.fillStyle = theme.textTitle;
  ctx.font = "bold 88px -apple-system, 'PingFang SC', 'Microsoft YaHei', sans-serif";
  ctx.textAlign = 'center';
  ctx.fillText('翻牌消消乐', W / 2, 410);

  // Emoji 行
  ctx.font = '80px sans-serif';
  ctx.fillText('🍎 🍊 🌸 ⭐ 🐶 🦊 🍀', W / 2, 550);

  // 副标题
  ctx.fillStyle = theme.textSub;
  ctx.font = "42px -apple-system, 'PingFang SC', sans-serif";
  ctx.fillText('日系清新记忆配对小游戏', W / 2, 660);

  // 虚线分割线
  ctx.strokeStyle = theme.dividerColor;
  ctx.lineWidth = 3;
  ctx.setLineDash([16, 12]);
  ctx.beginPath();
  ctx.moveTo(190, 740);
  ctx.lineTo(W - 190, 740);
  ctx.stroke();
  ctx.setLineDash([]);

  // 特性列表
  ctx.fillStyle = theme.textFeature;
  ctx.font = "40px -apple-system, 'PingFang SC', sans-serif";
  const features = [
    `🎯 难度：${config.label}`,
    `🃏 消除：${config.pairs} 对`,
    `🌸 日系清新风格`,
  ];
  features.forEach((f, i) => ctx.fillText(f, W / 2, 840 + i * 90));

  return canvas;
}
```

- [ ] **步骤 2：添加分享触发与遮罩逻辑**

接着在 `<script>` 末尾（`newGame()` 调用之后）添加：

```javascript
// === 分享功能 ===

function drawQRAndShow(canvas) {
  const ctx = canvas.getContext('2d');
  const W = 1200, H = 1600;
  const themeAccent = '#81C784';

  const qr = new Image();
  qr.crossOrigin = 'anonymous';
  qr.onload = () => {
    const qs = 260, qx = (W - qs) / 2, qy = H - 380;
    ctx.fillStyle = '#FFF';
    rr(ctx, qx - 24, qy - 24, qs + 48, qs + 48, 32);
    ctx.fill();
    ctx.strokeStyle = themeAccent;
    ctx.lineWidth = 3;
    rr(ctx, qx - 24, qy - 24, qs + 48, qs + 48, 32);
    ctx.stroke();
    ctx.drawImage(qr, qx, qy, qs, qs);

    // 底部提示
    ctx.fillStyle = '#AAA';
    ctx.font = "28px -apple-system, 'PingFang SC', sans-serif";
    ctx.fillText('扫码或长按识别 · 和朋友一起玩', W / 2, H - 48);

    showShareOverlay(canvas);
  };
  qr.onerror = () => showShareOverlay(canvas);
  qr.src = 'https://api.qrserver.com/v1/create-qr-code/?size=260x260&data='
    + encodeURIComponent(SHARE_URL || window.location.href)
    + '&margin=8';
}

function shareGame() {
  // 先关闭通关弹窗
  document.getElementById('win-overlay').classList.remove('active');

  const canvas = generateShareCard();
  drawQRAndShow(canvas);
}

function showShareOverlay(canvas) {
  const img = document.getElementById('share-card-img');
  if (img) img.src = canvas.toDataURL('image/png');
  const overlay = document.getElementById('share-overlay');
  if (overlay) {
    overlay.classList.add('active');
    document.body.style.overflow = 'hidden';
  }
}

function hideShareCard() {
  const overlay = document.getElementById('share-overlay');
  if (overlay) {
    overlay.classList.remove('active');
    document.body.style.overflow = '';
  }
}

// === 分享遮罩事件绑定 ===
document.getElementById('btn-share').addEventListener('click', shareGame);
document.getElementById('share-close').addEventListener('click', hideShareCard);
document.getElementById('share-overlay').addEventListener('click', function(e) {
  if (e.target === this) hideShareCard();
});
```

- [ ] **步骤 3：将分享函数暴露到全局**

确保 `window` 对象上已有分享相关函数（如果内联在 `<script>` 中的函数已在全局作用域则无需额外操作）。

- [ ] **步骤 4：浏览器验证**

浏览器打开 `index.html`。通关（消除所有配对）后：
- 点击"📤 分享给朋友"按钮
- 通关弹窗关闭，全屏遮罩弹出（深色背景）
- 显示完整的 1200×1600 分享卡片图片
- 图片含：标题"翻牌消消乐"、emoji 装饰行、副标题、当前难度和消除对数、QR 码（需联网）
- 底部显示"长按图片保存，发到微信群"提示
- 点击"关闭"或背景区域关闭遮罩

- [ ] **步骤 5：提交**

```bash
git add index.html
git commit -m "feat: 分享卡片生成与分享遮罩"
```

---

### 任务 8：响应式适配与细节润色

**文件：**
- 修改：`index.html` — 添加媒体查询、微调样式、处理边缘情况

**接口：**
- 无新接口，仅样式和交互细节调整

- [ ] **步骤 1：在 `</style>` 之前添加响应式媒体查询**

```css
/* === 响应式 === */
@media (max-width: 480px) {
  .game-title {
    font-size: 26px;
    letter-spacing: 2px;
  }

  .board {
    gap: 6px;
  }

  .card-front {
    font-size: 22px;
  }

  .card-back {
    font-size: 28px;
  }

  .diff-btn {
    padding: 6px 18px;
    font-size: 14px;
  }

  .board.cols-6 {
    gap: 4px;
  }
}

/* 大屏适配：限制棋盘宽度 */
@media (min-width: 600px) {
  .board {
    max-width: 480px;
  }
}
```

- [ ] **步骤 2：确保 `newGame()` 在关卡结束时正确处理弹窗状态**

检查 `newGame()` 函数确保包含：
```javascript
function newGame() {
  GAME.cards = createCards();
  GAME.firstCard = null;
  GAME.secondCard = null;
  GAME.isLocked = false;
  document.getElementById('win-overlay').classList.remove('active');
  renderBoard();
}
```

- [ ] **步骤 3：浏览器全功能验证**

在桌面和移动端（或 DevTools 设备模拟）打开 `index.html`：

- [ ] 三个难度切换正常，棋盘列数正确
- [ ] 翻牌 3D 动画流畅，0.4s
- [ ] 匹配成功脉冲高亮 → 缩小半透明
- [ ] 匹配失败 0.6s 翻回
- [ ] 翻牌期间锁定，
- [ ] 全部消除后通关弹窗弹出（`popIn` 动画）
- [ ] 分享按钮 → 生成卡片 → 遮罩显示 → 关闭
- [ ] 重新开始 / 再来一局 正常工作
- [ ] 移动端（375px 宽）卡片和文字合适

- [ ] **步骤 4：提交**

```bash
git add index.html
git commit -m "feat: 响应式适配与细节润色"
```

---

## 验证清单

在实现完成后，逐一确认：

- [ ] 零依赖：文件只含 HTML/CSS/JS，无 `<link>` 或外部 `<script>`
- [ ] 无 console 报错
- [ ] 三档难度：4×4、6×4、6×6，切换正常
- [ ] 翻牌动画 3D rotateY，0.4s ease
- [ ] 配对成功动画：脉冲高亮 → 缩小半透明（0.5s）
- [ ] 配对失败动画：震动 → 翻回（0.6s）
- [ ] 翻牌期间点击锁定
- [ ] 通关弹窗 `popIn` 动画
- [ ] 分享卡片：1200×1600，日系清新主题，含 QR 码
- [ ] 分享遮罩：长按保存提示，点击关闭
- [ ] 移动端 375px 宽度正常显示
