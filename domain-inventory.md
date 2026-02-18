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
- **Documentation Status**: 17 detailed, 34 pending
- **Related Domains**: Channel, Orders, Catalog Sources, Media

---

## Table Index

The Inventory domain contains 51 tables organized by function:

### Core Item Management ⭐ Documented in Detail
1. evp_inv_Item - Base product master
2. evp_inv_ItemCode - Channel-specific listings
3. evp_inv_SalesItemCode - Sales channel extension (shared Id)
4. evp_inv_SupplyItemCode - Supply channel extension (shared Id)
5. evp_inv_ItemAttribute - Flexible EAV attributes
6. evp_inv_ItemAvailability - Location-based inventory
7. evp_inv_ItemIdentifier - UPC/GTIN/ISBN/EAN codes
8. evp_inv_Manufacturer - Manufacturer/brand reference
9. evp_inv_ItemUom - Unit of measure definitions
10. evp_inv_ItemCategory - Category assignment (loose taxonomy)
11. evp_inv_ItemStatus - Arbitrary status tracking for Items
12. evp_inv_ItemCodeStatus - Arbitrary status tracking for ItemCodes
13. evp_inv_ItemRestriction - Channel selling restrictions
14. evp_inv_ItemChannelImage - DEPRECATED (use evp_med_ItemChannelImage)
15. evp_inv_ItemPhysicalLocation - Warehouse bin/shelf assignments
16. evp_inv_Lot - Lot/batch tracking
17. evp_inv_ItemPrice - Authoritative price/cost source

### Item Identifiers & Classification (Pending)
18. evp_inv_ItemCodeHistory
19. evp_inv_ItemImage
20. evp_inv_ItemNote
21. evp_inv_ItemPriceHistory
22. evp_inv_ItemTag
23. evp_inv_ManufacturerImage

### Kits & Bundles (Pending)
24. evp_inv_KitComponent
25. evp_inv_KitComponentMapping

### Purchase Orders (Pending - 5 tables)
26-30. PurchaseOrder, PurchaseOrderLine, etc.

### Inventory Operations (Pending - 7 tables)
31-37. InventoryMovement, InventoryReceiving, etc.

### Tracking & Quality (Pending - 4 tables)
38-41. AvailabilityHold, LotControl, SerialNumber, QualityCheck

### Repricing & Market Data (Pending - 7 tables)
42-48. CompetitorPrice, RepriceRule, MarketData, etc.

### Planning & Demand (Pending - 5 tables)
49-53. DemandForecast, Replenishment, ItemPlanning, etc.

---

## Key Tables Summary

### Pricing & Costs
**evp_inv_ItemPrice** - **CRITICAL**: Use this for current prices, NOT SalesItemCode.Price. Current record has `End IS NULL`. PriceTypeId values: "Price", "Cost", "MSRP", "UPP".

### Status Tracking
**evp_inv_ItemStatus / ItemCodeStatus** - Arbitrary Type/Status combinations. No standardization of timestamp usage. Query existing data to discover Types. Common pattern: backdate timestamps to force re-evaluation.

### Category Management  
**evp_inv_ItemCategory** - Links ItemCodes (not Items) to categories. CategoryCode is text with loose connection to evp_cla_Category. **IGNORE IsPrimary** - items have only one category.

### Channel Restrictions
**evp_inv_ItemRestriction** - Prevents ItemCode creation on specific sales channels. ItemId + ChannelId composite key.

### Identifiers
**evp_inv_ItemIdentifier** - UPC/GTIN/ISBN/EAN at item level. Filter by `Primary = 1` for main identifier. Value is text (preserves leading zeros). **WARNING**: UOM field exists but is unused.

### Unit of Measure
**evp_inv_ItemUom** - No automatic conversion. ItemCodes created at specific UOM levels. Suppliers may use PACK12, sales use EACH.

### Physical Locations
**evp_inv_ItemPhysicalLocation** - Warehouse bin/shelf assignments. Items can have multiple locations. References evp_inv_PhysicalLocation.

### Deprecated
**evp_inv_ItemChannelImage** - Use evp_med_ItemChannelImage instead.

---

## Common Query Patterns

### Get Current Selling Price (CORRECT WAY)
```sql
-- Use ItemPrice table, NOT SalesItemCode.Price
SELECT 
    ic.Code AS ChannelSku,
    ip.Amount AS CurrentPrice
FROM evp_inv_ItemCode ic WITH (NOLOCK)
INNER JOIN evp_inv_ItemPrice ip WITH (NOLOCK)
    ON ic.Id = ip.ItemCodeId
WHERE ic.ChannelId = @ChannelId
    AND ip.PriceTypeId = 'Price'
    AND ip.End IS NULL;  -- Current price
```

### Get Primary Item Identifier
```sql
SELECT 
    i.Sku,
    ii.IdentifierType,
    ii.Value AS PrimaryIdentifier
FROM evp_inv_Item i WITH (NOLOCK)
INNER JOIN evp_inv_ItemIdentifier ii WITH (NOLOCK)
    ON i.Id = ii.ItemId
WHERE ii.Primary = 1;
```

### Check Channel Restrictions
```sql
SELECT 
    i.Sku,
    c.Name AS RestrictedFromChannel
FROM evp_inv_ItemRestriction r WITH (NOLOCK)
INNER JOIN evp_inv_Item i WITH (NOLOCK) ON r.ItemId = i.Id
INNER JOIN evp_cha_Instance c WITH (NOLOCK) ON r.ChannelId = c.Id
WHERE r.ItemId = @ItemId;
```

### Discover Available Status Types
```sql
SELECT DISTINCT Type, Status, COUNT(*) AS Count
FROM evp_inv_ItemStatus WITH (NOLOCK)
GROUP BY Type, Status
ORDER BY Type, Status;
```

---

## Critical Gotchas

### ❌ Don't Use SalesItemCode.Price
Use `evp_inv_ItemPrice` where `PriceTypeId = 'Price'` and `End IS NULL` instead.

### ❌ Don't Trust IsPrimary in ItemCategory  
Ignore this field - items only have one category despite the flag.

### ❌ Don't Reference ItemIdentifier.UOM
This field exists but is unused - never join on it.

### ⚠️ Status Tables Have No Standardization
Query to discover Type/Status combinations. Timestamp field usage varies by Type.

---

## See Also

- [Schema Overview](schema-overview.md) - Design principles
- [SQL Guidelines](sql-guidelines.md) - Query patterns  
- [Channel Domain](domain-channel.md) - Channel configuration
- [Orders Domain](domain-orders.md) - Order processing