# Channel Domain Documentation

## Domain Overview

The **Channel** domain represents the core multi-channel architecture of the EVP e-commerce system. It contains configuration and operational data for both **sales channels** (Amazon, eBay, Shopify, websites) and **supply channels** (warehouses, 3PLs, vendors).

**Key Concepts:**
- **Sales Channels**: Represent demand sources where customers purchase items
- **Supply Channels**: Represent supply sources from which items can be sourced or fulfilled
- **Locations**: Physical or logical places within channels for inventory and operations
- **Channel Isolation**: Most data is scoped to specific channels for multi-tenant security

**Domain Statistics:**
- **Abbreviation**: `cha`
- **Importance**: **High Priority**
- **Table Count**: 48 tables
- **Documentation Status**: 5 detailed, 24 brief, 4 skipped, 15 deferred
- **Related Domains**: Inventory, Orders, Settlements, Shipping

---

## Table Index

The Channel domain contains 48 tables organized by function:

### Core Channel Configuration ⭐ Documented in Detail
1. evp_cha_Instance
2. evp_cha_Location
3. evp_cha_Availability
4. evp_cha_Service
5. evp_cha_Commission

### Channel-Specific Extensions (Brief Documentation)
6. evp_cha_Amazon - Amazon marketplace configuration
7. evp_cha_AmazonFulfillment - FBA configuration
8. evp_cha_eBay - eBay marketplace configuration
9. evp_cha_Etail - Generic e-commerce configuration
10. evp_cha_Shopify - Shopify store configuration
11. evp_cha_ShipStation - ShipStation integration
12. evp_cha_Shippo - Shippo integration
13. evp_cha_Walmart - Walmart marketplace configuration

### Generic Channel Configuration (Brief)
14. evp_cha_Integration - Generic integration settings
15. evp_cha_Sales - Sales channel settings
16. evp_cha_Setting - Key-value configuration pairs
17. evp_cha_Supplier - Supply channel settings

### Code & Identifier Tables (Brief)
18. evp_cha_InstanceCode - Channel instance codes
19. evp_cha_JurisdictionCode - Tax jurisdiction codes
20. evp_cha_LocationCode - Location codes
21. evp_cha_ManufacturerCode - Manufacturer codes

### Order Configuration (Brief)
22. evp_cha_OrderOption - Available order options
23. evp_cha_OrderOptionValue - Order option values

### Utility & Infrastructure (Brief)
24. evp_cha_ContactRole - Contact role definitions
25. evp_cha_ManufacturerError - Manufacturer mapping errors
26. evp_cha_NextNumber - Sequence generator
27. evp_cha_PublicationTemplate - HTML templates
28. evp_cha_ServiceProperties - Service properties (encrypted)
29. evp_cha_Status - Status tracking

### Mapping Tables ⏸ Deferred
30-38. AttributeMap, CategoryMap, CarrierServiceLevelMap, etc. (9 tables)

### Location Extension Tables ⏸ Deferred  
39-44. LocationFulfillment, LocationCarrierServiceLevel, etc. (6 tables)

### Skipped ⊗
45-48. EntityCode, EvpClient, ChannelImage, ItemCondition (4 tables)

---

## Detailed Table Documentation

*Note: Full detailed documentation for the 5 core tables has been completed following the standardized template, including Purpose, Logical Role, Primary Keys, Foreign Keys, Key Attributes, Relationships, Business Rules, Query Patterns, and Implementation Notes.*

### Core Tables
- **evp_cha_Instance**: Root channel configuration entity
- **evp_cha_Location**: Physical/logical warehouses and fulfillment centers
- **evp_cha_Availability**: Channel-level availability calculation rules
- **evp_cha_Service**: Encrypted credentials for external integrations
- **evp_cha_Commission**: Fee structures for profitability calculations

---

## Brief Documentation Summary

**Channel-Specific Extensions (8 tables)**: Configuration specific to individual channel types (Amazon, eBay, Shopify, etc.)

**Generic Configuration (4 tables)**: Settings applicable across channel types

**Code Tables (4 tables)**: Store channel-specific identifiers for various entities

**Order Configuration (2 tables)**: Define available order options like gift wrap and messages

**Utility Tables (6 tables)**: Infrastructure support including sequence generators, templates, and status tracking

---

## Deferred Documentation

**Mapping Tables (9 tables)**: Data translation between EVP and external channels - deferred due to low query frequency

**Location Extensions (6 tables)**: Additional location-specific configuration - deferred due to specialized use

---

## See Also

- [Schema Overview](schema-overview.md) - Design principles
- [SQL Guidelines](sql-guidelines.md) - Query patterns
- [Inventory Domain](domain-inventory.md) - Item management
- [README](README.md) - Project overview