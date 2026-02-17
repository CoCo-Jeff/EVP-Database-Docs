# Inventory Domain Documentation

## Domain Overview

The **Inventory** domain represents the product catalog and stock management system. It contains both channel-agnostic product definitions and channel-specific listing configurations, enabling multi-channel e-commerce with centralized inventory control.

**Key Concepts:**
- **Base Items**: Channel-agnostic product master records
- **Item Codes**: Channel-specific listings with UOM variants
- **Extension Tables**: Sales and Supply configurations share IDs with ItemCode
- **EAV Attributes**: Flexible key-value metadata for extensibility
- **Location-Based Availability**: Inventory tracked per item per location

**Domain Statistics:**
- **Abbreviation**: `inv`
- **Importance**: **High Priority**
- **Table Count**: 51 tables
- **Documentation Status**: 6 detailed, 45 pending
- **Related Domains**: Channel, Orders, Catalog Sources

---

## Table Index

The Inventory domain contains 51 tables organized by function:

### Core Item Management ⭐ Documented in Detail
1. evp_inv_Item
2. evp_inv_ItemCode
3. evp_inv_SalesItemCode
4. evp_inv_SupplyItemCode
5. evp_inv_ItemAttribute
6. evp_inv_ItemAvailability

### Item Identifiers & Classification
7. evp_inv_ItemCodeHistory
8. evp_inv_ItemIdentifier
9. evp_inv_ItemImage
10. evp_inv_ItemNote
11. evp_inv_ItemPriceHistory
12. evp_inv_ItemTag
13. evp_inv_Manufacturer
14. evp_inv_ManufacturerImage

### Kits & Bundles
15. evp_inv_KitComponent
16. evp_inv_KitComponentMapping

### Purchase Orders & Procurement
17. evp_inv_PurchaseOrder
18. evp_inv_PurchaseOrderLine
19. evp_inv_PurchaseOrderLineReceiving
20. evp_inv_PurchaseOrderNote
21. evp_inv_PurchaseOrderTerms

### Inventory Movements & Adjustments
22. evp_inv_InventoryAllocation
23. evp_inv_InventoryMovement
24. evp_inv_InventoryMovementLine
25. evp_inv_InventoryReceiving
26. evp_inv_InventoryReceivingLine
27. evp_inv_InventoryReservation

### Inventory Holds & Tracking
28. evp_inv_AvailabilityHold
29. evp_inv_AvailabilityHoldCode
30. evp_inv_LotControl
31. evp_inv_SerialNumber

### Repricing & Market Data
32. evp_inv_CompetitorPrice
33. evp_inv_CompetitorProductMatch
34. evp_inv_DataCollectionValue
35. evp_inv_MarketData
36. evp_inv_RepriceRule
37. evp_inv_RepriceRuleBand
38. evp_inv_RepriceRuleLevel

### Planning & Demand
39. evp_inv_DemandForecast
40. evp_inv_ItemPlanning
41. evp_inv_PlanningGroup
42. evp_inv_Replenishment
43. evp_inv_ReplenishmentLine

### Configuration & Reference
44. evp_inv_CategorySchema
45. evp_inv_ItemGroup
46. evp_inv_ItemProfile
47. evp_inv_ItemProfileItem
48. evp_inv_PricingRule
49. evp_inv_PurchaseOrderStatus
50. evp_inv_QualityCheck
51. evp_inv_UomConversion

---

## Detailed Table Documentation

*Full detailed documentation for the 6 core tables follows the standardized template with Purpose, Logical Role, Primary Keys, Foreign Keys, Key Attributes, Relationships, Business Rules, Query Patterns, and Implementation Notes.*

### Core Tables
- **evp_inv_Item**: Channel-agnostic product master records
- **evp_inv_ItemCode**: Channel-specific listings with UOM variants
- **evp_inv_SalesItemCode**: Sales channel extension (shared Id pattern)
- **evp_inv_SupplyItemCode**: Supply channel extension (shared Id pattern)
- **evp_inv_ItemAttribute**: Flexible EAV attributes for item codes
- **evp_inv_ItemAvailability**: Location-based inventory tracking

---

## Additional Tables

The remaining 45 tables in the Inventory domain are pending detailed documentation. These tables support:
- Purchase order management and receiving
- Inventory movements and adjustments
- Lot control and serial number tracking
- Repricing rules and market data
- Demand forecasting and planning
- Kit/bundle management

To contribute documentation for additional tables, follow the [Per-Table Template](README.md#per-table-markdown-template) in the README.

---

## Key Patterns

### Extension Table Pattern
`evp_inv_SalesItemCode` and `evp_inv_SupplyItemCode` **share the same Id** as `evp_inv_ItemCode`. This is NOT a foreign key relationship:

```sql
-- CORRECT: Join on shared Id
FROM evp_inv_ItemCode ic
INNER JOIN evp_inv_SalesItemCode sic ON ic.Id = sic.Id

-- WRONG: Treating Id as FK
FROM evp_inv_ItemCode ic  
INNER JOIN evp_inv_SalesItemCode sic ON ic.Id = sic.ItemCodeId  -- No such column!
```

### Entity-Attribute-Value Pattern
`evp_inv_ItemAttribute` uses EAV for flexible metadata without schema changes:

```sql
-- Find items with specific attribute
SELECT ic.Code, attr.Value AS Color
FROM evp_inv_ItemCode ic
INNER JOIN evp_inv_ItemAttribute attr ON ic.Id = attr.ItemCodeId
WHERE attr.AttributeCode = 'Color'
  AND attr.Value = 'Blue';
```

### Available-to-Sell Calculation
```sql
SELECT 
    Quantity - Allocated AS AvailableToSell
FROM evp_inv_ItemAvailability
WHERE ItemId = @ItemId AND LocationId = @LocationId;
```

---

## See Also

- [Schema Overview](schema-overview.md) - Design principles and patterns
- [SQL Guidelines](sql-guidelines.md) - Querying patterns for inventory
- [Channel Domain](domain-channel.md) - Channel configuration and locations
- [Orders Domain](domain-orders.md) - Order processing with inventory