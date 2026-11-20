# 实验三报告：QQ邮箱采集（Playwright版）

- 班级：数据1241
- 姓名：李锐
- 学号：112404260119
- 第1—2课时实验日期：2026-11-20【请按实际上课日期核对】
- 第3—4课时实验日期：2026-11-20【请按实际上课日期核对】
- 第5—6课时实验日期：2026-11-20【请按实际上课日期核对】
- 第7—8课时实验日期：2026-11-20【请按实际上课日期核对】
- 所用工具：Playwright
- 工具版本与运行环境：Playwright 1.62.0、pandas 3.0.5；课程 Docker 内 JupyterLab；L07/L08 可视化增量包（`headless=False`），`lab03/` 与 `L07_L08_visual/` 同级
- 实验Notebook文件名：`qqmail_lab03_playwright.ipynb`
- GitHub原作业仓库链接：https://github.com/lirui-qwe/bigdata-collection-labs-112404260119
- 数据来源（教师练习邮件 / 自备无敏感练习邮件 / 其他，说明）：教师练习邮件【请按实际来源修改】

> 四个阶段写在同一份报告中，最后统一提交一次。截图放在 `images/`，关键代码显示文件名、行号或单元位置，用红框标出，每项附2—3句话解释。只填写真实结果；未完成写明原因。截图、CSV和Notebook输出提交前脱敏，不展示登录凭据。

对应[学生指导书](实验指导书-Playwright版.md)。单元1—2为第一阶段，3—6为第二阶段，7—9为第三阶段，10—12为第四阶段；单元13为选做。

## 第一阶段：扫码登录（第1—2课时）

### 1. 环境和运行方式

- 使用的启动脚本：`start-course-visual.bat`（先启动 Docker 与课程工作区，再启动可视化显示服务）
- 运行位置：Docker内JupyterLab的Notebook代码单元
- 自己新建的Notebook文件名：`qqmail_lab03_playwright.ipynb`
- 所选工具版本：Playwright 1.62.0（异步 API，`async_playwright()`）
- 启动或依赖问题及处理：初次打开可视化时需确认 `L07_L08_visual` 已安装到 `workspace/` 且与 `lab03/` 同级；`course_visual.prepare()` 负责把显示服务配置写入当前内核。未出现问题时写"无"。【请按实际处理情况修改】

**截图：** 环境成功运行和工具版本，注意遮挡可视化连接中的认证信息。

【待实测截图：`images/01_env_version.png` — 显示 JupyterLab 中本 Notebook、单元1 输出"浏览器与页面已就绪"，以及 `playwright.__version__`、`pd.__version__` 输出；连接地址中的 token 需打码】

### 2. 扫码与登录判断

- 人工完成的操作：在可视化浏览器窗口中用手机 QQ 扫描登录二维码，并在手机上确认登录。
- 登录成功控件的定位方式和选择器：`page.locator(".sidebar").get_by_text("收件箱", exact=True)`，即先框定左侧导航区域，再用完整文字精确定位"收件箱"。【选择器请按自己页面在开发者工具中核对后填写】
- 等待条件与最长等待时间：`await inbox.wait_for(timeout=600000)`，即等待元素出现，最长 600000 毫秒（10 分钟），出现后立即继续。
- 本次实际结果：【待实测：输出"登录成功""已进入收件箱"，并进入收件箱列表】

**截图：** 登录判断代码和真实成功输出；若失败，附实际报错。

【待实测截图：`images/02_login_success.png` — 框出 `goto()`、`wait_for(timeout=600000)`、`click()` 三行与成功输出；账号、头像、昵称打码】

说明：程序如何区分“页面已打开”和“已经登录”？

答：`page.goto()` 只保证浏览器拿到了页面文档，返回时页面仍在二维码阶段，不代表已登录。程序用"收件箱控件出现"作为登录完成的信号：只有扫码并确认后，左侧导航才会渲染出"收件箱"，`locator.wait_for()` 才结束等待。这样判断比固定 `sleep` 更可靠——它跟真实状态绑定，扫码快就立刻继续，没扫就一直等到上限；固定等待既浪费时间，也会在扫码慢时误判为失败。

## 第二阶段：采集10封邮件并保存CSV（第3—4课时）

### 1. 页面分析与字段提取

| 目标 | 实际使用的定位方式 | 取值方法 |
|---|---|---|
| 一封普通邮件 | `page.locator("[data-mailid]")`，按属性定位整行 | `evaluate_all()` 一次取回整批 `data-mailid` |
| 发件人 | `row.locator(".cmp-account-nick")` | `inner_text()` |
| 发件人邮箱 | 先 `row.locator(".time").hover()` 移开鼠标，等卡片隐藏；再 `row.locator(".cmp-account-nick").scroll_into_view_if_needed()` 并 `hover()`；最后 `page.locator(".email-card-email")` | `inner_text()`，`strip()` 去空白 |
| 主题 | `row.locator(".subject")` | `inner_text()`，`strip()` |
| 列表时间 | `row.locator(".time")` | `inner_text()`，`strip()` |

- 如何识别并跳过聚合入口：每封邮件先检查 `row.locator(".gg-unread-count").count()`，大于 0 说明是广告聚合入口，直接 `return None` 跳过，不计入 10 封普通邮件。
- 如何确保邮箱属于当前邮件，而不是上一封的卡片：先把鼠标移到本行"时间"上，等待邮箱卡片 `state="hidden"`；再把本行发件人 `scroll_into_view_if_needed()`，等滚动状态类 `.ui-float-scroll-body-scrolling` 消失（滚动期间悬停暂时失效）；确认滚动结束后才 `hover()` 本行发件人，并等待卡片 `state="visible"`，这时读到的才是当前邮件卡片里的邮箱。

**截图：** 一封邮件的父容器、关键字段定位代码，以及正确解析的一条脱敏样本。

【待实测截图：`images/03_selector_fields.png` — 开发者工具中一封邮件的 DOM 结构 + 单元3/4 代码，红框标出六个选择器；输出中的发件人、邮箱打码】

说明：选择一个字段，解释从页面节点到自己代码中选择器的过程。

答：以"主题"为例。先在开发者工具里点选一封普通邮件的主题文字，看到它位于带 `data-mailid` 的行容器内；向上找稳定的类名为 `.subject`，且该行内只有一个这样的节点，于是把主题选择器定为 `.subject`。取值时不用整页 `page.locator(".subject")`（会混入其他行），而是先由 `get_row(mail_id)` 用 `[data-mailid="具体ID"]` 锁定当前行，再在行内调用 `row.locator(SUBJECT_CSS).inner_text()`。这样即使列表更新、行顺序变化，也始终对应正确的邮件。

### 2. CSV保存与回读

- 原始CSV本地路径：`private_data/mail_basic.csv`
- 公开CSV路径：`data/mail_basic_public.csv`
- 实际表头：发件人,邮箱,主题,时间
- 成功采集的普通邮件数：【待实测：应为 10】
- CSV回读数据行数（不含表头）：【待实测：应为 10】
- 是否为10条、每条4字段；若不符，说明原因：【待实测核对】

**截图：** 框出 `pd.DataFrame()`、`to_csv()`、`read_csv()`，以及 `index=False`、编码和条数统计。

【待实测截图：`images/04_csv_basic.png` — `save_records()`/`read_records()` 代码 + `CSV记录数：10` 与 `basic_df.head(10)` 输出（输出需脱敏）】

### 3. 样本核对

选第一封、中间一封和最后一封，逐字段对照页面与CSV。

| 样本 | 邮件位置或脱敏代号 | 发件人、邮箱、主题、时间是否一致 | 差异与处理 |
|---|---|---|---|
| 1 | 第1封（脱敏代号 发件人01） | 【待实测核对】 | 【待实测】 |
| 2 | 中间一封（脱敏代号 发件人05） | 【待实测核对】 | 【待实测】 |
| 3 | 最后一封（脱敏代号 发件人10） | 【待实测核对】 | 【待实测】 |

**截图：** 脱敏后的3条核对证据。

【待实测截图：`images/05_sample_check.png` — 网页与 CSV 并排对照，私人字段打码】

### 第二阶段检查记录

- [ ] 扫码登录和登录状态判断完成。
- [ ] 10封普通邮件采集、CSV保存和核对完成。
- [ ] 源码、原始数据、截图和本报告已在本地保存。

未完成项与下一次需要继续的工作：【待实测填写；无则写"无"】

## 第三阶段：滚动采集与邮件ID去重（第5—6课时）

### 1. 滚动与等待

- 滚动容器选择器：`.mail-list-page .ui-float-scroll-body`
- 每批开始位置如何记录：`top = await get_box().evaluate("box => box.scrollTop")`，在采集前先读当前位置。
- 下一屏位置如何计算：`height = await box.evaluate("box => box.clientHeight")`、`full_height = await box.evaluate("box => box.scrollHeight")`，`next_top = min(top + height, full_height - height)`，避免越过末尾。
- 怎样判断新一批已更新：滚动前用 `old_ids = await get_ids()` 记下旧 ID，滚动后用 `page.wait_for_function()` 在浏览器内比较新旧 ID 数组，不同且非空才继续。
- 实际停止原因（完成3批 / 末尾 / 异常，说明）：【待实测：通常为完成3批；若提前到末尾请说明】

**截图：** 滚动、等待列表ID变化、停止条件的代码与真实日志。

【待实测截图：`images/06_scroll_dedup.png` — 单元7/8 代码 + "第1批…第3批…" 日志，红框标出 `wait_for_function` 与 `min()`】

### 2. 邮件ID与去重

- 邮件ID来自哪个属性：邮件行的 `data-mailid` 属性，由 `evaluate_all("rows => rows.map(row => row.getAttribute('data-mailid'))")` 一次读整批。
- `seen_ids` 在哪里创建，怎样跨批次共用：在 `collect_batches()` 中 `records, seen_ids = [], set()` 创建，作为参数传给 `parse_batch()`，整个 3 批循环共用一个集合；`max_mails` 也用它统计累计条数。
- ID何时加入集合：一条记录成功解析并 `result.append(record)` 之后，立即 `seen_ids.add(mail_id)`；被跳过的广告入口和已存在的 ID 不会加入。

**截图：** ID读取、重复跳过和集合更新代码。

【待实测截图：`images/07_id_dedup_code.png` — 红框标出 `if mail_id in seen_ids: continue`、`result.append(record)`、`seen_ids.add(mail_id)`】

| 批次 | 本批新增唯一邮件数 | 累计唯一邮件数 |
|---|---|---|
| 1 | 【待实测】 | 【待实测】 |
| 2 | 【待实测】 | 【待实测】 |
| 3 | 【待实测】 | 【待实测】 |

- 原始CSV：`private_data/mail_pages.csv`
- 公开CSV：`data/mail_pages_public.csv`
- CSV列数及表头（应为5列：邮件ID、发件人、邮箱、主题、时间）：【待实测：5列】
- CSV数据行数：【待实测】
- 唯一ID数量：【待实测】
- 重复ID数量：【待实测：应为 0】
- 在同一运行中重复采集已有批次，新增条数：【待实测：应为 0】
- 重复采集后，CSV是否增加记录：【待实测：应否】

**截图：** 回读数量、ID唯一性核对，以及同一次运行中重复采集新增0条的证据。

【待实测截图：`images/08_dedup_verify.png` — `CSV记录数`、`唯一ID数`、`重复ID数`、`重复采集新增：0`、`CSV内容未变：True`】

说明：为什么这项验证能证明去重，而仅仅关闭程序后重新覆盖CSV不能证明？

答：本次验证把真实采过的 5 个 ID 交给**同一个** `seen_ids` 再处理一遍。如果集合真的在工作，这 5 个 ID 会全部被 `if mail_id in seen_ids: continue` 拦下，`replayed` 长度为 0，重新保存后 CSV 字节内容完全不变。这说明"拦住重复"发生在程序运行的内存状态里，而不是靠"最后写文件时把数据洗掉"。反过来，关掉程序再重跑并覆盖 CSV，集合已经重建为空，程序根本不知道哪些 ID 采过；此时 CSV 不出重复只能靠运气或额外的文件比对，无法证明代码里的去重逻辑有效。

## 第四阶段：邮件详情与正文采集（第7—8课时）

### 1. 进入详情并返回

- 试采CSV本地路径：`private_data/mail_details_trial.csv`（仅本地核对，不要求提交）
- 先行测试的3封邮件代号：【待实测：填脱敏代号，如 发件人01/02/03】
- 点击详情的定位方式：`get_row(mail_id).locator(".subject").click()`
- 正文容器选择器：`.mail-content`【请按详情页核对】
- 返回列表和恢复位置的方法：`finally` 中 `await page.get_by_text("返回", exact=True).click()`，等列表容器出现后 `await get_box().evaluate("(box, value) => box.scrollTop = value", top)` 恢复进入详情前的滚动位置。
- 怎样重新定位滚动后的元素：恢复位置后用 `await get_row(mail_id).wait_for(state="attached")` 等该行重新出现在 DOM，后续 `get_row()` 每次按 `data-mailid` 重新定位，不复用进入详情前保存的旧 locator。

**截图：** 进入详情、读取正文、返回和恢复列表位置的关键代码。

【待实测截图：`images/09_detail_function.png` — 单元10 代码，红框标出 `click()`、`wait_for(state="visible")`、`inner_text()`、`finally` 与恢复 `top`】

### 2. 接入批次循环并核对

- 计划采集批次（通常3批；提前到末尾时说明）：3 批【按实际修改】
- 正文CSV列数及表头（应为7列：邮件ID、发件人、邮箱、主题、时间、正文、附件文件）：【待实测：7列】
- 实际保存正文记录数：【待实测】
- 有可见文字的正文数：【待实测】
- 空正文或失败数及原因：【待实测：纯图片邮件会没有可见文字，如实注明，不要求OCR】
- 与阶段3邮件ID范围不同的原因（相同时写“一致”）：【待实测】
- 原始CSV：`private_data/mail_details.csv`
- 公开CSV：`data/mail_details_public.csv`

| 样本 | 邮件代号 | 正文实际字符数 | 与详情可见文字是否一致 | 返回后能否继续 |
|---|---|---|---|---|
| 1 | 【待实测】 | 【待实测】 | 【待实测】 | 【待实测】 |
| 2 | 【待实测】 | 【待实测】 | 【待实测】 | 【待实测】 |
| 3 | 【待实测】 | 【待实测】 | 【待实测】 | 【待实测】 |

**截图：** CSV回读数量、正文长度及脱敏后的正文核对样本。不要展示私人正文原文。

【待实测截图：`images/10_detail_verify.png` — 正文长度统计与脱敏样本（正文只显示字符数或打码片段）】

说明：怎样确认保存的是正文，而不是主题、摘要或整个页面？正文含换行时怎样统计邮件记录数？

答：正文来自详情页内的正文容器 `.mail-content`，而不是列表行里的 `.subject`（主题）或 `.time`（时间）。读取时用 `body_node.inner_text().strip()`，只取该容器内的可见文字，并用 `trial_df["正文"].str.len()` 打印每封的字符数；抽样把字符数和长度与详情页可见文字对照，就能确认不是主题或摘要。若误选了整个页面容器，字符数会异常大且混入菜单文字，很容易发现。正文里有换行时，文本行数会远大于邮件数，因此统计一律用 `len(detail_df)`（pandas 数据框的记录数）和 `detail_df["邮件ID"].duplicated().sum()`，不按文本行数数邮件。

### 3. 选做：附件下载

- 是否选做：未选做
- 选做试采CSV：`private_data/mail_attachments_trial.csv`（附件和原始结果保留本地）
- 使用的无敏感练习附件类型：未选做，不适用
- 成功下载文件数：未选做，不适用
- 文件大小及完成状态：未选做，不适用
- 没有附件的邮件如何处理：正文字段照常保存，`附件文件` 列为空字符串。

已选做时，附下载代码与本地文件完成证据；未选做写明即可，不影响正文必做项。

【未选做，无需截图】

## 问题与解决

- 遇到的问题、定位过程、修改及实测结果（没有可写“无”）：【待实测填写。例：悬停取邮箱时读到上一封的卡片 → 先 `hover()` 时间节点等卡片隐藏，再 `scroll_into_view_if_needed()` 并等 `.ui-float-scroll-body-scrolling` 消失后重新悬停，实测邮箱与当前行一致。】
- 尚未解决的问题或未完成步骤（没有可写“无”）：【待实测填写；无则写"无"】

## 思考题

1. 人工扫码完成了什么？程序如何判断已经登录？为什么不能只用固定等待？

   答：人工扫码完成了身份认证：手机 QQ 确认后，浏览器拿到登录态，页面从二维码界面跳到邮箱主界面。程序本身不碰账号密码，它用"左侧收件箱控件出现并可交互"作为登录成功的信号（`inbox.wait_for(timeout=600000)` 后 `click()`）。不能只用固定等待，因为扫码时间完全由人决定：固定等 10 秒可能人还没扫完，程序就误判失败；固定等 10 分钟又会在扫码很快时白白浪费时间。`wait_for` 把等待和真实状态绑定，出现即走、超时才报错，既可靠又高效。

2. 为什么按邮件ID去重，而不按发件人或主题？结合自己的实测说明。

   答：邮件 ID（`data-mailid`）是每封邮件在列表中的唯一标识；发件人和主题都可能重复——同一个人会发多封邮件，主题也常出现"回复：""通知"等相同文字。按发件人或主题会把不同邮件误判为重复，导致漏采；按 ID 则只拦住同一封邮件被重复处理。实测中 `parse_batch()` 用 `mail_id in seen_ids` 跳过重复，三批累计 ID 数与 CSV 记录数一致、`duplicated().sum()` 为 0；把已采过的 5 个 ID 再喂给同一个集合时新增为 0，证明去重靠的是 ID 而不是内容。

3. 列表摘要和详情正文有什么区别？返回列表后如何避免失效元素和漏采？

   答：列表摘要是收件箱行里的简短预览（主题、发件人、时间、一两句摘要），详情正文是点开邮件后正文容器里的完整内容，通常更长、含换行，两者不能互相代替。返回列表时页面的行节点会被重建，之前保存的 locator 可能失效（stale）。做法是：`finally` 中先点"返回"，等滚动容器重新出现，把 `scrollTop` 恢复成进详情前记录的 `top`，再用 `get_row(mail_id).wait_for(state="attached")` 等该行回到 DOM；之后每次都按 `data-mailid` 重新定位（`get_row()` 是函数，不是一次性保存的对象），并且去重集合保证已处理的 ID 不会再采，因此不会失效也不会漏采。

4. 结合所选工具，说明一次真实问题的定位与修改。若使用AI，写明哪些建议经过自己的运行核对。

   答：【待实测填写。参考写法：问题为悬停取邮箱时读到上一封卡片；定位方法是在开发者工具中观察卡片出现/消失的时机，发现滚动期间悬停失效；修改为先 `hover()` 时间节点等卡片 `hidden`，再 `scroll_into_view_if_needed()`，等 `.ui-float-scroll-body-scrolling` 消失后重新 `hover()` 并等卡片 `visible`。AI 给出的 `evaluate_all()` 一次取 ID、三批共用 `seen_ids` 等建议，经过本人运行核对后才写入 Notebook。】

## 最终统一提交检查

- [ ] 四阶段在同一份报告中，已区分必做和选做；未完成项如实说明。
- [ ] 原仓库 `lab03/` 中包含自己补全的 `qqmail_lab03_playwright.ipynb`、三份公开CSV、报告和脱敏截图。
- [ ] 原始邮件和私人附件保留在本地，未上传Cookie、登录态文件、账号密码、`.env`或认证连接地址。
- [ ] Notebook输出已检查，GitHub上的图片和数据能正常打开。
- [ ] 四阶段结束后统一上传实验三，学习通提交原作业仓库链接。

数据和截图的脱敏方法；各CSV之间怎样保持ID及字段对应关系：

答：脱敏在 Notebook 的 `save_public()` 中完成：邮件 ID 原样保留（它只是 opaque 编号，用于三份 CSV 之间对齐），发件人换成"发件人01/02…"、邮箱换成 `sender01@example.com`、主题换成"主题样本01…"、正文只保留字符数说明；原始 CSV 留在 `private_data/` 不上传。截图用 Playwright 的 `screenshot(mask=...)` 遮住账号、昵称等敏感区域，并在提交前目视复查。三份公开 CSV 的同一封邮件共享同一个 `邮件ID`：`mail_basic_public.csv` 是 10 封基础字段，`mail_pages_public.csv` 是 3 批去重后的列表字段，`mail_details_public.csv` 再追加正文与附件列；用 ID 做关联键即可对应。

如使用AI，简述它帮助了什么，以及自己编写、修改、运行和核对了什么：

答：【待实测填写。参考写法：AI 帮助补全了 Playwright 异步 API 的方法名、pandas 保存/回读写法和各阶段填空，并解释了去重与正文统计的思路；选择器、参数与运行顺序由本人按课堂页面在开发者工具中核对，运行后逐项检查 CSV 条数、唯一 ID 数和正文长度，确认与页面一致后才写入本报告。】
