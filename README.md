# Challenge4

please find the file named "module_4_challenge.ipynb" for the output
https://github.com/aVa7889/Challenge4/blob/main/Starter_Code/module_4_challenge.ipynb


# Summary of the challenge 4

I pulled the top 5 clients of jobs, and the industry areas are broad. They are from the Production designer, chief of staff, osteopath, dietitian, and pension scheme manager.
It is interesting to know that the top 5 clients are all across the board.  please see "module_4_challenge.ipynb' the output[28].

	client_number	Quantity	Shipping_Price	Total_Price	Cost	Profit	client_id	job
0	33615	64313	$1.83M	$8.38M	$6.18M	$2.20M	33615	'Production designer, theatre/television/film'

1	46820	75768	$1.60M	$9.74M	$7.01M	$2.74M	46820	'Chief of Staff'

2	66037	43018	$1.40M	$10.26M	$7.00M	$3.26M	66037	'Osteopath'

3	38378	73667	$3.43M	$12.91M	$9.63M	$3.27M	38378	'Dietitian

4	24741	239862	$5.13M	$82.27M	$45.69M	$36.58M	24741	'Pension scheme manager'


## Part 1: Explore the Data

Import the data and use Pandas to learn more about the dataset.
import pandas as pd

df = pd.read_csv('Resources/client_dataset.csv')

df.head()
# View the column names in the data

df.columns
# Use the describe function to gather some basic statistics

df.describe()
# Use this space to do any additional research
# and familiarize yourself with the data.

df = df.sort_values("client_id", ascending=False)
df.head()

order_id_order_year_df = df.sort_values(
    ["order_id", "order_year"], ascending=True)
order_id_order_year_df.head(15)

df = df.reset_index(drop=True)
df.head()

del df['line_number']
df.head()

# What three item categories had the most entries?
df['category'].value_counts().head(3)


# For the category with the most entries,
# which subcategory had the most entries?

df.loc[df['category'] == 'consumables', 'subcategory'].value_counts().head(1)
# Which five clients had the most entries in the data?

df['client_id'].value_counts().head(5)

# Store the client ids of those top 5 clients in a list.

top_5_clients = list(df['client_id'].value_counts().head(5).index)
print(top_5_clients)
# How many total units (the qty column) did the
# client with the most entries order order?
df.loc[df["client_id"] == 33615, "qty"].sum()




## Part 2: Transform the Data
Do we know that this client spent the more money than client 66037? If not, how would we find out? Transform the data using the steps below to prepare it for analysis.
# Create a column that calculates the 
# subtotal for each line using the unit_price
# and the qty

df['subtotal'] = df['unit_price'] * df['qty']
df.head()


# Create a column for shipping price.
# Assume a shipping price of $7 per pound
# for orders over 50 pounds and $10 per
# pound for items 50 pounds or under.

df['total_weight'] = df['unit_weight'] * df['qty']

def shipping_price(total_weight):
    if total_weight >  50:
        shipping_price = 7 * total_weight
    else:
        shipping_price = 10 * total_weight
    return shipping_price

df['shipping_price'] = df['total_weight'].apply(shipping_price)

df.head()


# Create a column for the total price
# using the subtotal and the shipping price
# along with a sales tax of 9.25%

df['totalPrice_noTax'] = df['subtotal'] + df['shipping_price']
df['tax'] = df['totalPrice_noTax'] * 0.0925
df['total_price'] = df['totalPrice_noTax'] + df['tax']
df.head()
# Create a column for the cost
# of each line using unit cost, qty, and
# shipping price (assume the shipping cost
# is exactly what is charged to the client).

df['cost'] = (df['unit_cost'] * df['qty']) + df['shipping_price']
df.head()
# Create a column for the profit of
# each line using line cost and line price

df['profit'] = df['total_price'] - df['cost']
df.head()
## Part 3: Confirm your work
You have email receipts showing that the total prices for 3 orders. Confirm that your calculations match the receipts. Remember, each order has multiple lines.

Order ID 2742071 had a total price of \$152,811.89

Order ID 2173913 had a total price of \$162,388.71

Order ID 6128929 had a total price of \$923,441.25

# Check your work using the totals above
def total_price_calculation(order_id, df):
    total_price = df[df['order_id'] == order_id]['total_price'].sum()
    return total_price

order_id = [2742071, 2173913, 6128929]
for order_number in order_id:
    total_price = total_price_calculation(order_number, df)
    display(f"Order ID {order_number} has a total price of ${total_price:.2f}")


## Part 4: Summarize and Analyze
Use the new columns with confirmed values to find the following information.
# How much did each of the top 5 clients by quantity
# spend? Check your work from Part 1 for client ids.
def client_spent(client_number, total_price):
    client_df = df.loc[df['client_id'] == client_number, total_price]
    return round(client_df.sum(), 2)

for client_number in top_5_clients:
    print(f"{client_number}: ${client_spent(client_number, 'total_price')}")


# Create a summary DataFrame showing the totals for the
# for the top 5 clients with the following information:
# total units purchased, total shipping price,
# total revenue, and total profit. Sort by total profit.

top_5_clients_dict = {"client_number": top_5_clients}


columns = ['qty', 'shipping_price', 'total_price','cost', 'profit']

for summary in columns:
    top_5_clients_dict[summary] = [client_spent(client_number, summary) for client_number in top_5_clients]

summary_df = pd.DataFrame(top_5_clients_dict).sort_values("profit", ascending=True)
summary_df


# Format the data and rename the columns
# to names suitable for presentation.
# Currency should be in millions of dollars.

column_dict = {'qty':'Quantity', 'cost': 'Cost', 'shipping_price': 'Shipping_Price', 'total_price': 'Total_Price', 'profit': 'Profit'}
summary_df = summary_df.rename(columns=column_dict)
summary_df

def format_millions(value):
    if value >= 1000000:
        return f"${value/1000000:.2f}M"
    else:
        return f"${value:,.2f}"

summary_df[['Cost', 'Shipping_Price', 'Total_Price', 'Profit']] = summary_df[['Cost', 'Shipping_Price', 'Total_Price', 'Profit']].map(format_millions)
summary_df

            
# Sort the updated data by "Total Profit" form highest to lowest
summary_df.sort_values('Profit', ascending=False)

