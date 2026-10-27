# 大数据采集与处理实验

- 姓名：李锐
- 学号：112404260119
- 班级：数据1241

## 目录

```text
bigdata-collection-labs-112404260119/
├── README.md
├── lab01/
│   ├── lab1_html_api.ipynb
│   ├── REPORT.md
│   ├── data/
│   │   ├── companies.json
│   │   └── devices.json
│   └── images/
│       ├── 01_device_api.png
│       ├── 02_company_source.png
│       ├── 03_company_next.png
│       └── 04_validation.png
└── lab02/
    ├── anjuke.ipynb
    ├── REPORT.md
    ├── data/
    │   └── houseinfo.csv
    └── images/
        ├── 01_mysql_table.png
        ├── 02_selector_html.png
        ├── 03_parse_first.png
        ├── 04_parse_function.png
        ├── 05_pagination_code.png
        ├── 06_pagination_log.png
        ├── 07_csv_code.png
        ├── 08_csv_output.png
        ├── 09_db_import_code.png
        ├── 10_db_read_output.png
        ├── 11_db_import_output.png
        ├── 12_mysql_count.png
        ├── 13_mysql_sample.png
        └── 14_db_check_output.png
```

## Lab 1 说明

- 平台：SmartPark DataLab（https://115.159.159.82）
- Device：动态加载，通过 `/api/devices` 分页采集 5000 条设备
- Company：静态加载，BeautifulSoup 解析，连续采集 3 页共 60 条企业
- 验证：数量、ID 唯一性、Company–Device 一致性全部通过

## Lab 2 说明

- 数据来源：安居客教学站 http://www.bspider.top/anjuke/ （在线采集，HTTP 200）
- 解析：requests + BeautifulSoup，CSS 选择器；房源容器 `div.property`，8 个字段按相对选择器提取
- 翻页：`a.next.next-active` 取 `href`，`urljoin` 补全；共 10 页 600 条，第 10 页按钮禁用后停止
- CSV：`data/houseinfo.csv`，`utf-8-sig`、`newline=""`，按 8 字段去重后 498 条
- MySQL：`smartpark_lab.houseinfo`，`executemany()` 批量入库 498 条；按 8 字段去重后重跑新增 0 条
