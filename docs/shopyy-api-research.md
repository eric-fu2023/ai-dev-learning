# ShopYY 商品与订单 Open API 调研

## 1. 调研结论

本次查阅的是 ShopYY 官方公开的 Open API 项目及开发者帮助资料。结论如下：

| 能力 | 结论 |
| --- | --- |
| 查询店铺商品 | 支持。可查询商品列表、分页总数和商品详情，返回商品、SKU、价格、库存等信息。 |
| 管理商品 | 支持。公开目录包含新增、编辑、批量编辑、库存更新、上下架和删除等接口。 |
| 查询订单 | 支持。可查询订单列表、订单详情和订单编号对应的详情。 |
| 已有订单支付 | 支持“手动支付”接口，但它操作的是已经存在的订单，不是创建订单。 |
| 发货、退款、作废 | 支持相应的订单管理接口。 |
| 创建新订单 | 在当前公开的 132 个接口目录中未发现 `POST /orders` 或等价的后台创建订单接口。 |

因此，ShopYY 的公开 Open API 可以满足商品同步和已有订单管理，但目前不能据此确认能够直接通过管理端 API 创建新订单。若业务目标是“程序下单”，需要使用店铺前台的官方结账流程，或向 ShopYY 申请有权限的订单创建能力。

## 2. 资料范围与认证

### 2.1 官方资料

- [ShopYY Open API 项目](https://www.apizza.net/project/e114fb8e628e0f604379f5b26f0d8330/browse)
- [ShopYY 开发者帮助](https://oemsaas.yuque.com/books/share/e1def969-9039-41db-95f3-bfc87c255c0c/stn3g2)
- [开发者应用说明](https://oemsaas.yuque.com/books/share/e1def969-9039-41db-95f3-bfc87c255c0c/ozu14nxkengbiolc)

公开 Apizza 项目信息：

- 项目名称：`OemSaas Open Api`
- 所属企业：`Shopyy`
- 公开项目 ID：`e114fb8e628e0f604379f5b26f0d8330`
- 目录接口数：`132`
- 所有者：`OemSaas / tech@35zh.cn`

### 2.2 业务 API 认证

业务接口的 URL 以 `{{host}}` 作为占位符，请从目标店铺后台的开发者应用配置中获取实际 API Host。请求头使用店铺开发者 Token：

```http
token: <SHOPYY_TOKEN>
```

这些接口面向已授权的自有店铺或合作店铺，并不是任意 ShopYY 店铺的公共商品接口。生产 Token 不应写入代码、提交到 Git 或直接发送到聊天工具，建议通过环境变量注入。

## 3. 商品 API

### 3.1 查询接口

| 用途 | 方法 | 路径 | 说明 |
| --- | --- | --- | --- |
| 商品列表（含总数） | `GET` | `/products/list?page=1&limit=20` | 分页查询商品列表并返回分页总数。`page`、`limit` 为必填查询参数。 |
| 商品列表 | `GET` | `/products` | 查询商品列表。具体筛选参数以文档和店铺权限为准。 |
| 商品详情 | `GET` | `/products/{id}` | 根据商品 ID 查询详情。 |

商品列表文档 ID：`c72a90e699174e46870514f11b6c1999`。

商品详情文档 ID：`f66019ab2a28232731c3c3c5adca7cb8`。

### 3.2 返回数据

商品响应中可见的主要字段包括：

- 商品：`id`、`title`、`handle`、`status`、`images`、`variants`
- SKU/变体：`id`、`sku`、`sku_code`、`price`、`compare_at_price`、`inventory_quantity`
- 规格：`option1`、`option2`、`option3` 及对应的标题和值
- 变体图片及商品集合信息

实际同步时应以变体 ID 或 SKU 作为库存和下单前校验的关键标识，不要只依赖商品标题。

### 3.3 商品管理接口

公开目录还包含以下商品写入接口。它们会修改店铺数据，调用前应先在测试店铺验证权限、请求体和幂等策略：

| 用途 | 方法 | 路径 |
| --- | --- | --- |
| 新增商品 | `POST` | `/products` |
| 编辑商品 | `PUT` | `/products/{id}` |
| 批量编辑 | `POST` | `/products/batchsave` |
| 批量更新库存 | `POST` | `/products/batchinventory` |
| 批量上下架 | `POST` | `/products/batchstatus` |
| 批量删除 | `POST` | `/products/batchdelete` |

新增商品文档 ID：`cd81024768387f6c5e71eebef9603b12`。

### 3.4 查询示例

以下示例只展示请求格式，不包含真实 Host 或 Token，也不会触发本地或远程业务请求：

```bash
curl \
  -H "token: $SHOPYY_TOKEN" \
  "$SHOPYY_API_HOST/products/list?page=1&limit=20"
```

其中：

- `SHOPYY_API_HOST` 是店铺后台提供的实际 API Host。
- `SHOPYY_TOKEN` 是该店铺开发者应用的授权 Token。

## 4. 订单 API

### 4.1 已公开的订单接口

| 用途 | 方法 | 路径 | 说明 |
| --- | --- | --- | --- |
| 订单列表 | `GET` | `/orders` | 查询已有订单。 |
| 按订单编号查询详情 | `GET` | `/orders/ordernumber/:order_number` | 根据订单编号查询已有订单详情。 |
| 按订单 ID 查询详情 | `GET` | `/orders/{id}` | 公开文档示例使用 `/orders/29506`，实际调用时应按文档约定替换订单 ID。 |
| 手动支付 | `POST` | `/orders/{id}/checkout` | 为已有订单登记手动支付信息，不是创建订单。 |
| 批量发货 | `POST` | `/orders/fulfillments` | 批量创建发货信息。 |
| 部分发货 | `POST` | `/orders/partfulfillments` | 对订单执行部分发货。 |
| 批量作废 | `POST` | `/orders/batchcancel` | 批量作废订单。 |
| 退款 | `POST` | `/orders/{id}/refund` | 对已有订单执行退款。 |
| 批量设置标签 | `POST` | `/orders/tags` | 为订单批量设置标签。 |

订单接口还包含取消发货和 COD 订单的查询、状态及发货接口，是否可用取决于店铺配置和授权范围。

### 4.2 手动支付请求体

公开文档中“手动支付”接口要求以下字段：

```json
{
  "order_number": "<已有订单编号>",
  "customer_name": "<付款人>",
  "payment_id": 0,
  "transaction_no": "<真实交易流水号>",
  "note": "手动支付"
}
```

该请求必须绑定到路径中的已有订单 ID，并提供真实的支付方式和交易流水信息。它不能用来凭空创建订单，也不应被用来绕过支付流程。

## 5. 下单能力边界

### 5.1 当前公开目录没有创建订单接口

对公开项目的完整接口目录进行检索后，订单相关接口主要集中在查询、支付登记、发货、作废、退款和标签管理；没有发现类似下面的后台创建订单接口：

```http
POST /orders
```

因此，不能把下面这个接口当作“下单 API”：

```http
POST /orders/{id}/checkout
```

它的文档名称是“手动支付”，请求体也要求已有订单编号、付款人、支付方式和交易流水号，语义是对已有订单补登记支付信息。

### 5.2 可能的实现路径

如果目标是让程序完成一次真实购买，建议按以下顺序确认：

1. 先向 ShopYY 确认店铺是否有官方的订单创建接口、应用权限或合作方接口。
2. 如果没有管理端创建接口，使用店铺前台公开的 checkout 流程，并按照官方文档处理结账、配送和支付回调。
3. 将商品查询、SKU/库存校验、价格校验和订单结果查询作为独立步骤实现。
4. 不抓取或依赖未公开的内部接口，不模拟或绕过第三方支付，也不使用手动支付接口伪造订单支付状态。

帮助中心中另有“开始结算”“订单生成”“单页结账/添加订单”“选择配送”和“订单支付”等前台结账流程章节。这些资料描述的是前台流程或回调数据，不能直接等同于公开的后台 `POST /orders` API；实际接入前仍需获取店铺对应版本的官方说明和授权。

## 6. 对接前置条件与建议

在开始编码前，需要从目标店铺确认：

- 目标是自有店铺还是已取得明确授权的第三方店铺。
- 开发者应用提供的实际 API Host。
- Token 的权限范围、有效期、调用频率限制和测试环境。
- 订单创建目标是后台订单，还是前台 checkout 订单。
- 支付、配送、退款和订单回调的业务责任边界。

建议先实现只读商品同步，再在测试店铺验证订单查询。订单创建和支付属于有副作用的操作，应在确认官方授权、幂等规则、库存扣减时机和异常补偿方案后再上线。

## 7. 资料核验说明

本报告记录的是截至 **2026-10-18** 对公开资料的核验结果。核验访问了公开 Apizza 项目页面、项目元数据和接口目录；没有使用任何店铺 Token，也没有调用 ShopYY 店铺业务接口、创建订单、支付、发货或修改商品。

Apizza 的以下地址是“读取 API 文档”的资料服务地址，不是 ShopYY 店铺业务 API：

```http
POST https://rest.apizza.net/rest/v2/project/getProjectInfo
POST https://rest.apizza.net/rest/v2/Category/getList
POST https://rest.apizza.net/rest/v2/Api/info
```

公开文档可能随 ShopYY 版本、权限或项目维护发生变化。正式开发前应以店铺后台当前显示的开发者文档、Host、权限和接口响应为准。
