# Voltage Product Library

VoltageEMS 产品模板库，定义储能系统中各类设备的测点、动作点和属性。

## 产品层级

```
Station (场站)
├── ESS (储能分类)
│   ├── Battery (电池)
│   └── PCS (变流器)
├── Generator (发电分类)
│   ├── Diesel (柴油机)
│   └── AC Inverter (交流逆变器)
├── Env (环境监控)
└── Load (负载分类)
    ├── Distribution Board (配电箱)
    ├── Single Phase Load (单相负载)
    └── Three Phase Load (三相负载)
```

## JSON 结构

```json
{
  "name": "Battery",
  "pName": "ESS",
  "canCreateInstance": true,
  "topology": {
    "enabled": true,
    "type": "standalone",
    "image": "battery.svg",
    "connectableProducts": ["Hybrid_Inverter", "PCS"]
  },
  "P": [{"id": 1, "name": "Max Power", "unit": "kw", "type": "number"}],
  "M": [{"id": 1, "name": "SOC", "unit": "%", "type": "number"}],
  "A": [{"id": 1, "name": "Start", "unit": "", "type": "string"}]
}
```

| 字段 | 说明 |
|------|------|
| `name` | 产品名称（唯一标识） |
| `pName` | 父产品名称（用于层级） |
| `canCreateInstance` | 是否允许创建实例 |
| `topology` | 拓扑图配置；不存在或 `enabled: false` 时不参与拓扑 |
| `P` | 属性定义（Property） |
| `M` | 测量点定义（Measurement） |
| `A` | 动作点定义（Action） |

`pName` 只表示产品目录层级，不表示实际设备实例的父子关系。

## 拓扑配置

`topology` 用于描述产品在拓扑图中的表现：

```json
"topology": {
  "enabled": true,
  "type": "standalone",
  "image": "battery.svg",
  "connectableProducts": ["Hybrid_Inverter", "PCS"],
  "components": []
}
```

### 连线兼容规则

`connectableProducts` 用于声明当前产品可与哪些具体产品建立拓扑连线。

```json
"connectableProducts": ["PCS", "Single_Phase_Load", "Three_Phase_Load"]
```

- 仅在 `topology.enabled: true` 的产品上配置该字段。
- 数组元素按产品 `name` 精确匹配；不支持产品族、`pName` 或名称模糊匹配。
- 产品库只需在一侧维护规则。消费端读取后必须将其归一化为无向关系：`A` 声明可连接 `B`，即允许 `A—B` 与 `B—A`。
- 当前版本仅有一种无方向边；`connectableProducts` 不表示电流方向、端口或边类型。
- 未配置该字段不主动声明可连接目标，但仍可被其他产品的规则匹配。
- 同一产品不可连接自身。

支持四种类型：

| 类型 | 说明 | 示例 |
|------|------|------|
| `top-level` | 顶层站点或建筑节点 | Station、Env |
| `standalone` | 独立设备节点 | Battery、PCS、Load |
| `composite` | 由固定设备组件组成的复合设备 | Hybrid Inverter |
| `container` | 拓扑容器或连接节点 | Distribution Board、PV Group |

### Composite 和 Container 组件

引用已有产品时使用 `productName`：

```json
{
  "productName": "Battery"
}
```

非产品化组件使用 `name`、`image`，需要设备选择时使用 `selectableProductTypes`：

```json
{
  "name": "Meter",
  "image": "meter.svg",
  "selectableProductTypes": [
    "Single Phase Load",
    "Three Phase Load"
  ]
}
```

当前不使用 `isLogicalNode`、`required`、`multiple` 字段。逻辑分类节点通常表现为 `canCreateInstance: false` 且没有启用的 `topology`；配电箱则是 `canCreateInstance: false` 且 `topology.enabled: true` 的拓扑容器。

产品字段的完整说明见 [PRODUCT_FIELD_SPECIFICATION.md](PRODUCT_FIELD_SPECIFICATION.md)。

## 使用方式

### Rust 项目

```rust
// 方式 1：Git Submodule
git submodule add https://github.com/your-org/voltage-product-lib products

// 方式 2：编译时嵌入
include_str!("products/Battery.json")
```

### Java 项目

```bash
# Git Submodule
git submodule add https://github.com/your-org/voltage-product-lib src/main/resources/products
```

```java
// 读取 JSON
ObjectMapper mapper = new ObjectMapper();
Product product = mapper.readValue(
    getClass().getResourceAsStream("/products/Battery.json"),
    Product.class
);
```

### 校验

使用 `schema/product.schema.json` 校验产品定义：

```bash
# 使用 ajv-cli
npx ajv validate -s schema/product.schema.json -d "products/*.json"
```

## License

MIT
