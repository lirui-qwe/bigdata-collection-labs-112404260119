# 大数据采集与处理实验

- 姓名：李锐
- 学号：112404260119
- 班级：数据1241

## 目录

```text
bigdata-collection-labs-112404260119/
├── README.md
└── lab01/
    ├── lab1_html_api.ipynb
    ├── REPORT.md
    ├── data/
    │   ├── companies.json
    │   └── devices.json
    └── images/
        ├── 01_device_api.png
        ├── 02_company_source.png
        ├── 03_company_next.png
        └── 04_validation.png
```

## Lab 1 说明

- 平台：SmartPark DataLab（https://115.159.159.82）
- Device：动态加载，通过 `/api/devices` 分页采集 5000 条设备
- Company：静态加载，BeautifulSoup 解析，连续采集 3 页共 60 条企业
- 验证：数量、ID 唯一性、Company–Device 一致性全部通过
