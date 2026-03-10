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
