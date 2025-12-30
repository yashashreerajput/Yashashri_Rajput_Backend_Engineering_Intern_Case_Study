Part 1: Code Review & Debugging (30 minutes)
   Issues 
The product is directly tied to a warehouse


SKU uniqueness is not enforced


No validation for required or optional fields


Price is handled without considering decimal precision


Two separate database commits are used


No transaction or rollback handling


Assumes inventory is always created


Does not verify warehouse existence


No handling for duplicate inventory records
Impact in Production
Products cannot exist across multiple warehouses, which breaks a core business requirement


Duplicate SKUs can cause reporting, billing, and inventory mismatches


Missing or invalid fields can crash the API


Floating-point price issues can cause incorrect pricing


Partial data can be saved if the first commit succeeds and the second fails


Duplicate inventory rows can lead to incorrect stock counts


System becomes harder to scale as warehouse count grows
Correct code
@app.route('/api/products', methods=['POST'])
def create_product():
    data = request.json

    if 'name' not in data or 'sku' not in data or 'price' not in data:
        return {"error": "Missing required fields"}, 400

    try:
        product = Product(
            name=data['name'],
            sku=data['sku'],
            price=Decimal(str(data['price']))
        )

        db.session.add(product)
        db.session.flush()

        if 'warehouse_id' in data and 'initial_quantity' in data:
            inventory = Inventory(
                product_id=product.id,
                warehouse_id=data['warehouse_id'],
                quantity=data['initial_quantity']
            )
            db.session.add(inventory)

        db.session.commit()

        return {"message": "Product created", "product_id": product.id}, 201

    except IntegrityError:
        db.session.rollback()
        return {"error": "SKU must be unique"}, 409

    except:
        db.session.rollback()
        return {"error": "Failed to create product"}, 500
Why This Fix Works
Products are created independently of warehouses


Inventory creation is optional and flexible


Decimal pricing is handled safely


Single transaction prevents partial failures


SKU uniqueness is enforced


Supports future expansion to multiple warehouses
