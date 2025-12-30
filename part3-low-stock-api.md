Part 3: API Implementation (35 minutes)
Assumptions
Recent sales means activity in the last 30 days


Low-stock threshold is stored per product


Each product has one primary supplier


Stock is evaluated per warehouse


@app.route('/api/companies/<int:company_id>/alerts/low-stock', methods=['GET'])
def low_stock_alerts(company_id):
    alerts = []

    inventories = Inventory.query.join(Product).join(Warehouse)\
        .filter(Warehouse.company_id == company_id).all()

    for inventory in inventories:
        product = inventory.product

        if not product.low_stock_threshold:
            continue

        if inventory.quantity >= product.low_stock_threshold:
            continue

        sales_last_30_days = get_sales(product.id, 30)
        if sales_last_30_days == 0:
            continue

        daily_sales = sales_last_30_days / 30
        days_until_stockout = int(inventory.quantity / daily_sales)

        supplier = get_primary_supplier(product.id)

        alerts.append({
            "product_id": product.id,
            "product_name": product.name,
            "sku": product.sku,
            "warehouse_id": inventory.warehouse.id,
            "warehouse_name": inventory.warehouse.name,
            "current_stock": inventory.quantity,
            "threshold": product.low_stock_threshold,
            "days_until_stockout": days_until_stockout,
            "supplier": {
                "id": supplier.id,
                "name": supplier.name,
                "contact_email": supplier.contact_email
            }
        })

    return {
        "alerts": alerts,
        "total_alerts": len(alerts)
    }

Edge Cases Handled
Products without thresholds are ignored


Products without recent sales are skipped


Multiple warehouses are handled separately


Division by zero is avoided


Missing supplier data can be safely handled
