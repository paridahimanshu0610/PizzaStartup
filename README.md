## Introduction
Jonathan was a pizza lover who stumbled upon a unique idea while scrolling through his social media feed - "Revolutionizing Pizza Delivery: Fast, Fresh, and Affordable!"

Determined to turn this idea into a business, Jonathan decided to launch Pizza Runner - a platform that would deliver freshly made pizzas to customers at lightning-fast speed. To bring his vision to life, he recruited a team of runners to deliver pizzas from his kitchen, while also investing in a mobile app that allowed customers to place their orders online.

With hard work and determination, Jonathan's Pizza Runner quickly gained popularity, paving the way for a successful pizza delivery empire.

## Dataset
The case study relies on several key datasets:

- **runners**: This table contains data about the runners who deliver pizzas for Pizza Runner. Specifically, it includes the date on which each runner registered with the company.
- **customer_orders**: This table records all customer pizza orders. Each row represents an individual pizza within an order, and includes the pizza_id (which indicates the type of pizza ordered), exclusions (a comma-separated list of ingredient IDs that should be removed from the pizza), and extras (a comma-separated list of ingredient IDs that should be added to the pizza).
- **runner_orders**: Once an order is received by Pizza Runner, it is assigned to a runner. The runner_orders table records this information, as well as whether the order was completed or cancelled. Additionally, it includes the pickup_time (the timestamp at which the runner arrives at Pizza Runner headquarters to pick up the pizzas), distance (the distance the runner traveled to deliver the order, measured in kilometers), duration (the time the runner spent delivering the order, measured in minutes), and cancellation (whether the order was cancelled by the customer or the restaurant).
- **pizza_names**: Pizza Runner offers only two pizza options: Meat Lovers and Vegetarian.
- **pizza_recipes**: This table specifies the standard toppings included on each type of pizza, identified by pizza_id.
- **pizza_toppings**: This table lists all possible pizza toppings, identified by topping_id, along with their corresponding topping_name.

## Data Clean
Before the datasets could be used in the case study, some data cleaning was necessary. The specific issues and actions taken for each table are as follows:

**customer_orders table:**

The **exclusions** and **extras** columns contained blank spaces and null values, which needed to be cleaned up before the columns could be used in queries.

**runner_orders table:**

The **pickup_time**, **distance**, **duration**, and **cancellation** columns contained null values, blank spaces, and extraneous text, which required cleaning before the data could be used. Specifically, the following actions were taken:
- **pickup_time**: Null values were removed.
- **distance**: Null values were removed, and the "km" unit was stripped from each value.
- **duration**: Null values were removed, and extraneous text ("minute", "minutes", "mins") was stripped from each value.
- **cancellation**: Blank spaces and null values were removed.

## Entity Relationship Diagram
![alt text](https://github.com/paridahimanshu0610/PizzaStartup/blob/main/ERD.jfif)