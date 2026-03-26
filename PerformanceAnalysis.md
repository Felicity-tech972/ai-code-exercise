def find_product_combinations_optimized(products, target_price, price_margin=10):
    products.sort(key=lambda x: x['price']) # Sort once
    results = []
    min_p, max_p = target_price - price_margin, target_price + price_margin

    for i in range(len(products)):
        for j in range(i + 1, len(products)): # Avoid duplicates naturally
            combined = products[i]['price'] + products[j]['price']
            if combined < min_p: continue
            if combined > max_p: break # Early exit: further products are too pricey
            
            results.append({
                'product1': products[i], 'product2': products[j],
                'combined_price': combined,
                'price_difference': abs(target_price - combined)
            })
    results.sort(key=lambda x: x['price_difference'])
    return results
