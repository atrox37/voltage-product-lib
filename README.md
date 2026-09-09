# Voltage Product Library

产品库定义设备分类、P/M/A 点位、默认展示数据及产品级连线限制。
当前保留 13 个产品。全部设备使用同一种节点形态，不再定义内部组件或容器。

## 产品分类

| type | 产品 |
| --- | --- |
| Station | Station |
| Environment | Env |
| ESS | Battery、PCS |
| Generator | Diesel、PV_Group |
| Inverter | AC_Inverter、Hybrid_Inverter |
| Meter | Meter |
| Load | Single_Phase_Load、Three_Phase_Load、EV_Charging_Load、HVAC_Load |

`type` 只用于产品分组和云端分路分类，不是父设备、连接端口或功率方向。
分类值不要求存在同名产品文件。ESS、Load、Generator、Inverter 分类占位产品及 Distribution_Board 已移除。
Meter 是独立计量产品，其 P/M/A 暂沿用三相负载配置，不表示具备负载调节能力。
AC_Inverter 和 PCS 仍用于独立设备；Hybrid_Inverter 以一个实例表达两类功能。

## 字段约定

| 字段 | 用途 |
| --- | --- |
| name | 稳定产品标识，实例及连接规则按此字段关联 |
| type | 分类 |
| description | 产品说明 |
| topology | 可选；仅参与拓扑绘制的产品定义此对象 |
| topology.image | 可选图片资源标识 |
| topology.connections | 产品级连线规则数组 |
| topology.description | 整体连接说明，用于 tooltip 底部 |
| defaultDisplayMeasureIds | 平级默认展示 Measure ID 数组，全部引用本产品 M |
| P / M / A | 参数、测量及动作定义 |

点位保留 `id/name/unit/type` 和可选 `description/options` 等原有元数据。
ID 在每个产品的每类点表内唯一；删除的 ID 不复用，协议地址在接入层维护。
`options` 是枚举说明，不代表后端已实现枚举校验或转换。
`defaultDisplayMeasureIds` 通常为 3–5 个不重复的 M ID，没有主次之分；Station 只有两个 M，全部展示，不为满足数量添加点位。
展示时过滤未映射或不可用点位，不以零值代替缺失值。

## 四个连接点共享产品规则

上、下、左、右四个画布连接点等效，不在产品中绑定业务角色或固定方位。
拓扑边可继续保存 sourceHandle/targetHandle 以恢复绘图位置；切换连接方位不改变业务关系和计数。

示例：Battery 的规则。

```json
{
  "connections": [
    {
      "products": ["Hybrid_Inverter", "PCS"],
      "min": 1,
      "max": 1
    }
  ],
  "description": "Connect to exactly one Hybrid Inverter or PCS across this group."
}
```

| 规则字段 | 含义 |
| --- | --- |
| products | 允许连接的具体产品 name；不按 type 分类匹配 |
| min | 发布时，该规则分组至少连接的对端设备数量 |
| max | 该规则分组最多连接的对端设备数量；null 表示没有产品级上限 |

单条 `connections` 规则只描述允许的对端产品和数量限制；不要添加 `description`。`topology.description` 是整组连接规则的业务说明，前端将其显示在该产品连接规则 tooltip 的底部。

未定义 `topology` 的产品不参与拓扑编辑器和拓扑画布；Station 与 Env 属于此类产品，仍可在设备管理和监控中使用。

### 校验语义

1. 双方产品都必须有允许对方的规则；未声明即不允许。未定义 topology 的产品不参与拓扑。
2. 四个连接点共同计数。同一条规则中不同 products 的对端数量合计，不是分别计算限额。
3. 一个对端产品只能出现在本产品的一条规则中，避免重复计数或不明确的优先级。
4. 禁止自环和同一对节点的重复边，包括从不同方位重复连线；当前库不声明同产品之间连线。
5. 新增连线时分别检查两端规则及 max。删除、换向后重新计算受影响节点的连接数。
6. min 允许在编辑和草稿阶段暂时不满足，发布时统一校验。max 按不同对端节点统计。
7. 连线无业务方向，不以 source/target 推断充放电或供电方向；运行态功率另外读取点位。
8. null 不表示实际接线、并联数量或电气容量无限制；这些仍由现场设备和配置决定。

### 当前默认规则

| 产品 | 对端产品分组 | min | max |
| --- | --- | --- | --- |
| Battery | Hybrid_Inverter、PCS 合计 | 1 | 1 |
| PV_Group | Hybrid_Inverter、AC_Inverter 合计 | 1 | 1 |
| Hybrid_Inverter | PV_Group | 0 | null |
| Hybrid_Inverter | Battery | 0 | null |
| Hybrid_Inverter | Meter | 1 | 1 |
| AC_Inverter | PV_Group | 0 | null |
| AC_Inverter | Meter | 1 | 1 |
| PCS | Battery | 0 | null |
| PCS | Meter | 1 | 1 |
| Diesel | Meter | 1 | 1 |
| 四类负载 | Meter | 1 | 1 |
| Meter | Hybrid_Inverter、AC_Inverter、PCS、Diesel、四类负载合计 | 0 | null |
| Station、Env | 不参与能量连线，保留固定信息卡 | — | — |

这些是本项目的初始配置，不是厂家通用电气限制。混合逆变器的光伏与电池连接暂不设必连组合条件。
Meter 关联表示业务拓扑，不表示把计量设备当作配电箱的实际接线端子。
每站 Meter 数量、全图连通性和环路策略属于站点级校验，不在每个产品里重复定义。

### Tooltip

先由规则自动生成“允许连接的产品、最少/最多数量、当前已连接数量”，底部附加 `topology.description`。
产品文件不保存当前数量、占用方位或实例 ID；这些由当前拓扑计算。
提示文案仅用于解释，程序校验以 products/min/max 为准。

## 消费端适配

当前文件是改造后的产品契约；本次仅修改产品库，没有修改后端、编辑器或运行环境。

- 删除 pName、canCreateInstance、topology.enabled、topology.type、components 和 connectableProducts。
- 旧 Rust 解析器仍要求部分已删除字段，新文件不能直接替换部署，需先更新解析类型和产品转换逻辑。
- 旧连接代码采用单侧声明即可连接；新规则要求双方允许并按分组共同计数，需同步实现。
- PointDef 与云端导入模型需要接收 description；边端转换时应透传，不能继续固定返回 None。
- defaultDisplayMeasureIds 是平级默认 Measure ID 数组；旧主指标/详情结构不再使用。
- Battery 的 P/M/A 保持原点位与编号，仅保留此前补充的说明；本轮不改变任何保留产品的 P/M/A。
- 旧拓扑迁移须明确处理 Distribution_Board 和其 Meter 子组件；不要仅替换产品名称便直接发布。
- 内嵌产品需要更新依赖并重新构建；外部产品目录需按后端实际加载机制刷新。

## 校验

在产品库目录执行：

```sh
node scripts/validate_products.cjs
```

检查 JSON、产品标识、规则数量范围、重复目标、互相允许、点位 ID、描述及 defaultDisplayMeasureIds 引用；并覆盖四个连接点共同计数的示例。
