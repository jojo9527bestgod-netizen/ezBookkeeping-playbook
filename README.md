# ezBookkeeping Playbook

> 来源：本地 ezBookkeeping 当前"支出 > 交易分类"页面抓取

## Docker 部署（必须用 volume 持久化！）

**重要：每次部署新容器必须用 `-v` 挂载 volume，否则删容器=删数据！**

```bash
# 正确部署方式
docker run -d --name ezbookkeeping -p 8080:8080 -v ~/ezbookkeeping-data:/ezbookkeeping/data mayswind/ezbookkeeping

# 数据位置
~/ezbookkeeping-data/ezbookkeeping.db

# 常用命令
docker restart ezbookkeeping  # 重启
docker logs ezbookkeeping      # 查看日志
```

### 账号信息

- 用户名：`jojo9527`
- 密码：`Jojo@2026ez!`
- 语言：中文（zh-Hans）
- 货币：CNY

---

## 依赖与前提

这套记账/分类流程默认依赖以下环境：

- **ezBookkeeping 服务可访问**：默认 `http://localhost:8080`
- **已存在可登录账号**：当前记录使用的本地账号是 `jojo9527`
- **Agent Browser 可用**：用于前端页面操作、读取 localStorage、辅助排障
- **Docker 可用**：当前 ezBookkeeping 运行在 Docker 容器中，容器名为 `ezbookkeeping`
- **后端接口可访问**：用于新增后核验、按 `categoryId` 直接修正分类

### 当前实际工具链

- OpenClaw skill：`Agent Browser`
- 浏览器控制：`agent-browser`
- 容器管理：`docker`
- 账本服务：`mayswind/ezbookkeeping`

### 说明

- 仅有分类表还不够；真正稳定执行"记录 + 分类 + 核验"，需要同时具备：
  1. ezBookkeeping 可登录
  2. token 可正常生成
  3. 数据库文件可写
  4. Agent Browser / Docker 可正常工作
- 如果其中任一环节异常，前端可能看起来能打开，但实际无法稳定记账或改分类。

## 一级分类 → 二级分类

### 食品饮料
- 食品
- 饮料
- 水果零食

### 服饰外貌
- 衣服
- 饰品
- 化妆品
- 美容美发

### 住宅家居
- 家居用品
- 电子产品
- 维修保养
- 家政服务
- 水电煤气
- 租金贷款

### 交通出行
- 公共交通
- 打车租车
- 私家车费用
- 火车票
- 飞机票

### 交流通讯
- 电话费
- 上网费
- 快递费

### 休闲娱乐
- 运动健身
- 聚会支出
- 电影演出
- 玩具游戏
- 会员订阅
- 宠物花费
- 旅游度假

### 教育学习
- 书报杂志
- 培训课程
- 认证考试

### 礼物捐赠
- 礼物
- 捐赠

### 医疗健康
- 检查治疗
- 药品
- 医疗器械

### 金融保险
- 税费支出
- 手续费
- 保险支出
- 利息支出
- 赔偿罚款

### 其他杂项
- 其他支出

---

## 默认映射建议（可继续补）

- 早餐 / 面包 / 午饭 / 买菜 / 零食 → 食品
- 打羽毛球 / 穿线 / 运动 → 运动健身
- 买锅 / 刀架 / 杯子 / 家居小物 → 家居用品
- 洗澡 / 洗护 / 理发 → 美容美发
- 洗车 → 私家车费用（如果是汽车相关）或公共交通（如果你仍想这么记）
- 快递 → 快递费
- 话费 → 电话费
- 网费 → 上网费

> 后续可继续扩充成"常见描述 → 推荐分类"的完整映射表。

---

## 已确认的真实后端接口

这次排查已经确认：

### 读取分类列表
```http
GET /api/v1/transaction/categories/list.json
```

### 读取当月交易列表
```http
GET /api/v1/transactions/list/by_month.json?year=YYYY&month=M&type=0&category_ids=&account_ids=&tag_filter=&amount_filter=&keyword=&trim_account=true&trim_category=true&trim_tag=true
```

### 读取单笔交易详情
```http
GET /api/v1/transactions/get.json?id=<transactionId>&with_pictures=true&trim_account=true&trim_category=true&trim_tag=true
```

### 修改交易（最关键）
```http
POST /api/v1/transactions/modify.json
```

用于修改：
- `categoryId`
- `comment`
- `sourceAmount`
- `time`
- `tagIds`
等字段。

---



### 新增交易
```http
POST /api/v1/transactions/add.json
```

**关键发现**：必须添加 `type: 3` 参数（实测有效）

请求体格式：
```json
{
  "type": 3,
  "categoryId": "<分类ID>",
  "sourceAccountId": "<账户ID>",
  "destinationAccountId": "0",
  "time": <时间戳>,
  "utcOffset": 480,
  "sourceAmount": <金额(分)>,
  "destinationAmount": 0,
  "hideAmount": false,
  "tagIds": [],
  "pictureIds": [],
  "comment": "<备注>"
}
```

**注意**：
- `type: 3` 是支出类型（实测）
- 金额单位是**分**（如 `1200` = ¥12.00）
- `time` 使用 Unix 时间戳（秒）
- 写入后建议立即用 `transactions/list/by_month.json` 验证

---

## 调用接口时需要的关键请求头

```http
Authorization: Bearer <ebk_user_token>
Accept-Language: zh-Hans
X-Timezone-Offset: 480
X-Timezone-Name: Asia/Shanghai
Content-Type: application/json
```

其中：
- `ebk_user_token` 当前存放在浏览器 localStorage 的 `ebk_user_token`
- 时区头缺失时，接口会报：`client timezone offset is invalid`

---

## modify.json 请求体示例

```json
{
  "id": "3807738548417724416",
  "categoryId": "3805945227907170336",
  "time": 1773116422,
  "utcOffset": 480,
  "sourceAccountId": "3806121177416466432",
  "destinationAccountId": "0",
  "sourceAmount": 1600,
  "destinationAmount": 0,
  "hideAmount": false,
  "tagIds": [],
  "pictureIds": [],
  "comment": "打羽毛球"
}
```

说明：
- 金额以"分"为单位（如 `1600` = ¥16.00）
- `categoryId` 改成目标分类的 id 即可完成分类修正
- 比起前端下拉框操作，直接走这个接口更稳

---

## 记账执行与核验规则（已踩坑）

1. 改完或新增后，先核后端/接口，再报结果。
2. 前端页面不作为唯一真相；列表刷新可能慢、缓存可能脏。
3. 当前更稳的核验方式是：
   - 交易写入后查询 `/api/v1/transactions/list/by_month.json`
   - 必要时再查 `/api/v1/transactions/get.json`
4. 如果前端显示异常，但后端已写入成功，优先怀疑前端刷新/容器状态，而不是马上判定写入失败。
5. 如果出现数据库、token、登录异常，先排查容器状态与数据库权限，再继续记账。

## 本次已确认坑位

- 曾出现"数据已改进数据库，但前端长时间不显示"的情况。
- 原因不是数据没写进去，而是容器仍占用旧数据库状态，导致前端没有及时读到新内容。
- 处理方式：修正数据库写权限后，重启 `ezbookkeeping` 容器。
- 结论：以后以"后端核验优先，前端显示次之"为准。

---

## 数据备份（重要！）

### 备份位置
- 备份目录：`~/ezbookkeeping-data/backups/`
- 文件格式：`ezbookkeeping_YYYYMMDD_HHMMSS.db`

### 自动备份机制（双重保护）

#### 1. 定时备份（cron）
- 每3天凌晨3点执行（电脑开着才执行）
- 保留最近10个备份

#### 2. 事件备份（launchd）
- **开机时**自动执行
- **唤醒时**自动执行
- 即使合盖睡眠也不会丢失备份机会

### 备份脚本位置
```bash
/Users/dora/.openclaw/workspace-assi/scripts/backup_ezbookkeeping.py
```

### 手动备份
```bash
python3 /Users/dora/.openclaw/workspace-assi/scripts/backup_ezbookkeeping.py
```

### 恢复数据

如果数据出问题，按以下步骤恢复：

```bash
# 1. 停止 ezBookkeeping
docker stop ezbookkeeping

# 2. 找到备份文件（最新的）
ls -la ~/ezbookkeeping-data/backups/

# 3. 替换数据库（选一个备份文件）
cp ~/ezbookkeeping-data/backups/ezbookkeeping_20260317_235044.db ~/ezbookkeeping-data/ezbookkeeping.db

# 4. 重启 ezBookkeeping
docker start ezbookkeeping
```

### 备份文件兼容性
- 备份是标准 SQLite 数据库
- 可用以下工具查看：
  - `sqlite3` 命令行
  - DB Browser for SQLite（图形界面）
  - Python pandas 导出 Excel：
    ```python
    import pandas as pd
    import sqlite3
    conn = sqlite3.connect('~/ezbookkeeping-data/ezbookkeeping.db')
    df = pd.read_sql_query('SELECT * FROM "transaction"', conn)
    df.to_excel('backup.xlsx', index=False)
    ```

---

## 记账回复格式

每次记录完成后，按以下格式回复用户：

```
已记录：
- **日期**：YYYY年M月D日（今天/昨天/明天）
- **分类**：<分类名称>
- **金额**：X 元
- **描述**：<备注>
```

**注意**：
- 日期后需要标注"今天"、"昨天"或"明天"
- 金额单位是元，不是分
- 描述即 comment 字段

---

## 记账核验规则（必须执行）

每次添加记录后，必须完整核验以下四项：

### 核验清单

| 字段 | 来源 | 说明 |
|------|------|------|
| 日期 | `time` 时间戳 | 用 `date -r <timestamp>` 转换为可读日期 |
| 分类 | `categoryId` | 需对照分类列表确认名称 |
| 金额 | `sourceAmount` | 单位是**分**，除以100得到元 |
| 描述 | `comment` | 直接核对文字 |

### 核验命令

**1. 查询单条记录**
```bash
curl -s 'http://localhost:8080/api/v1/transactions/get.json?id=<ID>&trim_account=true&trim_category=true&trim_tag=true' \
  -H 'Authorization: Bearer <token>' \
  -H 'Accept-Language: zh-Hans' \
  -H 'X-Timezone-Offset: 480' \
  -H 'X-Timezone-Name: Asia/Shanghai'
```

**2. 查询当月记录（核对最新几条）**
```bash
curl -s 'http://localhost:8080/api/v1/transactions/list/by_month.json?year=2026&month=4&type=0&trim_account=true&trim_category=true&trim_tag=true' \
  -H 'Authorization: Bearer <token>' \
  -H 'Accept-Language: zh-Hans' \
  -H 'X-Timezone-Offset: 480' \
  -H 'X-Timezone-Name: Asia/Shanghai'
```

**3. 时间戳转换**
```bash
date -r 1775037600   # 转换为可读日期
```

### 核验示例

添加一条"打羽毛球22元"记录后：

```json
{
  "id": "3812000749408223232",
  "categoryId": "3808164190548394016",
  "time": 1775037600,
  "sourceAmount": 2200,
  "comment": "打羽毛球"
}
```

核验结果：
- **日期**：`date -r 1775037600` → Wed Apr 1 18:00:00 CST 2026 ✓
- **分类**：`3808164190548394016` = 运动健身 ✓
- **金额**：2200分 = 22元 ✓
- **描述**：打羽毛球 ✓
