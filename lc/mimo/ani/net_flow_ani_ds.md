这是网络流最大流动画页面，采用与 MST / 最短路页面一致的风格。包含 Edmonds-Karp 和 Dinic 两种算法，可视化残量网络、反向边和增广路径。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>网络流 · Edmonds-Karp / Dinic 最大流动画</title>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/highlightjs/cdn-release@11.9.0/build/styles/atom-one-light.min.css">
<script src="https://cdn.jsdelivr.net/gh/highlightjs/cdn-release@11.9.0/build/highlight.min.js"></script>
<script src="https://cdn.jsdelivr.net/gh/highlightjs/cdn-release@11.9.0/build/languages/python.min.js"></script>

<style>
  * { box-sizing: border-box; }
  body {
    margin: 0; padding: 24px 16px 60px;
    background: #eef2f7;
    font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Microsoft YaHei", sans-serif;
    color: #1f2933;
    display: flex; flex-direction: column; align-items: center;
  }
  .wrap { width: min(1100px, 100%); }
  h1 { font-size: 19px; font-weight: 600; margin: 0 0 6px; color: #243b53; }
  .subtitle { font-size: 13px; color: #627d98; margin: 0 0 16px; }
  .subtitle code { background:#e3edf7; padding:1px 5px; border-radius:4px; font-size:12px; }
  .subtitle b { color: #2a7fd0; }

  .intro {
    margin-bottom: 16px; background: #fff; border-radius: 12px;
    padding: 16px 20px 18px; box-shadow: 0 4px 16px rgba(30, 50, 80, .07);
  }
  .intro h2 {
    font-size: 14.5px; font-weight: 700; color: #243b53; margin: 0 0 10px;
    display: flex; align-items: center; gap: 8px;
  }
  .intro h2::before {
    content: ''; width: 4px; height: 16px; background: #2a7fd0; border-radius: 2px;
  }
  .intro-lead { font-size: 13px; line-height: 1.8; color: #334e68; margin: 0 0 14px; }
  .intro-lead code {
    background: #eef4fa; color: #2369ad; padding: 1px 6px;
    border-radius: 3px; font-size: 12px; font-weight: 600;
  }
  .intro-lead b { color: #2a7fd0; }
  .intro-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; }
  @media (max-width: 720px) { .intro-grid { grid-template-columns: 1fr; } }
  .intro-card {
    border: 1.5px solid #dbe4ec; border-radius: 10px;
    padding: 10px 12px 11px; background: #fafcfe;
  }
  .intro-card.highlight { border-color: #ffb84d; background: #fffbf2; }
  .intro-card .num {
    display: inline-block; font-size: 11px; font-weight: 800;
    color: #2a7fd0; background: #e3edf7;
    padding: 1px 7px; border-radius: 4px; margin-bottom: 6px;
  }
  .intro-card.highlight .num { color: #b45a12; background: #ffebd2; }
  .intro-card .title { font-size: 13px; font-weight: 700; color: #243b53; margin-bottom: 5px; }
  .intro-card .desc { font-size: 12px; color: #334e68; line-height: 1.7; }
  .intro-card .desc code {
    background: #eef4fa; color: #2369ad; padding: 0 4px;
    border-radius: 3px; font-size: 11px;
  }
  .intro-card .desc b { color: #d64545; }

  svg#stage {
    width: 100%; height: auto; display: block;
    background: #fff; border-radius: 14px;
    box-shadow: 0 6px 24px rgba(30, 50, 80, .10);
  }

  /* 图节点 */
  .nf-node .node-bg {
    fill: #d5e8ff; stroke: #4a90d9; stroke-width: 2.5;
    transition: fill .3s, stroke .3s, stroke-width .3s;
  }
  .nf-node .node-id {
    font-size: 20px; font-weight: 800; fill: #2369ad;
    pointer-events: none; user-select: none;
  }
  .nf-node .node-level {
    font-family: ui-monospace, monospace;
    font-size: 11px; font-weight: 800; fill: #ffffff;
    pointer-events: none; user-select: none;
  }
  .nf-node .level-bg { fill: #7c5cd6; }
  .nf-node.source .node-bg { fill: #ffd0d0; stroke: #d64545; stroke-width: 3.8; }
  .nf-node.source .node-id { fill: #b33232; }
  .nf-node.sink .node-bg { fill: #b0e8c2; stroke: #2f7a34; stroke-width: 3.8; }
  .nf-node.sink .node-id { fill: #1c5e28; }
  .nf-node.current .node-bg { fill: #ffd98a; stroke: #e09b00; stroke-width: 4; }
  .nf-node.current .node-id { fill: #a35e00; }
  .nf-node.in-path .node-bg { fill: #ffe9a8; stroke: #e09b00; stroke-width: 3.6; }
  .nf-node.in-path .node-id { fill: #a35e00; }

  /* 图边 */
  .nf-edge {
    stroke: #cbd2d9; stroke-width: 2.2;
    fill: none;
    transition: stroke .3s, stroke-width .3s;
  }
  .nf-edge.has-flow {
    stroke: #3b82c4;
    stroke-width: 3;
  }
  .nf-edge.exploring {
    stroke: #e09b00; stroke-width: 3.6;
    stroke-dasharray: 8 5;
    animation: dashflow .9s linear infinite;
  }
  .nf-edge.path {
    stroke: #e09b00; stroke-width: 5.5;
    stroke-dasharray: 10 6;
    animation: dashflow .7s linear infinite;
  }
  .nf-edge.augmented {
    stroke: #2f7a34; stroke-width: 5;
  }
  .nf-edge.reverse {
    stroke: #7c5cd6; stroke-width: 2.5;
    stroke-dasharray: 5 4;
    opacity: 0.75;
  }
  @keyframes dashflow { to { stroke-dashoffset: -14; } }

  .edge-w {
    font-family: ui-monospace, monospace;
    font-size: 13px; font-weight: 800;
    fill: #4a5568;
    pointer-events: none; user-select: none;
    paint-order: stroke;
    stroke: #ffffff; stroke-width: 6;
    stroke-linejoin: round;
  }
  .edge-w.exploring { fill: #b45a12; }
  .edge-w.path      { fill: #8a5a00; }
  .edge-w.augmented { fill: #1c5e28; }
  .edge-w.has-flow  { fill: #1c5e98; }

  /* 面板 */
  .panel {
    margin-top: 12px; background: #fff; border-radius: 12px;
    padding: 6px 18px; box-shadow: 0 4px 16px rgba(30, 50, 80, .07);
    font-size: 14px;
  }
  .panel-row {
    display: flex; flex-wrap: wrap; align-items: center;
    gap: 10px; padding: 10px 0;
    border-bottom: 1px dashed #eef2f7;
  }
  .panel-row:last-child { border-bottom: none; }
  .row-label {
    display: inline-flex; align-items: center;
    font-size: 12.5px; font-weight: 700; color: #243b53;
    min-width: 88px;
  }
  .row-label.algo { color: #7c5cd6; }
  .row-hint {
    font-size: 11.5px; color: #829ab1; margin-left: 6px;
    line-height: 1.5;
  }
  .row-hint code {
    background: #f0f4f8; color: #4a5568;
    padding: 0 5px; border-radius: 3px; font-size: 11px;
  }
  .row-hint b { color: #d64545; font-weight: 700; }

  select {
    font: inherit; font-size: 13px; padding: 5px 10px;
    border-radius: 7px; border: 1px solid #d8c8ff;
    background: #f9f6ff; color: #7c5cd6;
    font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
    min-width: 340px;
    font-weight: 700;
  }
  select:disabled { opacity: .6; }

  button {
    font: inherit; font-size: 13px; padding: 6px 14px;
    border-radius: 8px; border: 1px solid #bcccdc;
    background: #f8fafc; color: #243b53; cursor: pointer; transition: .18s;
  }
  button:hover { background: #e8f0f8; border-color: #8faec9; }
  button:active { transform: translateY(1px); }
  button:disabled { opacity: .5; cursor: not-allowed; }
  button.primary { background: #2a7fd0; border-color: #2a7fd0; color: #fff; }
  button.danger  { background: #d64545; border-color: #d64545; color: #fff; }
  button.success { background: #57a957; border-color: #57a957; color: #fff; }
  button.success:hover { background: #3d8a3d; border-color: #3d8a3d; }

  /* 信息面板（流值） */
  .info-panel {
    margin-top: 12px; background: #fff; border-radius: 12px;
    padding: 12px 18px; box-shadow: 0 4px 16px rgba(30, 50, 80, .07);
    display: flex; flex-wrap: wrap; gap: 20px; align-items: center;
    font-size: 13.5px;
  }
  .info-panel .item {
    display: flex; align-items: center; gap: 8px;
  }
  .info-panel .item .title {
    color: #627d98; font-weight: 600;
  }
  .info-panel .item .stat {
    font-family: ui-monospace, monospace;
    font-size: 15px; font-weight: 800;
    background: #eaf9ee; color: #2f7a34;
    padding: 3px 12px; border-radius: 6px;
  }
  .info-panel .item .stat.round {
    background: #f2ecff; color: #5b3db8;
  }
  .info-panel .item .stat.path {
    background: #fffbf2; color: #a35e00;
  }

  .demo-bar {
    margin-top: 12px; background: #fff; border-radius: 12px;
    padding: 12px 18px; box-shadow: 0 4px 16px rgba(30, 50, 80, .07);
    display: flex; flex-wrap: wrap; gap: 10px 14px; align-items: center;
    font-size: 13.5px;
    border-left: 4px solid #57a957;
  }
  .demo-bar .title {
    font-weight: 700; color: #2f7a34;
    display: flex; align-items: center; gap: 6px;
  }
  .demo-bar .mode-label { color: #627d98; font-size: 12.5px; }
  .demo-bar button { font-size: 13px; padding: 6px 14px; }
  .radio-group {
    display: inline-flex; border: 1px solid #bcccdc;
    border-radius: 8px; overflow: hidden; background: #f8fafc;
  }
  .radio-group label {
    font-size: 12.5px; padding: 5px 12px; color: #627d98;
    cursor: pointer; user-select: none; transition: .15s;
    display: inline-flex; align-items: center; gap: 5px;
  }
  .radio-group label:hover { color: #243b53; }
  .radio-group input[type="radio"] { display: none; }
  .radio-group label.active { background: #57a957; color: #fff; }
  .radio-group label.active:hover { color: #fff; }

  .speed-wrap {
    display: flex; align-items: center; gap: 8px; margin-left: auto;
  }
  .speed-wrap .label { color: #627d98; font-size: 12.5px; }
  .speed-wrap input[type="range"] { width: 130px; accent-color: #2a7fd0; }
  .speed-wrap .val {
    font-family: ui-monospace, monospace;
    font-size: 12.5px; color: #2369ad;
    background: #eef4fa; padding: 2px 8px; border-radius: 5px;
    min-width: 46px; text-align: center;
  }

  .result {
    margin-top: 12px; padding: 12px 18px;
    background: #fff; border-radius: 12px;
    border-left: 4px solid #2a7fd0;
    font-size: 14px; color: #334e68;
    min-height: 46px; display: flex; align-items: center;
    box-shadow: 0 4px 16px rgba(30, 50, 80, .07);
    font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
  }

  /* 演示记录 */
  .log-module {
    margin-top: 12px; background: #fff; border-radius: 12px;
    box-shadow: 0 4px 16px rgba(30, 50, 80, .07); overflow: hidden;
  }
  .log-module > summary {
    padding: 14px 20px; cursor: pointer; font-size: 14.5px;
    font-weight: 700; color: #243b53; list-style: none;
    display: flex; align-items: center; gap: 8px;
    user-select: none; border-bottom: 1px solid #eef2f7;
  }
  .log-module > summary::-webkit-details-marker { display: none; }
  .log-module > summary::before {
    content: '▶'; font-size: 10px; color: #829ab1;
    transition: transform .2s; display: inline-block;
  }
  .log-module[open] > summary::before { transform: rotate(90deg); }
  .log-module > summary:hover { background: #f8fafc; }
  .log-module > summary .count {
    font-size: 11.5px; font-weight: 600;
    background: #eef4fa; color: #2369ad;
    padding: 2px 8px; border-radius: 5px;
    font-family: ui-monospace, monospace;
    margin-left: auto;
  }
  .log-module > summary .hint {
    font-size: 11.5px; font-weight: 400; color: #829ab1;
  }
  .log-body {
    max-height: 460px; overflow-y: auto;
    padding: 10px 20px 16px;
    background: #fbfdff;
  }
  .log-line {
    display: grid; grid-template-columns: 56px 1fr;
    gap: 12px; padding: 6px 0;
    border-bottom: 1px dashed #eef2f7;
    font-size: 12.5px;
    font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
    transition: background .15s;
  }
  .log-line:hover { background: #f0f7ff; }
  .log-line:last-child { border-bottom: none; }
  .log-line .idx {
    color: #829ab1; font-weight: 700; text-align: right;
  }
  .log-line .msg { color: #334e68; word-break: break-word; }
  .log-line.cur {
    background: #fffbf2;
    border-left: 3px solid #e09b00;
    margin-left: -3px; padding-left: 3px;
  }
  .log-line.cur .idx { color: #b45a12; }
  .log-line.cur .msg { color: #8a5a00; font-weight: 600; }
  .log-empty {
    font-size: 12.5px; color: #829ab1; font-style: italic;
    padding: 16px 0; text-align: center;
  }

  /* 知识模块 */
  .knowledge {
    margin-top: 16px; background: #fff; border-radius: 12px;
    padding: 16px 20px 18px; box-shadow: 0 4px 16px rgba(30, 50, 80, .07);
  }
  .knowledge h2 {
    font-size: 14.5px; font-weight: 700; color: #243b53; margin: 0 0 14px;
    display: flex; align-items: center; gap: 8px;
  }
  .knowledge h2::before {
    content: ''; width: 4px; height: 16px; background: #2a7fd0; border-radius: 2px;
  }
  .knowledge h2 .hint {
    font-size: 11.5px; font-weight: 400; color: #829ab1; margin-left: auto;
  }
  .k-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px; }
  @media (max-width: 720px) { .k-grid { grid-template-columns: 1fr; } }
  .k-card {
    border: 1.5px solid #dbe4ec; border-radius: 10px;
    padding: 12px 14px 13px; background: #fafcfe;
  }
  .k-card.key-point { border-color: #ffb84d; background: #fffbf2; }
  .k-card.compare-point { border-color: #7c5cd6; background: #f9f6ff; }
  .k-head { display: flex; align-items: center; gap: 8px; margin-bottom: 8px; }
  .k-icon {
    width: 26px; height: 26px; border-radius: 8px;
    background: linear-gradient(135deg, #d5e8ff 0%, #b9daff 100%);
    display: flex; align-items: center; justify-content: center; font-size: 15px;
  }
  .k-card.key-point .k-icon { background: linear-gradient(135deg, #ffe9a8 0%, #ffd98a 100%); }
  .k-card.compare-point .k-icon { background: linear-gradient(135deg, #e0d5ff 0%, #c9b8ff 100%); }
  .k-title { font-weight: 700; color: #243b53; font-size: 13.5px; }
  .k-card.key-point .k-title { color: #8a5a00; }
  .k-card.compare-point .k-title { color: #5b3db8; }
  .k-lines { list-style: none; margin: 0; padding: 0; }
  .k-lines li {
    font-size: 12px; color: #334e68; line-height: 1.7;
    padding-left: 14px; position: relative;
  }
  .k-lines li::before {
    content: '·'; position: absolute; left: 4px; top: 0;
    color: #2a7fd0; font-weight: 700; font-size: 16px; line-height: 1.4;
  }
  .k-card.key-point .k-lines li::before { color: #e09b00; }
  .k-card.compare-point .k-lines li::before { color: #7c5cd6; }
  .k-lines li code {
    background: #eef4fa; color: #2369ad;
    padding: 0 4px; border-radius: 3px; font-size: 11px;
  }
  .k-lines li b { color: #d64545; }

  /* 代码模块 */
  .code-module {
    margin-top: 16px; background: #fff; border-radius: 12px;
    box-shadow: 0 4px 16px rgba(30, 50, 80, .07); overflow: hidden;
  }
  .code-module > summary {
    padding: 14px 20px; cursor: pointer; font-size: 14.5px;
    font-weight: 700; color: #243b53; list-style: none;
    display: flex; align-items: center; gap: 8px;
    user-select: none; border-bottom: 1px solid #eef2f7;
  }
  .code-module > summary::-webkit-details-marker { display: none; }
  .code-module > summary::before {
    content: '▶'; font-size: 10px; color: #829ab1;
    transition: transform .2s; display: inline-block;
  }
  .code-module[open] > summary::before { transform: rotate(90deg); }
  .code-module > summary:hover { background: #f8fafc; }
  .code-module > summary .badge {
    font-size: 11px; font-weight: 700;
    background: #e2f4e2; color: #2f7a34;
    padding: 2px 8px; border-radius: 5px;
  }
  .code-body {
    padding: 18px 22px; background: #fbfdff;
    max-height: 640px; overflow: auto;
  }
  .code-body pre { margin: 0; }
  .code-body code {
    font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
    font-size: 12.5px; line-height: 1.7;
    background: transparent;
  }
</style>
</head>
<body>
<div class="wrap">
  <h1>网络流 · Edmonds-Karp / Dinic 最大流动画</h1>
  <p class="subtitle">
    源点 <b>S</b> → 汇点 <b>T</b> · 每条边 (u,v) 容量 c · 求最大流量 · 边上的数字是 <code>flow / cap</code>
  </p>

  <div class="intro">
    <h2>📖 原理简介 · 残量网络与增广路径</h2>
    <p class="intro-lead">
      网络流问题：给定有向图，每条边有<b>容量</b> c，求从源点 s 到汇点 t 能通过的最大流量。
      核心是<b>残量网络</b>：正向边剩余 <code>c − f</code>，反向边剩余 <code>f</code>（表示"可以回退的流量"）。
      只要残量网络中还存在从 s 到 t 的<b>增广路径</b>，就沿这条路径推流，流量增加"路径瓶颈"那么多。
      反复增广，直到没有增广路径为止。此时流量达到最大，且等于<b>最小割</b>容量（Max-Flow Min-Cut 定理）。
    </p>
    <div class="intro-grid">
      <div class="intro-card highlight">
        <div class="num">核心 1</div>
        <div class="title">残量网络</div>
        <div class="desc">
          每条边 (u, v, c) 对应两条残量边：<br>
          · 正向：<code>c − f</code>（还能再推多少）<br>
          · 反向：<code>f</code>（已经推了多少，可以回退）<br>
          图中边上的 <code>flow/cap</code> 就是当前状态。
        </div>
      </div>
      <div class="intro-card highlight">
        <div class="num">核心 2</div>
        <div class="title">增广路径 Augmenting Path</div>
        <div class="desc">
          残量网络中一条从 s 到 t 的路径。<br>
          路径的<b>瓶颈</b> = 路径上所有边剩余容量的最小值。<br>
          沿路径推流，流量 +瓶颈，<b>反向边 +瓶颈</b>。
        </div>
      </div>
      <div class="intro-card highlight">
        <div class="num">核心 3</div>
        <div class="title">Max-Flow Min-Cut</div>
        <div class="desc">
          最大流 = 最小割容量。<br>
          割：把 V 分成含 s 和含 t 两部分，割容量 = 从 s 侧到 t 侧的边容量和。<br>
          这个定理是网络流的理论基石。
        </div>
      </div>
      <div class="intro-card">
        <div class="num">算法 1</div>
        <div class="title">Edmonds-Karp（EK）</div>
        <div class="desc">
          每轮用 <b>BFS</b> 找一条<b>最短</b>增广路径（边数最少）。<br>
          复杂度 <code>O(VE²)</code>。<br>
          简单可靠，是最容易实现的 FF 优化。
        </div>
      </div>
      <div class="intro-card">
        <div class="num">算法 2</div>
        <div class="title">Dinic</div>
        <div class="desc">
          每轮 <b>BFS 分层</b>，只在"层数 +1"的边上 DFS，找<b>阻塞流</b>。<br>
          复杂度 <code>O(V²E)</code>，二分图上 <code>O(E√V)</code>。<br>
          现代最常用的最大流算法。
        </div>
      </div>
      <div class="intro-card">
        <div class="num">教学图</div>
        <div class="title">本页面用图</div>
        <div class="desc">
          6 节点 8 边，源点 S，汇点 T。<br>
          S→A(10)、S→B(10)<br>
          A→C(4)、A→D(8)、B→D(9)<br>
          C→D(3)、C→T(6)、D→T(10)<br>
          最大流 = 14
        </div>
      </div>
    </div>
  </div>

  <svg id="stage" viewBox="0 0 900 700" xmlns="http://www.w3.org/2000/svg"></svg>

  <div class="info-panel">
    <div class="item">
      <span class="title">当前流量</span>
      <span class="stat" id="totalFlow">0 / 14</span>
    </div>
    <div class="item">
      <span class="title">增广次数</span>
      <span class="stat path" id="pathCount">0</span>
    </div>
    <div class="item">
      <span class="title">轮次</span>
      <span class="stat round" id="roundInfo">—</span>
    </div>
  </div>

  <div class="panel">
    <div class="panel-row">
      <span class="row-label algo">算法选择</span>
      <select id="algoSel">
        <option value="ek">Edmonds-Karp —— BFS 找最短增广路径</option>
        <option value="dinic">Dinic —— BFS 分层 + DFS 阻塞流</option>
      </select>
    </div>
    <div class="panel-row">
      <span class="row-label algo">颜色说明</span>
      <span class="row-hint">
        <b>蓝色实线</b>=有流量的边 · <b>紫色虚线</b>=反向残量边 · <b>金色粗线</b>=增广路径 · <b>绿色粗线</b>=增广成功 · <b>节点背景</b>=Dinic 分层
      </span>
    </div>
  </div>

  <div class="demo-bar">
    <span class="title">🎬 演示</span>
    <span class="mode-label">模式：</span>
    <div class="radio-group" id="demoModeGroup">
      <label class="active"><input type="radio" name="demoMode" value="auto" checked>自动播放</label>
      <label><input type="radio" name="demoMode" value="step">逐步演示</label>
    </div>
    <button id="btnDemoStart" class="success">▶ 开始演示</button>
    <button id="btnDemoNext">⏭ 下一步</button>
    <button id="btnDemoStop" class="danger" disabled>⏹ 停止</button>
    <div class="speed-wrap">
      <span class="label">速度</span>
      <input type="range" id="speedRange" min="0.25" max="3" step="0.25" value="1">
      <span class="val" id="speedLabel">1.00x</span>
    </div>
  </div>

  <div class="result" id="result">就绪 · 选择算法后点击「开始演示」</div>

  <details class="log-module" id="logModule" open>
    <summary>
      📋 演示记录
      <span class="hint">（点击展开 / 折叠，可全量查看每步）</span>
      <span class="count" id="logCount">0 步</span>
    </summary>
    <div class="log-body" id="logBody">
      <div class="log-empty">点击「开始演示」后，每一步操作都会记录在这里</div>
    </div>
  </details>

  <div class="knowledge">
    <h2>📚 知识模块 · 网络流 <span class="hint">残量网络 + 增广 + 最小割</span></h2>
    <div class="k-grid" id="kGrid"></div>
  </div>

  <details class="code-module" open>
    <summary>
      🐍 Python 实现 · Edmonds-Karp + Dinic
      <span class="badge">邻接表 + 反向边</span>
    </summary>
    <div class="code-body">
<pre><code class="language-python">"""
网络流 —— 最大流 Edmonds-Karp / Dinic。

给定有向图 G = (V, E)，每条边 (u, v) 有非负容量 c。
源点 s，汇点 t，求从 s 到 t 能通过的最大流量。

核心概念：
    残量网络：正向边剩余 c-f，反向边剩余 f
    增广路径：残量网络中一条从 s 到 t 的路径
    瓶颈：路径上剩余容量的最小值

Ford-Fulkerson 方法 / Max-Flow Min-Cut 定理：
    f 是最大流 ⇔ 残量网络中不存在 s→t 增广路径
    最大流 = 最小割容量

Edmonds-Karp：每轮 BFS 找最短增广路径，O(VE²)
Dinic：每轮 BFS 分层 + DFS 阻塞流，O(V²E)
"""

from collections import deque


class MaxFlow:
    def __init__(self, n):
        self.n = n
        # 邻接表，graph[u] 中每条边是 [to, cap, rev_idx]
        # 正反边成对存储，rev_idx 互指
        self.graph = [[] for _ in range(n)]

    def add_edge(self, u, v, cap):
        # 正向边容量 cap，反向边容量 0
        self.graph[u].append([v, cap, len(self.graph[v])])
        self.graph[v].append([u, 0, len(self.graph[u]) - 1])

    # ---------------- Edmonds-Karp ----------------
    def max_flow_ek(self, s, t):
        flow = 0
        while True:
            # BFS 找增广路径
            parent = [-1] * self.n
            parent_edge = [-1] * self.n
            visited = [False] * self.n
            visited[s] = True
            q = deque([s])

            while q:
                u = q.popleft()
                if u == t:
                    break
                for i, (v, cap, _) in enumerate(self.graph[u]):
                    if not visited[v] and cap > 0:
                        visited[v] = True
                        parent[v] = u
                        parent_edge[v] = i
                        q.append(v)

            if not visited[t]:
                break

            # 找路径瓶颈
            d = float('inf')
            v = t
            while v != s:
                u = parent[v]
                d = min(d, self.graph[u][parent_edge[v]][1])
                v = u

            # 更新残量网络
            v = t
            while v != s:
                u = parent[v]
                e = self.graph[u][parent_edge[v]]
                e[1] -= d
                self.graph[v][e[2]][1] += d
                v = u

            flow += d

        return flow

    # ---------------- Dinic ----------------
    def max_flow_dinic(self, s, t):
        flow = 0
        while self._bfs(s, t):
            self.it = [0] * self.n
            while True:
                f = self._dfs(s, t, float('inf'))
                if f == 0:
                    break
                flow += f
        return flow

    def _bfs(self, s, t):
        # 在残量网络上分层
        self.level = [-1] * self.n
        self.level[s] = 0
        q = deque([s])
        while q:
            u = q.popleft()
            for v, cap, _ in self.graph[u]:
                if cap > 0 and self.level[v] < 0:
                    self.level[v] = self.level[u] + 1
                    q.append(v)
        return self.level[t] >= 0

    def _dfs(self, u, t, f):
        # 沿层次图找阻塞流，返回本次推的流量
        if u == t:
            return f
        for i in range(self.it[u], len(self.graph[u])):
            self.it[u] = i
            v, cap, rev = self.graph[u][i]
            if cap > 0 and self.level[v] == self.level[u] + 1:
                d = self._dfs(v, t, min(f, cap))
                if d > 0:
                    self.graph[u][i][1] -= d
                    self.graph[v][rev][1] += d
                    return d
        return 0


if __name__ == '__main__':
    # 教学图：6 节点 8 边
    # S→A(10), S→B(10)
    # A→C(4), A→D(8), B→D(9)
    # C→D(3), C→T(6), D→T(10)
    n = 6
    edges = [
        (0, 1, 10),  # S→A
        (0, 2, 10),  # S→B
        (1, 3, 4),   # A→C
        (1, 4, 8),   # A→D
        (2, 4, 9),   # B→D
        (3, 4, 3),   # C→D
        (3, 5, 6),   # C→T
        (4, 5, 10),  # D→T
    ]

    f1 = MaxFlow(n)
    for u, v, w in edges:
        f1.add_edge(u, v, w)
    print('Edmonds-Karp 最大流 =', f1.max_flow_ek(0, 5))

    f2 = MaxFlow(n)
    for u, v, w in edges:
        f2.add_edge(u, v, w)
    print('Dinic        最大流 =', f2.max_flow_dinic(0, 5))
</code></pre>
    </div>
  </details>
</div>

<script>
/* =========================================================
   1. 图结构（6 节点 8 边）
   ========================================================= */
const NODES = [
  { id: 'S', x: 120, y: 350 },
  { id: 'A', x: 320, y: 150 },
  { id: 'B', x: 320, y: 550 },
  { id: 'C', x: 580, y: 150 },
  { id: 'D', x: 580, y: 550 },
  { id: 'T', x: 780, y: 350 },
];
const NODE_POS = {};
for (const n of NODES) NODE_POS[n.id] = { x: n.x, y: n.y };

const EDGES = [
  { from: 'S', to: 'A', cap: 10 },
  { from: 'S', to: 'B', cap: 10 },
  { from: 'A', to: 'C', cap: 4 },
  { from: 'A', to: 'D', cap: 8 },
  { from: 'B', to: 'D', cap: 9 },
  { from: 'C', to: 'D', cap: 3 },
  { from: 'C', to: 'T', cap: 6 },
  { from: 'D', to: 'T', cap: 10 },
];
const SRC = 'S';
const SINK = 'T';

const NODE_R = 34;

/* 邻接（有向） */
const ADJ = {};
for (const n of NODES) ADJ[n.id] = [];
for (const e of EDGES) ADJ[e.from].push(e.to);

function eKey(u, v) { return `${u}>${v}`; }

/* 边的容量查询 */
const CAP = {};
for (const e of EDGES) CAP[eKey(e.from, e.to)] = e.cap;

/* =========================================================
   2. 渲染状态
   ========================================================= */
let flow = {};             /* key -> 当前流量 */
let edgeHl = {};           /* key -> '' | 'exploring' | 'path' | 'augmented' */
let nodeHl = {};           /* id -> '' | 'current' | 'in-path' */
let nodeLevel = {};        /* id -> level 数字（Dinic） */

let curMaxFlow = 0;
let curPathCount = 0;
let curRound = '';

/* =========================================================
   3. SVG 图层
   ========================================================= */
const NS = 'http://www.w3.org/2000/svg';
const svg = document.getElementById('stage');
const defs = document.createElementNS(NS, 'defs');
svg.appendChild(defs);

/* 箭头 marker */
function makeMarker(id, color) {
  const m = document.createElementNS(NS, 'marker');
  m.setAttribute('id', id);
  m.setAttribute('viewBox', '0 0 10 10');
  m.setAttribute('refX', '9'); m.setAttribute('refY', '5');
  m.setAttribute('markerWidth', '7'); m.setAttribute('markerHeight', '7');
  m.setAttribute('orient', 'auto');
  const p = document.createElementNS(NS, 'path');
  p.setAttribute('d', 'M0,0 L10,5 L0,10 z');
  p.setAttribute('fill', color);
  m.appendChild(p);
  defs.appendChild(m);
}
makeMarker('arrow-gray',  '#cbd2d9');
makeMarker('arrow-blue',  '#3b82c4');
makeMarker('arrow-gold',  '#e09b00');
makeMarker('arrow-green', '#2f7a34');
makeMarker('arrow-purple','#7c5cd6');

const edgeLayer = document.createElementNS(NS, 'g');
const nodeLayer = document.createElementNS(NS, 'g');
svg.appendChild(edgeLayer);
svg.appendChild(nodeLayer);

/* =========================================================
   4. 渲染
   ========================================================= */
function computeEnds(a, b, offset) {
  const dx = b.x - a.x, dy = b.y - a.y;
  const len = Math.hypot(dx, dy) || 1;
  const r = NODE_R;
  const ox = -dy / len * (offset || 0);
  const oy = dx / len * (offset || 0);
  return {
    x1: a.x + dx / len * r + ox,
    y1: a.y + dy / len * r + oy,
    x2: b.x - dx / len * r + ox,
    y2: b.y - dy / len * r + oy,
  };
}

function render() {
  edgeLayer.innerHTML = '';
  nodeLayer.innerHTML = '';

  /* 边 */
  for (const e of EDGES) {
    const a = NODE_POS[e.from];
    const b = NODE_POS[e.to];
    const key = eKey(e.from, e.to);
    const f = flow[key] || 0;

    /* 正向边（偏移 -1，反向边偏移 +1 以区分） */
    const st = edgeHl[key] || '';
    const baseCls = f > 0 ? ' has-flow' : '';
    let cls = 'nf-edge' + baseCls;
    if (st) cls = 'nf-edge ' + st;
    let marker = 'url(#arrow-gray)';
    if (f > 0) marker = 'url(#arrow-blue)';
    if (st === 'exploring') marker = 'url(#arrow-gold)';
    if (st === 'path')      marker = 'url(#arrow-gold)';
    if (st === 'augmented') marker = 'url(#arrow-green)';

    const { x1, y1, x2, y2 } = computeEnds(a, b, -3);
    const line = document.createElementNS(NS, 'line');
    line.setAttribute('x1', x1); line.setAttribute('y1', y1);
    line.setAttribute('x2', x2); line.setAttribute('y2', y2);
    line.setAttribute('class', cls);
    line.setAttribute('marker-end', marker);
    edgeLayer.appendChild(line);

    /* 反向边（仅在 flow > 0 时显示） */
    if (f > 0) {
      const { x1: rx1, y1: ry1, x2: rx2, y2: ry2 } = computeEnds(b, a, 4);
      const rline = document.createElementNS(NS, 'line');
      rline.setAttribute('x1', rx1); rline.setAttribute('y1', ry1);
      rline.setAttribute('x2', rx2); rline.setAttribute('y2', ry2);
      rline.setAttribute('class', 'nf-edge reverse');
      rline.setAttribute('marker-end', 'url(#arrow-purple)');
      edgeLayer.appendChild(rline);
    }

    /* 标签：flow / cap */
    const mx = (a.x + b.x) / 2;
    const my = (a.y + b.y) / 2;
    const dx = b.x - a.x, dy = b.y - a.y;
    const len = Math.hypot(dx, dy) || 1;
    const ox = -dy / len * 20;
    const oy = dx / len * 20;

    const labelCls = 'edge-w' + (st ? ' ' + st : (f > 0 ? ' has-flow' : ''));

    const t = document.createElementNS(NS, 'text');
    t.setAttribute('class', labelCls);
    t.setAttribute('text-anchor', 'middle');
    t.setAttribute('dominant-baseline', 'central');
    t.setAttribute('x', mx + ox);
    t.setAttribute('y', my + oy);
    t.textContent = `${f}/${e.cap}`;
    edgeLayer.appendChild(t);
  }

  /* 节点 */
  for (const n of NODES) {
    const g = document.createElementNS(NS, 'g');
    let cls = 'nf-node';
    if (n.id === SRC) cls += ' source';
    else if (n.id === SINK) cls += ' sink';
    if (nodeHl[n.id]) cls += ' ' + nodeHl[n.id];
    g.setAttribute('class', cls);

    const c = document.createElementNS(NS, 'circle');
    c.setAttribute('cx', n.x); c.setAttribute('cy', n.y);
    c.setAttribute('r', NODE_R);
    c.setAttribute('class', 'node-bg');
    g.appendChild(c);

    const t = document.createElementNS(NS, 'text');
    t.setAttribute('class', 'node-id');
    t.setAttribute('text-anchor', 'middle');
    t.setAttribute('dominant-baseline', 'central');
    t.setAttribute('x', n.x); t.setAttribute('y', n.y);
    t.textContent = n.id;
    g.appendChild(t);

    /* Dinic 分层标记 */
    if (nodeLevel[n.id] !== undefined && nodeLevel[n.id] >= 0) {
      const lbBg = document.createElementNS(NS, 'rect');
      lbBg.setAttribute('class', 'level-bg');
      lbBg.setAttribute('x', n.x - 14);
      lbBg.setAttribute('y', n.y - NODE_R - 20);
      lbBg.setAttribute('width', 28);
      lbBg.setAttribute('height', 16);
      lbBg.setAttribute('rx', 4);
      g.appendChild(lbBg);

      const lbT = document.createElementNS(NS, 'text');
      lbT.setAttribute('class', 'node-level');
      lbT.setAttribute('text-anchor', 'middle');
      lbT.setAttribute('dominant-baseline', 'central');
      lbT.setAttribute('x', n.x);
      lbT.setAttribute('y', n.y - NODE_R - 12);
      lbT.textContent = 'L' + nodeLevel[n.id];
      g.appendChild(lbT);
    }

    nodeLayer.appendChild(g);
  }

  updateInfoPanel();
}

function updateInfoPanel() {
  document.getElementById('totalFlow').textContent = `${curMaxFlow} / 14`;
  document.getElementById('pathCount').textContent = curPathCount;
  document.getElementById('roundInfo').textContent = curRound || '—';
}

/* =========================================================
   5. 算法辅助（用于生成步骤）
   ========================================================= */
function cloneFlow(f) { return JSON.parse(JSON.stringify(f)); }

/* 残量容量 */
function residual(key, f) {
  const [u, v] = key.split('>');
  const fwdKey = `${u}>${v}`;
  const revKey = `${v}>${u}`;
  if (CAP[fwdKey] !== undefined) {
    return CAP[fwdKey] - (f[fwdKey] || 0);
  } else if (CAP[revKey] !== undefined) {
    return (f[revKey] || 0);
  }
  return 0;
}

/* 施加增广 */
function augmentFlow(f, path, amount) {
  for (const key of path) {
    const [u, v] = key.split('>');
    const fwdKey = `${u}>${v}`;
    const revKey = `${v}>${u}`;
    if (CAP[fwdKey] !== undefined) {
      f[fwdKey] = (f[fwdKey] || 0) + amount;
    } else if (CAP[revKey] !== undefined) {
      f[revKey] = (f[revKey] || 0) - amount;
    }
  }
}

/* =========================================================
   6. Edmonds-Karp 步骤生成
   ========================================================= */
function buildEKSteps() {
  const steps = [];
  const f = {};
  for (const e of EDGES) f[eKey(e.from, e.to)] = 0;

  let totalFlow = 0;
  let pathCount = 0;

  function snap(msg, opts) {
    const o = opts || {};
    steps.push({
      msg,
      flow: cloneFlow(f),
      edgeHl: { ...(o.edgeHl || {}) },
      nodeHl: { ...(o.nodeHl || {}) },
      totalFlow: o.totalFlow !== undefined ? o.totalFlow : totalFlow,
      pathCount: o.pathCount !== undefined ? o.pathCount : pathCount,
      round: '',
    });
  }

  snap('Edmonds-Karp 开始：所有边流量 = 0，不断用 BFS 找增广路径', {});

  while (true) {
    /* BFS 在残量网络上找增广路径 */
    const parent = {};
    const visited = { [SRC]: true };
    const q = [SRC];
    while (q.length > 0) {
      const u = q.shift();
      if (u === SINK) break;
      for (const v of ADJ[u]) {
        if (visited[v]) continue;
        if (residual(eKey(u, v), f) > 0) {
          visited[v] = true;
          parent[v] = u;
          q.push(v);
        }
      }
    }

    if (!visited[SINK]) {
      snap(`残量网络中不存在 S→T 路径 → 算法结束`, {});
      break;
    }

    /* 回溯路径 */
    const path = [];
    let cur = SINK;
    while (cur !== SRC) {
      const p = parent[cur];
      path.unshift(eKey(p, cur));
      cur = p;
    }

    /* 找瓶颈 */
    let bottleneck = Infinity;
    for (const key of path) {
      bottleneck = Math.min(bottleneck, residual(key, f));
    }

    snap(
      `BFS 找到增广路径：${SRC}→${path.map(k => k.split('>')[1]).join('→')}，瓶颈 = ${bottleneck}`,
      {
        edgeHl: Object.fromEntries(path.map(k => [k, 'path'])),
      }
    );

    /* 沿路径推流 */
    augmentFlow(f, path, bottleneck);
    totalFlow += bottleneck;
    pathCount += 1;

    snap(
      `✓ 沿路径推流 ${bottleneck}，当前总流量 = ${totalFlow}`,
      {
        edgeHl: Object.fromEntries(path.map(k => [k, 'augmented'])),
        totalFlow,
        pathCount,
      }
    );
  }

  snap(`🎉 Edmonds-Karp 完成！最大流 = ${totalFlow}`, {
    totalFlow,
    pathCount,
  });

  return steps;
}

/* =========================================================
   7. Dinic 步骤生成
   ========================================================= */
function buildDinicSteps() {
  const steps = [];
  const f = {};
  for (const e of EDGES) f[eKey(e.from, e.to)] = 0;

  let totalFlow = 0;
  let pathCount = 0;
  let round = 0;

  function snap(msg, opts) {
    const o = opts || {};
    steps.push({
      msg,
      flow: cloneFlow(f),
      edgeHl: { ...(o.edgeHl || {}) },
      nodeHl: { ...(o.nodeHl || {}) },
      nodeLevel: o.nodeLevel ? { ...o.nodeLevel } : (o.clearLevel ? {} : undefined),
      totalFlow: o.totalFlow !== undefined ? o.totalFlow : totalFlow,
      pathCount: o.pathCount !== undefined ? o.pathCount : pathCount,
      round: `第 ${round} 轮`,
    });
  }

  snap('Dinic 开始：每轮 BFS 分层，然后在层次图上找阻塞流', { clearLevel: true });

  while (true) {
    round += 1;
    /* BFS 分层 */
    const level = { [SRC]: 0 };
    const q = [SRC];
    while (q.length > 0) {
      const u = q.shift();
      for (const v of ADJ[u]) {
        if (level[v] === undefined && residual(eKey(u, v), f) > 0) {
          level[v] = level[u] + 1;
          q.push(v);
        }
      }
    }

    if (level[SINK] === undefined) {
      snap(`第 ${round} 轮：BFS 无法到达 T → 算法结束`, { clearLevel: true });
      break;
    }

    snap(
      `第 ${round} 轮 BFS 分层：${Object.entries(level).map(([k, v]) => `${k}:L${v}`).join(', ')}`,
      { nodeLevel: level }
    );

    /* 在层次图上找阻塞流：反复 DFS 找增广路径 */
    while (true) {
      /* 单次 DFS 找一条层次图路径 */
      const path = [];
      const parentMap = {};
      const visited = { [SRC]: true };

      /* 用栈模拟 DFS，但需要回溯；用递归生成路径 */
      function dfs(u) {
        if (u === SINK) return true;
        for (const v of ADJ[u]) {
          if (visited[v]) continue;
          if (level[v] !== level[u] + 1) continue;
          if (residual(eKey(u, v), f) <= 0) continue;
          visited[v] = true;
          path.push(eKey(u, v));
          if (dfs(v)) return true;
          path.pop();
        }
        return false;
      }

      if (!dfs(SRC)) break;

      /* 找瓶颈 */
      let bottleneck = Infinity;
      for (const key of path) {
        bottleneck = Math.min(bottleneck, residual(key, f));
      }

      /* 高亮正在探索的层次图路径 */
      snap(
        `  层次图上找到路径：${SRC}→${path.map(k => k.split('>')[1]).join('→')}，瓶颈 = ${bottleneck}`,
        {
          edgeHl: Object.fromEntries(path.map(k => [k, 'path'])),
          nodeLevel: level,
        }
      );

      augmentFlow(f, path, bottleneck);
      totalFlow += bottleneck;
      pathCount += 1;

      snap(
        `  ✓ 推流 ${bottleneck}，累计最大流 = ${totalFlow}`,
        {
          edgeHl: Object.fromEntries(path.map(k => [k, 'augmented'])),
          nodeLevel: level,
          totalFlow,
          pathCount,
        }
      );
    }

    snap(`第 ${round} 轮阻塞流结束，本轮最大可能流量已榨干`, { nodeLevel: level });
  }

  snap(`🎉 Dinic 完成！最大流 = ${totalFlow}`, {
    totalFlow,
    pathCount,
    clearLevel: true,
  });

  return steps;
}

/* =========================================================
   8. 步骤应用
   ========================================================= */
function applyStep(step) {
  flow = cloneFlow(step.flow);
  edgeHl = { ...step.edgeHl };
  nodeHl = { ...(step.nodeHl || {}) };

  /* 节点高亮：在增广路径上的节点 */
  for (const k of Object.keys(edgeHl)) {
    if (edgeHl[k] === 'path') {
      const [u, v] = k.split('>');
      if (!nodeHl[u]) nodeHl[u] = 'in-path';
      if (!nodeHl[v]) nodeHl[v] = 'in-path';
    }
  }

  /* 分层 */
  if (step.nodeLevel !== undefined) {
    nodeLevel = { ...step.nodeLevel };
  } else if (step.clearLevel) {
    nodeLevel = {};
  }

  curMaxFlow = step.totalFlow;
  curPathCount = step.pathCount;
  curRound = step.round || '';

  render();
}

/* =========================================================
   9. 演示记录
   ========================================================= */
const logBody = document.getElementById('logBody');
const logCount = document.getElementById('logCount');
const resultEl = document.getElementById('result');
function setStatus(t) { resultEl.textContent = t; }

let logLines = [];
function clearLog() {
  logLines = [];
  logBody.innerHTML = '<div class="log-empty">点击「开始演示」后，每一步操作都会记录在这里</div>';
  logCount.textContent = '0 步';
}
function addLog(msg) {
  logLines.push(msg);
  logCount.textContent = logLines.length + ' 步';
  if (logLines.length === 1) logBody.innerHTML = '';
  const div = document.createElement('div');
  div.className = 'log-line';
  div.dataset.idx = logLines.length - 1;
  div.innerHTML = `<div class="idx">#${logLines.length}</div><div class="msg">${escapeHtml(msg)}</div>`;
  logBody.appendChild(div);
}
function highlightLog(idx) {
  document.querySelectorAll('.log-line.cur').forEach(el => el.classList.remove('cur'));
  const el = document.querySelector(`.log-line[data-idx="${idx}"]`);
  if (el) {
    el.classList.add('cur');
    el.scrollIntoView({ block: 'nearest', behavior: 'smooth' });
  }
}
function escapeHtml(s) {
  return String(s).replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
}

/* =========================================================
   10. 演示控制器
   ========================================================= */
let demoSpeed = 1;
let demoPaused = false;
let demoAborted = false;
let demoStepMode = false;
let stepResolver = null;

async function tick(ms = 0) {
  if (demoAborted) throw new Error('aborted');
  if (demoStepMode) {
    await new Promise(resolve => { stepResolver = resolve; });
    if (demoAborted) throw new Error('aborted');
  }
  while (demoPaused && !demoAborted) {
    await new Promise(r => setTimeout(r, 60));
  }
  if (demoAborted) throw new Error('aborted');
  if (ms > 0) await new Promise(r => setTimeout(r, ms / demoSpeed));
  if (demoAborted) throw new Error('aborted');
}
function releaseStep() {
  if (stepResolver) {
    const r = stepResolver;
    stepResolver = null;
    r();
  }
}

/* =========================================================
   11. 主流程
   ========================================================= */
async function runDemo(algo) {
  /* 重置状态 */
  flow = {};
  for (const e of EDGES) flow[eKey(e.from, e.to)] = 0;
  edgeHl = {};
  nodeHl = {};
  nodeLevel = {};
  curMaxFlow = 0;
  curPathCount = 0;
  curRound = '';
  render();
  clearLog();
  logLines = [];

  let steps, algoName;
  if (algo === 'ek') { steps = buildEKSteps(); algoName = 'Edmonds-Karp'; }
  else { steps = buildDinicSteps(); algoName = 'Dinic'; }

  setStatus(`准备 ${algoName} 演示，共 ${steps.length} 步`);
  await tick(500);

  for (let i = 0; i < steps.length; i++) {
    const s = steps[i];
    applyStep(s);
    setStatus(`[${i+1}/${steps.length}] ${s.msg}`);
    addLog(s.msg);
    highlightLog(i);
    await tick(900);
  }

  setStatus(`✔ ${algoName} 演示结束，最大流 = ${curMaxFlow}`);
  await tick(700);
}

/* =========================================================
   12. 演示 UI 控制
   ========================================================= */
const btnDemoStart = document.getElementById('btnDemoStart');
const btnDemoNext  = document.getElementById('btnDemoNext');
const btnDemoStop  = document.getElementById('btnDemoStop');
const speedRange   = document.getElementById('speedRange');
const speedLabel   = document.getElementById('speedLabel');
const algoSel      = document.getElementById('algoSel');

let demoRunning = false;

function currentDemoMode() {
  return document.querySelector('input[name="demoMode"]:checked').value;
}
function currentAlgo() { return algoSel.value; }

function updateDemoUI() {
  if (!demoRunning) {
    btnDemoStart.textContent = '▶ 开始演示';
    btnDemoStart.disabled = false;
    btnDemoNext.disabled = false;
    btnDemoStop.disabled = true;
    algoSel.disabled = false;
  } else if (demoStepMode) {
    btnDemoStart.textContent = '▶ 切到自动';
    btnDemoStart.disabled = false;
    btnDemoNext.disabled = false;
    btnDemoStop.disabled = false;
    algoSel.disabled = true;
  } else if (demoPaused) {
    btnDemoStart.textContent = '▶ 继续';
    btnDemoStart.disabled = false;
    btnDemoNext.disabled = false;
    btnDemoStop.disabled = false;
    algoSel.disabled = true;
  } else {
    btnDemoStart.textContent = '⏸ 暂停';
    btnDemoStart.disabled = false;
    btnDemoNext.disabled = false;
    btnDemoStop.disabled = false;
    algoSel.disabled = true;
  }
}

async function startDemo(mode) {
  if (demoRunning) return;
  demoRunning = true;
  demoAborted = false;
  demoPaused = false;
  demoStepMode = (mode === 'step');
  updateDemoUI();

  if (mode === 'step') {
    setStatus('⏭ 逐步演示已就绪：每次点「下一步」推进一个动作');
  }

  try {
    await runDemo(currentAlgo());
  } catch (e) {
    if (e.message === 'aborted') setStatus('⏹ 演示已停止');
    else console.error(e);
  }

  demoRunning = false;
  demoAborted = false;
  demoPaused = false;
  demoStepMode = false;
  updateDemoUI();
}

btnDemoStart.addEventListener('click', () => {
  if (!demoRunning) { startDemo(currentDemoMode()); return; }
  if (demoStepMode) {
    demoStepMode = false; demoPaused = false; releaseStep();
  } else if (demoPaused) {
    demoPaused = false;
  } else {
    demoPaused = true;
  }
  updateDemoUI();
});

btnDemoNext.addEventListener('click', () => {
  if (!demoRunning) {
    startDemo('step');
    setTimeout(() => releaseStep(), 30);
    return;
  }
  if (!demoStepMode) demoStepMode = true;
  demoPaused = false;
  releaseStep();
  updateDemoUI();
});

btnDemoStop.addEventListener('click', () => {
  if (!demoRunning) return;
  demoAborted = true; demoPaused = false; demoStepMode = false; releaseStep();
});

speedRange.addEventListener('input', () => {
  demoSpeed = parseFloat(speedRange.value);
  speedLabel.textContent = demoSpeed.toFixed(2) + 'x';
});

document.querySelectorAll('input[name="demoMode"]').forEach(r => {
  r.addEventListener('change', () => {
    document.querySelectorAll('#demoModeGroup label').forEach(l => l.classList.remove('active'));
    r.parentElement.classList.add('active');
    if (!demoRunning) {
      const m = currentDemoMode();
      setStatus(m === 'auto'
        ? '已选择「自动播放」：点击「开始演示」后连续播放'
        : '已选择「逐步演示」：点击「开始演示」后每次点「下一步」推进');
    }
  });
});

algoSel.addEventListener('change', () => {
  if (demoRunning) return;
  flow = {};
  for (const e of EDGES) flow[eKey(e.from, e.to)] = 0;
  edgeHl = {};
  nodeHl = {};
  nodeLevel = {};
  curMaxFlow = 0;
  curPathCount = 0;
  curRound = '';
  render();
  clearLog();
  const name = algoSel.options[algoSel.selectedIndex].text.split(' ')[0];
  setStatus(`已切换到 ${name}，点击「开始演示」`);
});

/* =========================================================
   13. 知识模块
   ========================================================= */
const KNOWLEDGE = [
  { icon: '🎯', title: '网络流问题', key: true, lines: [
    '有向图 G = (V, E)，每条边有容量 c(u,v) ≥ 0',
    '源点 s、汇点 t，求从 s 到 t 能通过的最大流量',
    '约束：每条边的流量 f(u,v) ≤ c(u,v)',
    '<b>守恒</b>：除 s、t 外，每个点入流 = 出流',
    '实际应用：物流、通信带宽、任务调度',
  ]},
  { icon: '🔑', title: '残量网络：核心数据结构', key: true, lines: [
    '每条边 (u, v, c) 对应两条残量边：',
    '　· 正向：<code>c − f</code>（还能再推）',
    '　· 反向：<code>f</code>（可以回退）',
    '图上的 <code>flow/cap</code> 就是当前流量状态',
    '<b>反向边</b>是网络流算法的关键——它让"撤回流"成为可能',
  ]},
  { icon: '🛤️', title: '增广路径与瓶颈', key: true, lines: [
    '残量网络中一条从 s 到 t 的路径',
    '<b>瓶颈</b> = 路径上所有边剩余容量的最小值',
    '沿路径推流瓶颈 → 流量 +瓶颈',
    '同时反向边 +瓶颈（表示这些流量可以回退）',
  ]},
  { icon: '⚡', title: 'Edmonds-Karp', key: true, lines: [
    '每轮用 <b>BFS</b> 找一条<b>最短</b>增广路径',
    'BFS 保证找的路径边数最少 → 避免指数时间',
    '复杂度 <code>O(VE²)</code>',
    '简单可靠，是入门首选',
    '类比：二分图匹配的匈牙利算法也是不断找增广路径',
  ]},
  { icon: '🌊', title: 'Dinic：分层 + 阻塞流', key: true, lines: [
    '<b>第 1 步</b>：BFS 从 s 出发分层，每个点标记"距 s 的最短边数"',
    '<b>第 2 步</b>：只沿"层数 +1"的边 DFS，找多条增广路径',
    '当无法再找时，这轮的"阻塞流"榨干，重新 BFS',
    '复杂度 <code>O(V²E)</code>，二分图上 <code>O(E√V)</code>',
    '现代最常用的最大流算法',
  ]},
  { icon: '🎯', title: 'Max-Flow Min-Cut 定理', key: true, lines: [
    '<b>割</b>：把 V 分成 S 侧（含 s）和 T 侧（含 t）',
    '<b>割容量</b> = 从 S 侧指向 T 侧的边容量和',
    '<b>最大流 = 最小割</b>（Ford-Fulkerson 定理）',
    '这是网络流最深刻的理论结果',
    '应用：最小割模型常用来解二元决策问题',
  ]},
  { icon: '🎨', title: '不同场景需要不同的图', key: true, lines: [
    '<b>基础最大流</b>：本页面 6 节点图，展示残量网络与反向边',
    '<b>二分图匹配</b>：S → 左部 → 右部 → T，容量都是 1',
    '<b>最小割问题</b>：需构建能体现 max-flow = min-cut 的图',
    '<b>费用流（MCMF）</b>：每条边带费用，选"费用最小的最大流"',
    '<b>多源多汇</b>：加超级源点 / 汇点连接所有源 / 汇',
    '<b>带上下界流</b>：每边有下界 L 和上界 U，需转化',
  ]},
  { icon: '🧩', title: '典型应用', key: false, lines: [
    '<b>二分图匹配</b>：选课、任务分配、棋盘覆盖',
    '<b>最小路径覆盖</b>：DAG 覆盖 = V − 最大匹配',
    '<b>图像分割</b>：Graph Cut，用于前景背景分割',
    '<b>项目选择</b>：最大权闭合子图',
    '<b>网络流量规划</b>：带宽分配、管道流量',
    '<b>运输问题</b>：多源多汇的货物调配',
  ]},
];

function renderKnowledge() {
  const grid = document.getElementById('kGrid');
  grid.innerHTML = '';
  for (const card of KNOWLEDGE) {
    const el = document.createElement('div');
    let cls = 'k-card';
    if (card.key) cls += ' key-point';
    if (card.title === '不同场景需要不同的图') cls = 'k-card compare-point';
    el.className = cls;
    el.innerHTML = `
      <div class="k-head">
        <div class="k-icon">${card.icon}</div>
        <div class="k-title">${card.title}</div>
      </div>
      <ul class="k-lines">
        ${card.lines.map(l => `<li>${l}</li>`).join('')}
      </ul>
    `;
    grid.appendChild(el);
  }
}

/* =========================================================
   14. 初始化
   ========================================================= */
renderKnowledge();
if (window.hljs) hljs.highlightAll();

/* 初始渲染 */
flow = {};
for (const e of EDGES) flow[eKey(e.from, e.to)] = 0;
edgeHl = {};
nodeHl = {};
nodeLevel = {};
curMaxFlow = 0;
curPathCount = 0;
curRound = '';
render();
clearLog();
updateDemoUI();
</script>
</body>
</html>
```

### 页面设计要点

**1. 与 MST / 最短路页面统一的骨架**

原理简介（总起 + 6 张卡片）、SVG 画布、信息面板、算法选择面板、演示控制条、结果状态、演示记录（可折叠）、知识模块（8 张卡片）、Python 代码（highlight.js）。

**2. 网络流特有的可视化**

| 元素 | 视觉 | 含义 |
|---|---|---|
| **节点 S** | 红边 | 源点 |
| **节点 T** | 绿边 | 汇点 |
| **正向边（无流）** | 灰色细线 | 未使用的边 |
| **正向边（有流）** | 蓝色粗线 + 箭头 | 承载流量的边 |
| **反向残量边** | 紫色虚线 + 箭头 | flow > 0 时出现，表示"可回退" |
| **增广路径** | 金色流动虚线 | 当前正在推流的路径 |
| **增广完成** | 绿色粗线 | 已成功增广 |
| **边上标签** | `flow/cap` | 当前流量 / 容量 |
| **节点分层标记** | 紫色 L0/L1/L2 | Dinic 每轮的层次号 |

**3. 教学图**

6 节点 8 条有向边：

```
        A ──4──→ C
       /│        │ \
     10 │        │  6
     /  │       3│   \
    S   │8       ↓    T
     \  ↓        D   ↑
     10 B ──9──→ ↑   │
       \         │  10
        \────────┘
```

- S→A(10)、S→B(10)
- A→C(4)、A→D(8)、B→D(9)
- C→D(3)、C→T(6)、D→T(10)

最大流 = 14，EK 需要 3 次增广就能完成。

**4. 两种算法的动画差异**

| 算法 | 视觉重点 |
|---|---|
| **Edmonds-Karp** | 每轮 BFS 找**一条**最短增广路径，立即推流 |
| **Dinic** | 先 BFS 分层（节点上显示 L0/L1/...），然后在层次图上**连续**找多条增广路径，本轮榨干后重新分层 |

**5. 演示记录**

可折叠、当前步高亮、自动滚动。每一步的详细描述都能在日志里回看。

**6. 关于"不同场景用不同图"**

知识模块里专门有一张卡片讨论：

- **基础最大流** → 本页面 6 节点图（展示残量网络）
- **二分图匹配** → S → 左部 → 右部 → T，容量都是 1
- **最小割问题** → 需要让 max-flow = min-cut 的经典图
- **费用流** → 每条边带费用，需要展示"费用优先"
- **多源多汇** → 加超级源点 / 汇点
- **带上下界的流** → 每边有下界 L 和上界 U，需要转化

### Python 代码

用 `MaxFlow` 类封装，邻接表存正反边（`[to, cap, rev_idx]` 三元组），两个算法共用数据结构：

- `max_flow_ek(s, t)` —— Edmonds-Karp
- `max_flow_dinic(s, t)` + `_bfs` + `_dfs` —— Dinic

代码底部用页面同样的图做完整验证，两种算法都输出 14。