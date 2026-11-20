# data/ — 提交用的脱敏 CSV

运行 `qqmail_lab03_playwright.ipynb` 后，Notebook 会在本目录生成以下三份脱敏 CSV。
当前文件为目录说明，运行前本目录没有真实数据；请勿手工编造邮件内容。

| 文件名 | 生成单元 | 列 | 说明 |
|---|---|---|---|
| `mail_basic_public.csv` | 单元6 `save_public(...)` | 发件人,邮箱,主题,时间 | 10 封普通邮件，4 列 |
| `mail_pages_public.csv` | 单元8 `save_public(...)` | 邮件ID,发件人,邮箱,主题,时间 | 3 批去重结果，5 列 |
| `mail_details_public.csv` | 单元12 `save_public(...)` | 邮件ID,发件人,邮箱,主题,时间,正文,附件文件 | 正式正文，7 列 |

脱敏规则见 Notebook 的 `save_public()`：邮件 ID 保留用于三份 CSV 之间对齐；
发件人、邮箱、主题、正文均替换为代号或字符数说明。
原始 CSV 保存在 `private_data/`，由 `.gitignore` 忽略，不要上传。
