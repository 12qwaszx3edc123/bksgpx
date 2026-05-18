# pkg

## MerchantInfoService

### ES添加
```go
// 同步 Elastic添加
m := map[string]interface{}{
“id”, “name”, “image”, “price”, “stock”, // 菜品基本信息
“merchant_id”, “type”, “month_sales”, // 商家和销量信息
“rating”, “is_new”, “status”, // 状态信息
}
_, err = config.Esc.Index().
Index(“commodity”). // 索引名称
BodyJson(m). // 文档内容
Do(context.Background()) // 执行
```

---

### ES列表

```go
// 分页
offset, size := pkg.Paginate(int(req.Page), int(req.Size))

// 布尔查询构建
boolQuery := elastic.NewBoolQuery()

// 名称模糊搜索
elastic.NewMatchQuery("name", req.Name).Must(boolQuery)

// ID精准搜索
elastic.NewTermQuery("merchant_id", req.MerchantId).Must(boolQuery)

// 价格区间查询
elastic.NewRangeQuery("price").Gte(req.MinPrice).Lte(req.MaxPrice).Must(boolQuery)

// 排序：格式 "字段_方向"，如 "Rating_Desc"、"price_Asc"
sortQuery := elastic.NewFieldSort("rating")      // 按评分排序
sortQuery := elastic.NewFieldSort("month_sales")  // 按月销量排序
sortQuery := elastic.NewFieldSort("price")        // 按价格排序
sortQuery.Desc()                                   // 倒序
sortQuery.Asc()                                    // 正序

// 执行搜索
do, err := config.Esc.Search().
    Index("commodity").    // 索引名称
    From(offset).          // 偏移量
    Size(size).            // 每页数量
    Query(boolQuery).      // 查询条件
    SortBy(sortQuery).     // 排序规则
    Do(context.Background())

// 总条数
total := do.Hits.TotalHits

// 结果解析
for _, hit := range do.Hits.Hits {
    json.Unmarshal(hit.Source, list)  // 反序列化每条记录
}
```

---

### ES查询

```go
// 根据ID查询单条文档
do, err := config.Esc.Get().
    Index("commodity").              // 索引名称
    Id(strconv.Itoa(int(id))).       // 文档ID
    Do(context.Background())        // 执行

// 解析结果
if do.Found {
    json.Unmarshal(do.Source, &result)  // 反序列化文档内容
}
```

---

### ES修改

```go
// 根据ID更新文档字段
m := map[string]interface{}{
    "name",   // 修改的字段
    "price",  // 修改的字段
    "stock",  // 修改的字段
}
_, err = config.Esc.Update().
    Index("commodity").              // 索引名称
    Id(strconv.Itoa(int(id))).       // 文档ID
    Doc(m).                          // 更新内容
    Do(context.Background())        // 执行
```

---

### ES删除

```go
// 根据ID删除文档
_, err = config.Esc.Delete().
    Index("commodity").              // 索引名称
    Id(strconv.Itoa(int(id))).       // 文档ID
    Do(context.Background())        // 执行
```

---

## Redis 购物车

Key格式：`cart:{UserId}:{DrugId}`

### 购物车添加/修改

```go
key := fmt.Sprintf("cart:%d:%d", UserId, DrugId)  // key格式
config.RDB.HMSet(config.Ctx, key, drugMap).Err()   // 添加/修改商品
```

---

### 购物车是否存在

```go
key := fmt.Sprintf("cart:%d:%d", UserId, DrugId)   // key格式
config.RDB.Exists(config.Ctx, key).Val() > 0       // 返回true/false
```

---

### 购物车数量更新

```go
key := fmt.Sprintf("cart:%d:%d", UserId, DrugId)         // key格式
config.RDB.HIncrBy(config.Ctx, key, "quantity", 1).Err()  // 数量+1
config.RDB.HIncrBy(config.Ctx, key, "quantity", -1).Err() // 数量-1
```

---

### 购物车列表

```go
key := fmt.Sprintf("cart:%d:*", UserId)       // 匹配该用户所有商品key
config.RDB.Keys(config.Ctx, key).Val()        // 返回 []string
```

---

### 购物车删除单个

```go
key := fmt.Sprintf("cart:%d:%d", UserId, DrugId)  // key格式
config.RDB.Del(config.Ctx, key).Err()              // 删除商品
```

---

### 购物车清空

```go
key := fmt.Sprintf("cart:%d:*", UserId)            // 匹配该用户所有商品key
keys := config.RDB.Keys(config.Ctx, key).Val()    // 获取所有key
for _, k := range keys {
    config.RDB.Del(config.Ctx, k).Err()             // 逐个删除
}
```

---

## 支付异步回调

### Notify

```go
func Notify(c *gin.Context) {
	err := c.Request.ParseForm()
	if err != nil {
		fmt.Println("参数获取失败")
		c.String(200, "参数解析失败")
	}
	m := make(map[string]string)
	for s, strings := range c.Request.PostForm {
		m[s] = strings[0]
	}
	fmt.Println(m)
	if m["trade_status"] != "TRADE_SUCCESS" {
		c.String(200, "交易失败")
		return
	}
	orderSn := m["out_trade_no"]
	if orderSn == "" {
		c.String(200, "订单号不存在")
		return
	}
	var order model.Order

	if err := order.FindOrderByOrderSn(config.DB, orderSn); err != nil {
		c.String(200, "订单不存在")
		return
	}
	tx := config.DB.Begin()
	order.OrderType = 2
	order.PayType = 2

	if err := order.OrderSave(tx); err != nil {
		tx.Rollback()
		c.String(200, "订单状态更新失败")
		return
	}
	var drug model.DrugAdd

	if err := drug.FindDrugById(tx, int64(order.DrugId)); err != nil {
		tx.Rollback()
		c.String(200, "药品信息不存在")
		return
	}
	drug.Stock -= order.Number

	if err := drug.Save(tx); err != nil {
		tx.Rollback()
		c.String(200, "修改失败")
		return
	}
	tx.Commit()
	c.String(200, "交易成功")
}
```
```