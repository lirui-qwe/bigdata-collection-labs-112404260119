# images/ — 脱敏截图清单

运行 Notebook 后按下列文件名截图，提交前确认已遮挡账号、头像、昵称、邮件标题与正文。
Notebook 中的 `shot()` 辅助函数支持 `mask=` 参数做色块遮挡。

| 文件名 | 对应报告位置 | 内容 |
|---|---|---|
| `01_env_version.png` | 第一阶段1 | 环境运行与 Playwright/pandas 版本 |
| `02_login_success.png` | 第一阶段2 | 登录判断代码与"登录成功"输出 |
| `03_selector_fields.png` | 第二阶段1 | 一封邮件的 DOM 与六个选择器 |
| `04_csv_basic.png` | 第二阶段2 | DataFrame/to_csv/read_csv 与 10 条回读 |
| `05_sample_check.png` | 第二阶段3 | 第一/中间/最后一封的脱敏核对 |
| `06_scroll_dedup.png` | 第三阶段1 | 滚动等待代码与 3 批日志 |
| `07_id_dedup_code.png` | 第三阶段2 | ID 读取、重复跳过、集合更新 |
| `08_dedup_verify.png` | 第三阶段2 | 唯一 ID 数、重复 0、CSV 未变 |
| `09_detail_function.png` | 第四阶段1 | 进入详情、读正文、返回并恢复位置 |
| `10_detail_verify.png` | 第四阶段2 | 正文记录数、正文长度与脱敏样本 |

Unit 13 为选做，本版未选做，无需附件截图。
