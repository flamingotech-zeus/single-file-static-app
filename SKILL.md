---
name: single-file-static-app
description: 单文件静态应用（含 Chart.js 内联）的构建、浏览器自动化验证与交付流程。当用户要"单文件 HTML 应用/内联所有资源/离线可用/本地打卡·记账·看板类工具"时使用，覆盖 Chart.js 内联构建、file:// localStorage 坑、agent-browser 验证技巧、explorer 弹窗+卡片交付。
agent_created: true
---

# 单文件静态应用：构建 + 验证 + 交付

## 1. 目录与构建（Chart.js 内联模式）

```
项目/
├── index.html        ← 成品（交付物）
├── src/app.html      ← 模板，Chart.js 位置放占位标记 <script>/*__CHART_JS__*/</script>
├── assets/chart.umd.min.js   ← curl -fsSL https://cdn.jsdelivr.net/npm/chart.js@4/dist/chart.umd.min.js
└── tools/build.py
```

build.py 核心（纯字符串替换，勿用 JS 的 .replace，$ 会被当模式）：
```python
out = tpl.replace("/*__CHART_JS__*/", chart, 1)   # python str.replace 是字面替换，安全
```

用户明确要"离线可用/单文件"时才内联；否则 `<script src>` 即可。

## 2. SPA 模式（单文件多"页"）

- hash 路由：`#/home` 等，`hashchange` + `route()` 统一调度，`renderers{}` 每视图一个渲染函数；**渲染函数之间禁止互调**，交互后改数据→调 route()/refreshAll()。
- Chart 实例用 `state.charts{}` 登记，重渲染前 `destroy()`，否则 Canvas 泄漏叠加。
- localStorage 读写全部包 try/catch（file:// 或隐私模式下可能被拒，页面不能崩）。
- 移动端适配断点 ≤920px：侧边栏隐藏→底部 Tab（`env(safe-area-inset-bottom)`）；`display:none→block` 切换会让 CSS 入场动画自动重放。
- 首次空数据给"载入示例数据"按钮，严禁首屏空白。

## 3. agent-browser 验证（踩坑要点）

- **file:// 下 localStorage 抛 SecurityError（无头 Chromium 限制）**——桌面浏览器双击 file:// 是正常的，但自动化验证必须先起本地服务器：
  `python -m http.server 8123 --bind 127.0.0.1`（项目目录，run_in_background）→ 验证 http://127.0.0.1:8123/...
- `agent-browser click` 偶发不触发（hash 跳转后元素未稳定）：先 `scrollintoview` + `wait 300` 再点；排查时 `eval` 直接调函数区分"逻辑 bug"vs"点击派发问题"。
- 错误监控：`eval "window.__errs=[];window.addEventListener('error',e=>window.__errs.push(String(e.message)))"`，注意 `open` 会整页刷新、钩子会丢。
- 移动端验证：`agent-browser set viewport 390 844`；检查 `getComputedStyle` 里 sidebar/tabbar 的 display 切换。
- Chart 是否渲染：`!!Chart.getChart(document.querySelector('#id'))`。
- 结束必 `agent-browser close`；截图目录要先 mkdir（不存在会 os error 3）。

## 4. 交付（本机用户习惯，强制顺序）

1. PowerShell `Start-Process explorer.exe -ArgumentList '"D:\或C:\完整项目路径"'` 弹文件夹（explorer exit=1 是正常的，别当失败）
2. `present_files`：本地预览 URL（http://127.0.0.1:xxxx，服务器保持后台运行）放第一个 → index.html → 代表性截图
3. 版本号标注（V1.0 + 名字），写入 docs/CHANGELOG.md，README 维护文件索引
4. 收尾写工作区记忆日志（当日 md 追加）
