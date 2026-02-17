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
- **Documentation Status**: 9 detailed, 42 pending
- **Related Domains**: Channel, Orders, Catalog Sources

---

## Table Index

The Inventory domain contains 51 tables organized by function:

### Core Item Management ⭐ Documented in Detail
1. evp_inv_Item - Base product master
2. evp_inv_ItemCode - Channel-specific listings
3. evp_inv_SalesItemCode - Sales channel extension
4. evp_inv_SupplyItemCode - Supply channel extension
5. evp_inv_ItemAttribute - Flexible EAV attributes
6. evp_inv_ItemAvailability - Location-based inventory
7. evp_inv_ItemIdentifier - UPC/GTIN/ISBN/EAN codes
8. evp_inv_Manufacturer - Manufacturer reference
9. evp_inv_ItemUom - Unit of measure definitions

### Item Identifiers & Classification (Pending)
10. evp_inv_ItemCodeHistory
11. evp_inv_ItemImage
12. evp_inv_ItemNote
13. evp_inv_ItemPriceHistory
14. evp_inv_ItemTag
15. evp_inv_ManufacturerImage

### Kits & Bundles (Pending)
16. evp_inv_KitComponent
17. evp_inv_KitComponentMapping

### Purchase Orders & Procurement (Pending)
18. evp_inv_PurchaseOrder
19. evp_inv_PurchaseOrderLine
20. evp_inv_PurchaseOrderLineReceiving
21. evp_inv_PurchaseOrderNote
22. evp_inv_PurchaseOrderTerms

### Inventory Movements & Adjustments (Pending)
23. evp_inv_InventoryAllocation
24. evp_inv_InventoryMovement
25. evp_inv_InventoryMovementLine
26. evp_inv_InventoryReceiving
27. evp_inv_InventoryReceivingLine
28. evp_inv_InventoryReservation

### Inventory Holds & Tracking (Pending)
29. evp_inv_AvailabilityHold
30. evp_inv_AvailabilityHoldCode
31. evp_inv_LotControl
32. evp_inv_SerialNumber

### Repricing & Market Data (Pending)
33. evp_inv_CompetitorPrice
34. evp_inv_CompetitorProductMatch
35. evp_inv_DataCollectionValue
36. evp_inv_MarketData
37. evp_inv_RepriceRule
38. evp_inv_RepriceRuleBand
39. evp_inv_RepriceRuleLevel

### Planning & Demand (Pending)
40. evp_inv_DemandForecast
41. evp_inv_ItemPlanning
42. evp_inv_PlanningGroup
43. evp_inv_Replenishment
44. evp_inv_ReplenishmentLine

### Configuration & Reference (Pending)
45. evp_inv_CategorySchema
46. evp_inv_ItemGroup
47. evp_inv_ItemProfile
48. evp_inv_ItemProfileItem
49. evp_inv_PricingRule
50. evp_inv_PurchaseOrderStatus
51. evp_inv_QualityCheck
52. evp_inv_UomConversion

---

## Detailed Table Documentation

### Core Item Tables

**evp_inv_Item** - Channel-agnostic product master records. The root entity containing SKU, title, manufacturer, and base product attributes.

**evp_inv_ItemCode** - Channel-specific listings with UOM variants. Bridge between items and channels, parameterized by unit of measure.

**evp_inv_SalesItemCode** - Sales channel extension (shared Id pattern). Contains pricing, replenishment, and demand metrics for sales channels.

**evp_inv_SupplyItemCode** - Supply channel extension (shared Id pattern). Contains fulfillment configuration for supply channels.

**evp_inv_ItemAttribute** - Flexible EAV attributes. Stores extensible key-value pairs for item codes (Color, Size, ASIN, etc.).

**evp_inv_ItemAvailability** - Location-based inventory tracking. On-hand and allocated quantities per item per location.

### Item Identifiers

**evp_inv_ItemIdentifier** - Stores alternate product identifiers (UPC, GTIN, ISBN, EAN). Item-level (not channel-specific). Items commonly have multiple identifiers with one flagged as primary. Value stored as text to preserve leading zeros.

*Key Query Pattern:*
```sql
-- Get primary identifier for item
SELECT IdentifierType, Value
FROM evp_inv_ItemIdentifier WITH (NOLOCK)
WHERE ItemId = @ItemId AND Primary = 1;
```

**⚠️ Important:** UOM field exists in this table but is unused - never reference or join on it.

### Manufacturer Reference

**evp_inv_Manufacturer** - Master list of product manufacturers and brands. Referenced by evp_inv_Item.ManufacturerId. Primary fields are Name and Abbr. Description, ParentId, Verified, and Brand fields exist but are rarely used.

### Unit of Measure

**evp_inv_ItemUom** - Defines units of measure (EACH, PACK12, CASE, etc.) with quantities, weights, and dimensions. Code is the primary identifier (e.g., "EACH", "PACK12"). Quantity represents the multiplier (EACH=1, PACK12=12). No automatic conversion - ItemCodes are created at specific UOM levels. Suppliers may sell in PACK12 while sales channels sell in EACH. Stores "best quality" weights and dimensions aggregated from multiple sources.

*Key Concept:* An item may have multiple UOMs. Supply channels and sales channels can use different UOMs for the same item.

---

## Additional Tables

The remaining 42 tables in the Inventory domain are pending detailed documentation. These tables support:
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