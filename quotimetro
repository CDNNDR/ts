import os
import sqlite3
import pandas as pd
import streamlit as st

# Database setup
db_name = "shelly_products_and_relays.db"

# Connect to the database
conn = sqlite3.connect(db_name)
cursor = conn.cursor()

# Create and initialize the `products` table if it doesn't exist
cursor.execute("""
CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    price REAL NOT NULL
)
""")

# Insert products into the database if table is empty
cursor.execute("SELECT COUNT(*) FROM products")
if cursor.fetchone()[0] == 0:
    products = [
        {"name": "Shelly Pro 1 - IP Smart Relay DIN 1ch. LAN/WiFi/BT", "price": 35.70},
        {"name": "Shelly Pro 1PM - IP Smart Relay DIN 1ch. LAN/WiFi/BT + PM", "price": 44.84},
        {"name": "Shelly Pro 2 - IP Smart Relay DIN 2ch. LAN/WiFi/BT", "price": 49.82},
        {"name": "Shelly Pro 2PM - IP Smart Relay DIN 2ch. LAN/WiFi/BT + PM", "price": 60.50},
        {"name": "Shelly Mini 1PM Gen 3 - Smart Relay 8A AC WiFi/BT + PM", "price": 11.05},
        {"name": "Shelly Pro 3EM 120A - Contatore energia DIN + 3 pinze 120A", "price": 87.99},
        {"name": "Shelly Pro 3EM 400A - Contatore Energia DIN + 3 pinze 400A", "price": 169.00}
    ]
    cursor.executemany("""
    INSERT INTO products (name, price) VALUES (:name, :price)
    """, products)

# Create and initialize the `relays` table if it doesn't exist
cursor.execute("""
CREATE TABLE IF NOT EXISTS relays (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    ampere INTEGER NOT NULL,
    type TEXT NOT NULL,
    phases TEXT NOT NULL,
    usage TEXT NOT NULL
)
""")

# Insert relays into the database if table is empty
cursor.execute("SELECT COUNT(*) FROM relays")
if cursor.fetchone()[0] == 0:
    relays = [
        {"name": "Relay 10A Type 1", "ampere": 10, "type": "Type 1", "phases": "monophase", "usage": "automation"},
        {"name": "Relay 16A Type 2", "ampere": 16, "type": "Type 2", "phases": "monophase", "usage": "monitoring"},
        {"name": "Relay 20A Type 3", "ampere": 20, "type": "Type 3", "phases": "triphase", "usage": "automation"},
        {"name": "Relay 25A Type 4", "ampere": 25, "type": "Type 4", "phases": "triphase", "usage": "monitoring"}
    ]
    cursor.executemany("""
    INSERT INTO relays (name, ampere, type, phases, usage) 
    VALUES (:name, :ampere, :type, :phases, :usage)
    """, relays)

# Create and initialize the `sintropy_products` table if it doesn't exist
cursor.execute("""
CREATE TABLE IF NOT EXISTS sintropy_products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    price REAL NOT NULL
)
""")

# Insert Sintropy products into the database if table is empty
cursor.execute("SELECT COUNT(*) FROM sintropy_products")
if cursor.fetchone()[0] == 0:
    sintropy_products = [
        {"name": "AltAir", "price": 100.00},
        {"name": "TAG BLE", "price": 25.00},
        {"name": "HUB", "price": 50.00}
    ]
    cursor.executemany("""
    INSERT INTO sintropy_products (name, price) VALUES (:name, :price)
    """, sintropy_products)

conn.commit()

# Fetch products, relays, and Sintropy products from the database
cursor.execute("SELECT id, name, price FROM products")
products_data = cursor.fetchall()

cursor.execute("SELECT id, name FROM relays")
relays_data = cursor.fetchall()

cursor.execute("SELECT id, name, price FROM sintropy_products")
sintropy_data = cursor.fetchall()

# Initialize session state
if "product_counts" not in st.session_state:
    st.session_state.product_counts = {}
if "relay_counts" not in st.session_state:
    st.session_state.relay_counts = {}
if "sintropy_counts" not in st.session_state:
    st.session_state.sintropy_counts = {}

# Function to calculate costs
def calculate_quote(product_counts, relay_counts, sintropy_counts, product_multiplier, work_multiplier,
                    base_cost=1500, base_devices=21, hourly_rate=100, vat_rate=0.22,
                    site_visit_hours=2, customer_meeting_hours=8, project_setup_hours=8, relay_price=50,
                    business_model='Standard', energy_cost=0, estimated_savings_pct=0, contract_duration=0,
                    fixed_monthly_fee=0, savings_charge_pct=0):
    total_device_count = 0
    total_product_cost = 0

    # Calculate product costs
    cost_data = []
    for product_id, quantity in product_counts.items():
        if quantity > 0:
            for prod in products_data:
                if prod[0] == product_id:
                    product_total = prod[2] * quantity
                    total_product_cost += product_total
                    total_device_count += quantity
                    product_with_vat = product_total * (1 + vat_rate)
                    cost_data.append({"Item": prod[1], "Quantity": quantity, "Total Cost (€)": round(product_with_vat, 2)})

    # Calculate Sintropy product costs
    for sintropy_id, quantity in sintropy_counts.items():
        if quantity > 0:
            for sintropy in sintropy_data:
                if sintropy[0] == sintropy_id:
                    sintropy_total = sintropy[2] * quantity
                    total_product_cost += sintropy_total
                    product_with_vat = sintropy_total * (1 + vat_rate)
                    cost_data.append({"Item": sintropy[1], "Quantity": quantity, "Total Cost (€)": round(product_with_vat, 2)})

    # Calculate relay costs
    for relay_id, quantity in relay_counts.items():
        if quantity > 0:
            for relay in relays_data:
                if relay[0] == relay_id:
                    relay_total = quantity * relay_price
                    cost_data.append({"Item": relay[1], "Quantity": quantity, "Total Cost (€)": relay_total})
                    total_device_count += quantity

    # Labor and your time costs
    cost_per_device = base_cost / base_devices
    labor_cost = round(cost_per_device * total_device_count, 2)
    total_hours = site_visit_hours + customer_meeting_hours + project_setup_hours
    your_cost = round(total_hours * hourly_rate, 2)

    # Total costs
    grand_total_cost = round(total_product_cost * (1 + vat_rate) + labor_cost + your_cost, 2)
    cost_data.append({"Item": "Labor (Electricians)", "Quantity": total_device_count, "Total Cost (€)": labor_cost})
    cost_data.append({"Item": "Your Work (Hours)", "Quantity": total_hours, "Total Cost (€)": your_cost})
    cost_data.append({"Item": "TOTAL COST", "Quantity": "", "Total Cost (€)": grand_total_cost})

    # Convert to a DataFrame
    df_cost = pd.DataFrame(cost_data)

    # Calculate quote (customer price)
    customer_product_cost = total_product_cost * (1 + vat_rate) * product_multiplier
    customer_labor_cost = labor_cost * work_multiplier
    customer_your_cost = your_cost * work_multiplier
    grand_total_price = round(customer_product_cost + customer_labor_cost + customer_your_cost, 2)

    quote_data = []

    if business_model == 'Standard':
        # Standard model calculations
        quote_data = [
            {"Item": "Products (with VAT)", "Multiplier": product_multiplier, "Total Price (€)": round(customer_product_cost, 2)},
            {"Item": "Labor (Electricians)", "Multiplier": work_multiplier, "Total Price (€)": round(customer_labor_cost, 2)},
            {"Item": "Your Work", "Multiplier": work_multiplier, "Total Price (€)": round(customer_your_cost, 2)},
            {"Item": "TOTAL", "Multiplier": "", "Total Price (€)": grand_total_price},
        ]
    elif business_model == 'Start-up Cost with Monthly Fee':
        # Calculate start-up cost as the sum of customer product cost and technician costs
        # For this model, multipliers are 1.0
        customer_product_cost = total_product_cost * (1 + vat_rate)
        customer_labor_cost = labor_cost
        customer_your_cost = your_cost
        start_up_cost = round(customer_product_cost + customer_labor_cost + customer_your_cost, 2)

        # Calculate estimated savings
        estimated_monthly_savings = energy_cost * (estimated_savings_pct / 100)
        savings_based_fee = estimated_monthly_savings * (savings_charge_pct / 100)
        total_monthly_fee = fixed_monthly_fee + savings_based_fee
        total_monthly_fee = round(total_monthly_fee, 2)

        # Total fees over contract duration
        total_monthly_fees = total_monthly_fee * contract_duration
        total_fees = start_up_cost + total_monthly_fees

        # Customer's total savings over contract
        total_savings = estimated_monthly_savings * contract_duration
        net_savings = total_savings - total_fees

        # Enhanced quote data with detailed explanations
        quote_data = [
            {"Item": "Start-up Cost (Products and Installation)", "Details": "", "Amount (€)": round(start_up_cost, 2)},
            {"Item": "Fixed Monthly Fee", "Details": "Monthly service charge", "Amount (€)": round(fixed_monthly_fee, 2)},
            {"Item": "Estimated Monthly Energy Savings", "Details": f"{estimated_savings_pct}% of current energy cost", "Amount (€)": round(estimated_monthly_savings, 2)},
            {"Item": "Savings-Based Fee", "Details": f"{savings_charge_pct}% of estimated savings (€{round(estimated_monthly_savings, 2)})", "Amount (€)": round(savings_based_fee, 2)},
            {"Item": "Total Monthly Fee", "Details": "Fixed fee + Savings-based fee", "Amount (€)": total_monthly_fee},
            {"Item": "Contract Duration", "Details": "", "Amount (€)": f"{contract_duration} months"},
            {"Item": "Total Fees Over Contract", "Details": "Start-up cost + Total monthly fees", "Amount (€)": round(total_fees, 2)},
            {"Item": "Total Estimated Savings", "Details": f"Estimated monthly savings x Contract duration", "Amount (€)": round(total_savings, 2)},
            {"Item": "Net Savings After Fees", "Details": "Total estimated savings - Total fees", "Amount (€)": round(net_savings, 2)}
        ]

    df_quote = pd.DataFrame(quote_data)

    return df_cost, df_quote

# Streamlit app
st.title("Shelly Quote Generator")

# Sidebar menu
with st.sidebar:
    st.title("Options")
    if st.button("Clear Data"):
        # Reset session state for all counts
        st.session_state.product_counts = {}
        st.session_state.relay_counts = {}
        st.session_state.sintropy_counts = {}

        # Rerun the app to clear all inputs and reset the page
        st.experimental_set_query_params()

    # Business Model Selection
    st.header("Business Model")
    business_model = st.selectbox("Select Business Model", ["Standard", "Start-up Cost with Monthly Fee"])

    # Adjustable parameters
    st.header("Adjustable Parameters")
    vat_rate_input = st.number_input("VAT Rate (%)", min_value=0.0, max_value=100.0, value=22.0, step=0.1)
    vat_rate = vat_rate_input / 100

    base_cost = st.number_input("Base Cost (€)", min_value=0.0, value=1500.0, step=50.0)
    base_devices = st.number_input("Base Number of Devices", min_value=1, value=21, step=1)

    hourly_rate = st.number_input("Hourly Rate (€)", min_value=0.0, value=100.0, step=10.0)

    relay_price = st.number_input("Relay Price (€)", min_value=0.0, value=50.0, step=5.0)

    site_visit_hours = st.number_input("Site Visit Hours", min_value=0.0, value=2.0, step=0.5)
    customer_meeting_hours = st.number_input("Customer Meeting Hours", min_value=0.0, value=8.0, step=0.5)
    project_setup_hours = st.number_input("Project Setup Hours", min_value=0.0, value=8.0, step=0.5)

    # Multipliers (only for Standard model)
    if business_model == 'Standard':
        product_multiplier = st.slider("Product Multiplier", min_value=1.0, max_value=2.0, value=1.5, step=0.1)
        work_multiplier = st.slider("Work Multiplier", min_value=1.0, max_value=2.0, value=1.3, step=0.1)
    else:
        product_multiplier = 1.0
        work_multiplier = 1.0

# Initialize variables with default values
energy_cost = 0.0
estimated_savings_pct = 0.0
contract_duration = 0
fixed_monthly_fee = 0.0
savings_charge_pct = 0.0

# Additional inputs for 'Start-up Cost with Monthly Fee' model
if business_model == 'Start-up Cost with Monthly Fee':
    st.header("Customer Energy Details")
    energy_cost = st.number_input("Current Monthly Energy Cost (€)", min_value=0.0, value=2000.0, step=100.0)
    estimated_savings_pct = st.number_input("Estimated Energy Savings (%)", min_value=0.0, max_value=100.0, value=15.0, step=1.0)
    contract_duration = st.number_input("Contract Duration (Months)", min_value=1, value=36, step=1)

    st.header("Financial Parameters")
    fixed_monthly_fee = st.number_input("Fixed Monthly Fee (€)", min_value=0.0, value=150.0, step=10.0)
    savings_charge_pct = st.number_input("Percentage of Savings Charged (%)", min_value=0.0, max_value=100.0, value=35.0, step=1.0)

# Product selection
st.header("Select Products")
col1, col2, col3 = st.columns(3)

# Column 1: Shelly Products
with col1:
    st.subheader("Shelly Products")
    for prod in products_data:
        quantity = st.number_input(f"{prod[1]} (€{prod[2]} each)", min_value=0, step=1, key=f"product_{prod[0]}")
        st.session_state.product_counts[prod[0]] = quantity

# Column 2: Relays
with col2:
    st.subheader("Relays")
    for relay in relays_data:
        quantity = st.number_input(f"{relay[1]} (€{relay_price} each)", min_value=0, step=1, key=f"relay_{relay[0]}")
        st.session_state.relay_counts[relay[0]] = quantity

# Column 3: Sintropy Products
with col3:
    st.subheader("Sintropy Products")
    for sintropy_prod in sintropy_data:
        quantity = st.number_input(f"{sintropy_prod[1]} (€{sintropy_prod[2]} each)", min_value=0, step=1, key=f"sintropy_{sintropy_prod[0]}")
        st.session_state.sintropy_counts[sintropy_prod[0]] = quantity

# Generate quote
if st.button("Generate Quote"):
    if not any(st.session_state.product_counts.values()) and not any(st.session_state.relay_counts.values()) and not any(st.session_state.sintropy_counts.values()):
        st.error("Please select at least one product, relay, or Sintropy product!")
    else:
        cost_table, quote_table = calculate_quote(
            st.session_state.product_counts,
            st.session_state.relay_counts,
            st.session_state.sintropy_counts,
            product_multiplier,
            work_multiplier,
            base_cost=base_cost,
            base_devices=base_devices,
            hourly_rate=hourly_rate,
            vat_rate=vat_rate,
            site_visit_hours=site_visit_hours,
            customer_meeting_hours=customer_meeting_hours,
            project_setup_hours=project_setup_hours,
            relay_price=relay_price,
            business_model=business_model,
            energy_cost=energy_cost,
            estimated_savings_pct=estimated_savings_pct,
            contract_duration=contract_duration,
            fixed_monthly_fee=fixed_monthly_fee,
            savings_charge_pct=savings_charge_pct
        )
        st.subheader("Cost Table (Your Costs)")
        st.dataframe(cost_table)

        st.subheader("Quote Table (Customer Pricing)")
        st.dataframe(quote_table)

        st.download_button("Download Cost Table", cost_table.to_csv(index=False), "cost_table.csv", "text/csv")
        st.download_button("Download Quote Table", quote_table.to_csv(index=False), "quote_table.csv", "text/csv")

# Close the connection
conn.close()
