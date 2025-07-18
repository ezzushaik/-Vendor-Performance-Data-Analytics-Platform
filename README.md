import pandas as pd
import matplotlib.pyplot as plt

# Load CSV files
purchases = pd.read_csv("data/purchases.csv")
purchase_prices = pd.read_csv("data/purchase_prices.csv")
vendor_invoice = pd.read_csv("data/vendor_invoice.csv")
begin_inventory = pd.read_csv("data/begin_inventory.csv")
end_inventory = pd.read_csv("data/end_inventory.csv")
sales = pd.read_csv("data/sales.csv")

# Clean data
for df in [purchases, purchase_prices, vendor_invoice, begin_inventory, end_inventory, sales]:
    df.dropna(inplace=True)

# Merge sales and purchase prices to calculate gross profit
sales_prices = sales.merge(purchase_prices, on="product_id", how="left")
sales_prices["gross_profit"] = sales_prices["sales_price"] - sales_prices["purchase_price"]
sales_prices["total_profit"] = sales_prices["gross_profit"] * sales_prices["quantity"]

# Join with vendor info
sales_with_vendor = sales_prices.merge(purchases[["product_id", "vendor_id"]], on="product_id", how="left")
sales_with_vendor = sales_with_vendor.merge(vendor_invoice[["vendor_id", "vendor_name"]], on="vendor_id", how="left")

# Summarize gross profit by vendor
vendor_profit = sales_with_vendor.groupby(["vendor_id", "vendor_name"]).agg(
    total_profit=pd.NamedAgg(column="total_profit", aggfunc="sum"),
    avg_profit_per_unit=pd.NamedAgg(column="gross_profit", aggfunc="mean"),
    total_units_sold=pd.NamedAgg(column="quantity", aggfunc="sum")
).reset_index()

# Show top 5 vendors by profit
top_vendors = vendor_profit.sort_values(by="total_profit", ascending=False).head(5)
print("Top 5 Vendors by Total Profit:\n", top_vendors)

# Save vendor summary
vendor_profit.to_csv("vendor_profit_summary.csv", index=False)

# Plot profitability
plt.figure(figsize=(10, 6))
plt.bar(top_vendors["vendor_name"], top_vendors["total_profit"], color='skyblue')
plt.title("Top 5 Vendors by Total Profit")
plt.ylabel("Total Profit")
plt.xticks(rotation=45)
plt.tight_layout()
plt.savefig("top_vendors_profit.png")
plt.show()
