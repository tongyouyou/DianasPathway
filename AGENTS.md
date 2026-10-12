# Diana's Pathway —— 维护约定

纯静态单页应用，无构建、无后端。用户数据存在浏览器 localStorage，键名 `wn-word-coach-v4`。
用户不懂技术，全程用中文沟通；一步一步改，每次只改她指定的地方，改完要验证并说明。

## 文件
- `index.html` / `styles.css` / `app.js` —— 应用本体
- `parent.html` / `parent.css` / `parent.js` —— 家长端（移动端单页），与本体**共用** localStorage 数据；登录用「学生姓名或手机号 + 密码（默认 123456）」，会话存在 `wn-parent-student`
- `wechat-miniprogram/` —— 家长端微信小程序工程（可直接用微信开发者工具导入）。数据是 `utils/data.real.js` 静态快照；`utils/data.js` 是真实/演示数据开关；数据访问全在 `utils/store.js`
- `deploy/` —— 多人共用的云端部署包。`server.js`（零依赖 Node 后台）或 `worker.js`（Cloudflare 免费版），数据放 `deploy/data/state.json`。**改完网页必须跑 `deploy/sync.sh`**（它会同步 `public/` 并把 `app.js` 的 `CLOUD_MODE` 改成 true）。接口：`POST /api/login`、`GET/PUT /api/state`（带 rev 防覆盖）
- `data-seed.js` —— 内置初始数据（`window.__SEED__`），只在该浏览器本地没有数据时用于首次初始化
- `数据备份-*.json` —— 数据快照（`旧数据备份-…（清空前）` 是历史存档）

## 改完功能后必做（用户明确要求）
每次改完功能，都要把 `data-seed.js` 同步成用户当时的最新数据，
这样她换电脑 / 换浏览器登录后看到的也是最新数据，不用手动备份恢复。

1. 取最新快照（`数据备份-*.json`，或应用「用户信息 → 备份数据」导出的内容）
2. 生成 `data-seed.js`（去掉登录态与界面状态）：

   python3 片段：
   d = json.load(open('数据备份-xxx.json', encoding='utf-8'))
   d.pop('user', None)              # 不能带登录态，首次打开要登录
   d.pop('sidebarCollapsed', None)
   s = json.dumps(d, ensure_ascii=False).replace('</', '<\\/')
   写入内容 = "/* Diana's Pathway —— 初始数据（首次打开自动载入，不含登录态） */\nwindow.__SEED__ = " + s + ";\n"

3. 校验：`node --check data-seed.js`，并核对 students / courses 条数与实际一致
4. 确认 `data-seed.js` 去掉 user 后与最新备份一致

## 红线（会导致用户数据丢失或登录不上）
- 不改 localStorage 键名 `wn-word-coach-v4`
- 不改 S 的字段结构：students / classes / records / wrong / progress / courses / coaches / user
- defaultState() 必须支持 window.__SEED__，含 students 为空数组的空数据也要走种子分支
- __SEED__ 里不能带 user，否则首次打开不会要求登录
- data-seed.js 只在本地没有数据时生效，绝不覆盖已有数据
- 登录账号来自 S.coaches（当前 Diana / 123456）；清数据时必须保留 coaches 和 user

## 本机调试备忘
- 预览：项目目录起 `python3 -m http.server 8777`，访问 http://127.0.0.1:8777/
- 浏览器自动化用 cua_repl；绑定标签页优先用 agent.browsers.get("2").tabs.get("2")，
  cua.getTab 会自动打印无障碍树，遇到长文本弹窗会超限报错
- 不要经 cua_repl 传大于约 200KB 的字符串（会被截断）；读写 localStorage 请用同源临时页面或分块
- 临时页面用完删掉，临时标签记得关掉
