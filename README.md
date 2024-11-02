from flask import Flask, render_template_string, request, jsonify

app = Flask(_name_)

# Inventory data
inventory = {
    'Tomatoes': {'quantity': 100, 'price': 1.5},
    'Carrots': {'quantity': 100, 'price': 0.8},
    'Spinach': {'quantity': 100, 'price': 1.0},
    'Apples': {'quantity': 100, 'price': 2.0},
    'Bananas': {'quantity': 100, 'price': 1.2}
}

# HTML Template with inline CSS and JavaScript
html_template = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Urban Farming Delivery</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        h1, h2 { color: #4CAF50; }
        #inventory .item { margin: 10px 0; }
        button { background-color: #4CAF50; color: white; padding: 10px; border: none; cursor: pointer; }
        button:hover { background-color: #45a049; }
        #order-summary { margin-top: 20px; padding: 10px; border: 1px solid #ddd; }
    </style>
</head>
<body>
    <h1>Urban Farming Organic Delivery</h1>
    
    <h2>Available Produce</h2>
    <div id="inventory">
        {% for item, details in inventory.items() %}
            <div class="item">
                <span>{{ item }} - ${{ details.price }} each</span>
                <input type="number" id="{{ item }}" placeholder="Quantity" min="0">
            </div>
        {% endfor %}
    </div>

    <button onclick="placeOrder()">Place Order</button>

    <h2>Order Summary</h2>
    <div id="order-summary"></div>

    <script>
        async function placeOrder() {
            const order = {};
            const items = document.querySelectorAll("#inventory .item");

            items.forEach(item => {
                const input = item.querySelector("input");
                const quantity = parseInt(input.value);
                if (quantity > 0) {
                    order[input.id] = quantity;
                }
            });

            const response = await fetch('/place_order', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ order })
            });

            const orderSummaryDiv = document.getElementById('order-summary');
            orderSummaryDiv.innerHTML = '';

            if (response.ok) {
                const data = await response.json();
                data.order_summary.forEach(item => {
                    const div = document.createElement('div');
                    div.textContent = ${item.quantity} units of ${item.item} - $${item.cost.toFixed(2)};
                    orderSummaryDiv.appendChild(div);
                });
                const totalDiv = document.createElement('div');
                totalDiv.textContent = Total Cost: $${data.total_cost.toFixed(2)};
                orderSummaryDiv.appendChild(totalDiv);
            } else {
                const error = await response.json();
                orderSummaryDiv.textContent = Error: ${error.message};
            }
        }
    </script>
</body>
</html>
"""

@app.route('/')
def index():
    # Render the HTML with the current inventory
    return render_template_string(html_template, inventory=inventory)

@app.route('/place_order', methods=['POST'])
def place_order():
    order_items = request.json.get('order', {})
    total_cost = 0
    delivery_items = []

    # Process each item in the order
    for item, qty in order_items.items():
        if item in inventory and inventory[item]['quantity'] >= qty:
            inventory[item]['quantity'] -= qty
            cost = inventory[item]['price'] * qty
            total_cost += cost
            delivery_items.append({'item': item, 'quantity': qty, 'cost': cost})
        else:
            return jsonify({'status': 'error', 'message': f"Not enough {item} available."}), 400

    return jsonify({
        'status': 'success',
        'order_summary': delivery_items,
        'total_cost': total_cost
    })

if _name_ == "_main_":
    app.run(debug=True)
