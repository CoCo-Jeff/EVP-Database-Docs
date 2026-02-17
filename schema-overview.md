# EVP Database Schema Overview

## Purpose

This database supports a multi-channel e-commerce system with centralized inventory and order management. The system is channel-centric: all transactions, listings, and operations are organized around **sales channels** (Amazon, eBay, Shopify) and **supply channels** (warehouses, vendors).

## Channel Types

### Sales Channels
Represent demand sources such as Amazon, eBay, Shopify, and web storefronts where customers purchase items.

### Supply Channels
Represent supply sources such as owned warehouses, 3PLs, and external vendors from which items can be sourced or fulfilled.

## Key Design Principles

### 1. GUID Field Naming Convention
**All Guid fields ending with `Id` are primary or foreign keys.** Non-Guid fields ending with `Id` are non-key attributes with misleading naming (e.g., `ConditionId` as a string field).

### 2. Channel Isolation
Most entities are scoped to a channel via `ChannelId` or an indirect relationship through location or another channel-linked entity. This ensures data separation between different sales and supply channels.

### 3. Extension Tables (Shared-Id Pattern)
Sales and supply variants of item codes (`evp_inv_SalesItemCode`, `evp_inv_SupplyItemCode`) **share the same Id** as their parent `evp_inv_ItemCode` rather than using a foreign key. This effectively splits the logical entity across physical tables.

**Important:** When joining these extension tables, use:
```sql
FROM evp_inv_ItemCode ic
INNER JOIN evp_inv_SalesItemCode sic ON ic.Id = sic.Id
```

NOT a traditional foreign key join.

### 4. Flexible Attributes (Entity-Attribute-Value Pattern)
Item attributes are stored in an entity–attribute–value pattern to support extensible, non-rigid metadata. See `evp_inv_ItemAttribute` for implementation.

### 5. Audit Fields
All tables include standard audit fields:
- `CreatedBy` (nvarchar(100))
- `CreatedDate` (datetime2)
- `ModifiedBy` (nvarchar(100))
- `ModifiedDate` (datetime2)
- `ProcessName` (nvarchar(200)) - Integration or workflow identifier

## Core Entity Hierarchy

```
evp_cha_Instance (Channel)
  ↓
evp_cha_Location (Location within channel)
  ↓
evp_inv_ItemAvailability (Inventory at location)

evp_inv_Item (Base product - channel-agnostic)
  ↓
evp_inv_ItemCode (Channel-specific listing)
  ↓ (shared Id)
evp_inv_SalesItemCode (Sales channel extension)
evp_inv_SupplyItemCode (Supply channel extension)
  ↓
evp_inv_ItemAttribute (Flexible key-value attributes)
```

## Domain Organization

The database is organized into 37 functional domains. See the [Database Domains Reference](README.md#database-domains-reference) for the complete list.

**High-Priority Domains:**
- **Channel** (cha) - 48 tables - Channel configuration and operations
- **Data Warehouse** (dwh) - 5 tables - Reporting and analytics
- **Inventory** (inv) - 51 tables - Product catalog and stock management
- **Merchant** (mer) - 16 tables - Tenant and merchant configuration

## Data Flow

1. **Channels** are configured in `evp_cha_Instance`
2. **Locations** (warehouses, stores) are defined per channel in `evp_cha_Location`
3. **Base items** are created in `evp_inv_Item` (channel-agnostic)
4. **Item codes** are created in `evp_inv_ItemCode` for each channel+UOM combination
5. **Sales/Supply extensions** share the same Id in `evp_inv_SalesItemCode` or `evp_inv_SupplyItemCode`
6. **Inventory** is tracked per item+location in `evp_inv_ItemAvailability`
7. **Orders** flow through the Orders domain, linking to channels, items, and locations

## Common Patterns

### Item Code is the Listing Unit
**Item codes**, not base items, are what get listed, priced, and sold in a channel. Always join through item codes when looking at channel-specific data.

### Location Contains Channel
All locations belong to exactly one channel. Use `LocationId → ChannelId` to navigate to the owning channel if needed.

### Channel Context is Paramount
Always consider whether you are working with a sales or supply channel and filter queries accordingly using `ChannelTypeId` or explicit channel references.

## Navigation

- For SQL querying patterns, see [SQL Guidelines](sql-guidelines.md)
- For domain-specific documentation, see individual domain files (e.g., [Channel Domain](domain-channel.md), [Inventory Domain](domain-inventory.md))
- For complete schema reference, see `schema-all-fields.json`