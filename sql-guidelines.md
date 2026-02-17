# SQL Guidelines for EVP Database

## Overview

This document provides SQL querying patterns, conventions, and best practices specific to the EVP database schema.

## SQL Server Syntax Requirements

The EVP database runs on **SQL Server**. All queries must use SQL Server-specific syntax and semantics.

### Critical Syntax Rules

1. **NOLOCK Hints**
   - Use `WITH (NOLOCK)` directly after table names
   - Correct: `FROM MyTable t WITH (NOLOCK)`
   - Never start a query with `WITH` (avoids CTE misparsing)

2. **JOIN Clause Ordering**
   - All `JOIN` clauses must appear **before** `WHERE`
   - Never insert joins between `WHERE` predicates

3. **Subquery Aliases**
   - Always provide explicit aliases for subqueries
   - Correct: `FROM (SELECT ...) AS subquery_alias`

4. **LEFT JOIN Filtering**
   - When using `LEFT JOIN` but requiring a match, filter with `IS NOT NULL` in `WHERE`
   - Example: `LEFT JOIN Table2 t2 ... WHERE t2.Id IS NOT NULL`

5. **Reserved Words**
   - Avoid using reserved words as identifiers
   - If unavoidable, wrap them in square brackets: `[Order]`, `[User]`

## Common Query Patterns

### Get All Item Codes for a Channel

```sql
-- Get all item codes listed on a specific channel
SELECT 
    ic.Id,
    ic.Code AS ChannelSku,
    ic.Status,
    ic.ListingState,
    i.Sku AS MasterSku,
    i.Title
FROM evp_inv_ItemCode ic WITH (NOLOCK)
INNER JOIN evp_inv_Item i WITH (NOLOCK) 
    ON ic.ItemId = i.Id
WHERE ic.ChannelId = @ChannelId
    AND ic.Status = 'Active'
ORDER BY ic.Code;
```

### Get Sales-Specific Details for Channel Item Codes

```sql
-- Get sales channel configuration and pricing
SELECT 
    ic.Code AS ChannelSku,
    sic.Price,
    sic.FulfillmentMethod,
    sic.ReplenishmentMode,
    sic.MinimumStockLevel,
    sic.MaximumStockLevel,
    sic.LastPublishPrice,
    sic.LastSaleTime
FROM evp_inv_ItemCode ic WITH (NOLOCK)
INNER JOIN evp_inv_SalesItemCode sic WITH (NOLOCK) 
    ON ic.Id = sic.Id  -- Shared Id pattern, not FK!
WHERE ic.ChannelId = @SalesChannelId
    AND sic.PlanningEnabled = 1;
```

### Get Supply-Specific Details for Supplier/Warehouse

```sql
-- Get supply channel fulfillment configuration
SELECT 
    ic.Code AS SupplierSku,
    suic.FulfillmentMethod,
    suic.FulfillmentLatency,
    suic.MinimumOrderQuantity,
    suic.MaximumOrderQuantity
FROM evp_inv_ItemCode ic WITH (NOLOCK)
INNER JOIN evp_inv_SupplyItemCode suic WITH (NOLOCK) 
    ON ic.Id = suic.Id  -- Shared Id pattern
WHERE ic.ChannelId = @SupplyChannelId;
```

### Get Inventory at All Locations for an Item

```sql
-- Get current inventory position across all locations
SELECT 
    ia.Quantity,
    ia.Allocated,
    ia.AverageCost,
    ia.FulfillmentLatency,
    l.Name AS LocationName,
    l.Code AS LocationCode,
    c.Name AS ChannelName,
    i.Sku AS MasterSku,
    i.Title
FROM evp_inv_ItemAvailability ia WITH (NOLOCK)
INNER JOIN evp_cha_Location l WITH (NOLOCK) 
    ON ia.LocationId = l.Id
INNER JOIN evp_cha_Instance c WITH (NOLOCK) 
    ON l.ChannelId = c.Id
INNER JOIN evp_inv_Item i WITH (NOLOCK) 
    ON ia.ItemId = i.Id
WHERE ia.ItemId = @ItemId
    AND ia.Quantity > 0
ORDER BY l.Priority, ia.Quantity DESC;
```

### Get Attributes for an Item Code

```sql
-- Get all flexible attributes for a channel item code
SELECT 
    ia.AttributeCode,
    ia.Value,
    ia.LanguageId,
    ia.Sequence
FROM evp_inv_ItemAttribute ia WITH (NOLOCK)
WHERE ia.ItemCodeId = @ItemCodeId
ORDER BY ia.Sequence, ia.AttributeCode;
```

### Get Item with Specific Attribute Value

```sql
-- Find items with a specific attribute (e.g., ASIN)
SELECT 
    ic.Code AS ChannelSku,
    i.Sku AS MasterSku,
    i.Title,
    asinAttr.Value AS ASIN
FROM evp_inv_ItemCode ic WITH (NOLOCK)
INNER JOIN evp_inv_Item i WITH (NOLOCK) 
    ON ic.ItemId = i.Id
INNER JOIN evp_inv_ItemAttribute asinAttr WITH (NOLOCK) 
    ON ic.Id = asinAttr.ItemCodeId
WHERE ic.ChannelId = @ChannelId
    AND asinAttr.AttributeCode = 'ASIN'
    AND asinAttr.Value = @AsinValue;
```

### Get Available-to-Sell Quantity

```sql
-- Calculate available-to-sell (on-hand minus allocated)
SELECT 
    i.Sku,
    l.Name AS Location,
    ia.Quantity AS OnHand,
    ia.Allocated,
    (ia.Quantity - ia.Allocated) AS AvailableToSell,
    ia.FulfillmentLatency
FROM evp_inv_ItemAvailability ia WITH (NOLOCK)
INNER JOIN evp_inv_Item i WITH (NOLOCK) 
    ON ia.ItemId = i.Id
INNER JOIN evp_cha_Location l WITH (NOLOCK) 
    ON ia.LocationId = l.Id
WHERE (ia.Quantity - ia.Allocated) > 0
    AND l.AllowSalesFrom = 1
ORDER BY i.Sku, l.Priority;
```

## Aliasing Conventions

### Channel SKUs
Alias the `Code` field from `evp_inv_ItemCode` as `ChannelSku` or include the channel name:
- `ic.Code AS ChannelSku`
- `ic.Code AS AmazonSku`
- `ic.Code AS EbaySku`

### Locations
Prefix references to location-specific IDs with the role:
- `FulfillmentLocationId`
- `ReturnLocationId`
- `ShipFromId`
- `ShipToId`

### Channels
Clearly differentiate between sales and supply channel contexts:
- `@SalesChannelId`
- `@SupplyChannelId`
- `c.Name AS SalesChannelName`

### Table Aliases
Use short, readable aliases:
- `ic` for `evp_inv_ItemCode`
- `i` for `evp_inv_Item`
- `sic` for `evp_inv_SalesItemCode`
- `suic` for `evp_inv_SupplyItemCode`
- `ia` for `evp_inv_ItemAvailability`
- `l` for `evp_cha_Location`
- `c` for `evp_cha_Instance`

## Query Documentation Standards

### Always Include Comments

1. **Block comment at top** describing overall query purpose
2. **Inline comments** for:
   - Key joins (especially channel IDs, location IDs)
   - Business rules and filters
   - Complex calculations
   - Non-obvious logic

## Common Pitfalls

### ❌ Don't: Use traditional FK join for extension tables
```sql
-- WRONG - This treats shared Id as a foreign key
FROM evp_inv_ItemCode ic
INNER JOIN evp_inv_SalesItemCode sic ON ic.Id = sic.ItemCodeId  -- No such column!
```

### ✓ Do: Use shared Id pattern
```sql
-- CORRECT - Extension tables share the same Id
FROM evp_inv_ItemCode ic
INNER JOIN evp_inv_SalesItemCode sic ON ic.Id = sic.Id
```

### ❌ Don't: Forget channel context
```sql
-- WRONG - Missing channel filter
SELECT * FROM evp_inv_ItemCode WHERE Status = 'Active';
```

### ✓ Do: Always include channel context
```sql
-- CORRECT - Filtered to specific channel
SELECT * FROM evp_inv_ItemCode 
WHERE ChannelId = @ChannelId AND Status = 'Active';
```

## Additional Resources

- [Schema Overview](schema-overview.md) - High-level design principles
- [Channel Domain](domain-channel.md) - Channel-specific tables and patterns
- [Inventory Domain](domain-inventory.md) - Inventory-specific tables and patterns