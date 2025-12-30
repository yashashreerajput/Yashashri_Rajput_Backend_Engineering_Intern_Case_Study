Part 2: Database Design (25 minutes)
Schema Design
companies
id


name


created_at


warehouses
id


company_id


name


location


products
id


company_id


name


sku (unique)


price


is_bundle


inventory
id


product_id


warehouse_id


quantity


updated_at


inventory_history
id


inventory_id


change_amount


created_at


suppliers
id


name


contact_email


product_suppliers
product_id


supplier_id


product_bundles
bundle_product_id


child_product_id


quantity



Missing Requirements / Questions
Can a product have multiple suppliers or only one?


Should low stock be calculated per warehouse or per company?


How recent does “recent sales activity” mean?


Should bundle sales automatically reduce child product inventory?


Is historical inventory data required indefinitely?



Design Decisions Explained
Inventory is separated from products to support multiple warehouses


Inventory history allows tracking stock changes and audits


Unique constraints prevent duplicate stock records


Bundle table supports composite products without duplication


Indexes on sku, product_id, and warehouse_id improve query speed
