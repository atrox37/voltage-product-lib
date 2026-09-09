# Product Library Change Log

Version identifiers use the release date in `YYYY-MM-DD` format. New entries are added at the top and describe changes relative to the preceding recorded version.

## 2026-09-08

### Change statistics

| Item | Count |
| --- | ---: |
| Current product definitions | 13 |
| New product definitions | 1 (`Meter`) |
| Removed product definitions | 5 |
| Updated retained product definitions | 12 |
| Updated documentation files | 1 |

### Product catalog

- Added `Meter` as an independent product for topology metering and electrical aggregation.
- Removed obsolete placeholder or container definitions: `ESS`, `Load`, `Generator`, `Inverter`, and `Distribution_Board`.
- Removed product-level `pName` and `canCreateInstance` fields.
- Removed topology `enabled`, `type`, `components`, and `connectableProducts` fields.
- Standardized all retained topology products as independent device nodes; `Hybrid_Inverter` is represented as one device instead of a Composite with internal nodes.
- Added product `type`, product descriptions, and flat default `display` measurement lists.
- Kept the existing Battery P/M/A point IDs and definitions; this release only adds description metadata.

### Field changes

| Scope | Field | Change | Notes |
| --- | --- | --- | --- |
| Product | `type` | Added | Product classification used by the gateway and topology library. |
| Product | `description` | Added or standardized | English product description without unit text. |
| Product | `pName` | Removed | The stable product identifier is `name`. |
| Product | `canCreateInstance` | Removed | All retained device products use the same standalone-node model. |
| Product | `display` | Changed | Flat list of 3–5 Measure IDs; no primary/detail hierarchy. |
| Topology | `topology.image` | Retained | Product image resource reference. |
| Topology | `topology.connections` | Added | Node-wide connection-rule groups. |
| Topology | `topology.description` | Added | English business explanation displayed below the connection-rule tooltip. |
| Topology | `topology.enabled` | Removed | Presence of `topology` now determines whether a product participates in the topology library. |
| Topology | `topology.type` | Removed | Standalone/composite/container categorization is no longer stored in products. |
| Topology | `topology.components` | Removed | Hybrid Inverter no longer creates internal component nodes. |
| Topology | `topology.connectableProducts` | Removed | Replaced by reciprocal connection groups. |
| Connection rule | `products` | Added | Explicit list of permitted peer product names for the current node. |
| Connection rule | `min` | Added | Minimum number of connected peer instances across the whole product group. |
| Connection rule | `max` | Added | Maximum number of connected peer instances across the whole product group; `null` means unlimited. |
| Connection rule | rule-level `description` | Removed | Tooltip explanation belongs only to `topology.description`. |
| Point | `description` | Added or standardized | English point description without unit text; units remain in `unit`. |

### Connection rules

- Replaced legacy one-sided product connection lists with reciprocal `topology.connections` groups using `products`, `min`, and `max`.
- Defined connection constraints at the whole-node level. The four canvas handles remain visually selectable but are business-equivalent.
- Added one topology-level connection description per participating product for the connection-rule tooltip.
- Added Meter as the electrical aggregation node. Its rule allows the supported source and load products without imposing a Meter-side connection count.
- Removed `topology` from `Station` and `Env`; they are excluded from the product drag library and energy connection graph.

### Point metadata

- Added descriptions to product, topology, Property, Measure, and Action definitions.
- Standardized every JSON `description` as English text without unit text. Units remain exclusively in the `unit` field.
- Simplified Hybrid Inverter measurements and actions around PV conversion, PCS functions, battery status, and Start/Stop status.

### Documentation

- Reworked `README.md` to document the simplified product model, node-wide connection-group semantics, tooltip behavior, and display-point rules.

## Entry format

For each subsequent version, add a new `## YYYY-MM-DD` section above this entry with:

- a compact change-statistics table;
- added, removed, and changed product definitions;
- connection-rule or topology behavior changes;
- point-definition and metadata changes;
- affected documentation or validation changes.
