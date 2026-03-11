# ezBookkeeping 分类表

> 来源：本地 ezBookkeeping 当前“支出 > 交易分类”页面抓取

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

> 后续可继续扩充成“常见描述 → 推荐分类”的完整映射表。

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
- 金额以“分”为单位（如 `1600` = ¥16.00）
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

- 曾出现“数据已改进数据库，但前端长时间不显示”的情况。
- 原因不是数据没写进去，而是容器仍占用旧数据库状态，导致前端没有及时读到新内容。
- 处理方式：修正数据库写权限后，重启 `ezbookkeeping` 容器。
- 结论：以后以“后端核验优先，前端显示次之”为准。
