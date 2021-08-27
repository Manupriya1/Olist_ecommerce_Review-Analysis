# Olist_ecommerce_Review-Analysis


Selected Topic: Brazilian eCommerce Business Trends

In this project, I have analyzed customer orders placed between 2016-2018 from sellers on the Brazilian eCommerce platform Olist Store. From this data, we will first determine the number of customers by geographic region (looking at zip code prefix, city, and state), most popular products, number of repeat customers and purchases, and total purchases by date. After data cleaning and initial analysis, we will use machine learning to make predictions on customer behavior, providing sellers on Olist the opportunity to increase sales and their customer base.

Why we selected our topic:

We chose this topic because this dataset has a range of information and analysis options available from eCommerce data and the potential it has to make predictions with machine learning models, with the information available on customers, sellers, products and geographic regions. This dataset also gave us the opportunity to use SQL joins to create new tables where we joined it on a PostgreSQL server. Our ultimate goal is to predict consumer behavior and review score from this dataset by using machine learning.

Description of our source of data:

We chose the Brazilian-eCommerce dataset from Kaggle for our analysis. This dataset contains approximately 100,000 customer orders, along with corresponding files on product information and English translations of product categories originally in Portuguese. Seller names in this dataset were anonymized and replaced with Game of Thrones House names. Six files from the original Kaggle dataset were chosen for further analysis: geolocation dataset, olist_customers_dataset, olist_product_dataset, order_item_dataset, olist_orders_dataset, and product_category_name_translation.

Data Source: https://www.kaggle.com/olistbr/brazilian-ecommerce

Questions we hope to answer with our data:

With this data, we hope to answer...

What are business trends like for Olist, in general?
What data features impact review scores the most?
Can we predict review scores using machine learning?
Database
We used Postgres to create tables in SQL. The ERD below shows connectivity among 9 tables.

QuickDBD-export (1)

The description of these tables is as follows:

olist_orders_dataset: This table is connected to 4 other tables. It is used to connect all the details related to an order.
olist_order_items_dataset: It contains the details of an item that had been purchased such as shipping date, price and so on.
olist_order_reviews_dataset: It contains details related to any reviews posted by the customer on a particular product that he had purchased.
olist_products_dataset: It contains related to a product such as the ID, category name and measurements.
olist_order_payments_dataset: The information contained in this table is related to the payment details associated with a particular order.
olist_customers_dataset: Details the customer base information of this firm.
olist_geolocation_dataset: It contains geographical information related to both the sellers and customers.
olist_sellers_dataset: This table contains the information related to all the sellers who have registered with this firm.
olist_product_category_name_translation: This table is connected to products database.
We cleaned all these tables using Jupyter Notebook and imported them to a postgresSQL 11 server for joining the tables to create one big database to further analyze. SQL inner joins were used to connect all the tables.

Link to Data_Cleaning Jupyter Notebook

Link to Schema

SQL

Using RDS on AWS with Jupyter notebooks
Step 1: Setting up the Database
We used an AWS free tier template to create our RDS database on AWS. We learned that free templates are only available for Postgres 12 and higher versions.

AWS

We made our database accessible from anywhere so it’s accessible outside of the default VPC.

Step 2: Create AWS server on postgres
Once the database was created on AWS, we created an AWS server with the name final_project on postgres.

pic

We used the schema we created before to create all the tables again and then join them using inner joins.

query

The final joined database was imported to jupyter notebook and it has a total of 34 columns and 91,596 rows.

info

Step 3: Simplifying our data (Machine Learning)
We created a profile of our data to understand the relation between various features. We realized many features in our data did not affect the review score so we dropped those columns. For example, features like "customer_city", "customer_state", "geolocation_lat","geolocation_lng", "customer_id", "order_purchase_timestamp", "order_approved_at", "order_delivered_carrier_date", "payment_sequential" were dropped.

profiling

We confirmed our results by creating a correlation matrix:

corelation_matrix

Finally, we decided to keeping the following columns in our data:

zipcode order_status
price freight_value review_score
payment_type
payment_value product_id
product_photos_qty
product_category
seller_zip
seller_state
time_order_to_delivery
time_estimate_to_delivery
Step 4: Feature Engineering (Machine Learning)
We used the following methods to encode the catagorical variables:

lambda function
label encoder
feature engineering1

Feature engineering 2

Our final data for machine learning has 13 columns:

final

Step 5: Importing CSV
As a first step to connect with AWS database we imported our dataframe to csv file.

CSV

Step 6: Setting up a config file
Once psycopg2 was imported, we created a config.py file to store the details for accessing our database. To connect, we needed the following details:

Endpoint
Port
Name
User’s Name
User’s Password
Step 7: Connecting & Creating a table
After importing psycopg2 and our config file, we created a function that connects to the database and sets up a cursor. This function uses our credentials from our config.py file to create the conn_string, and uses the conn_string to create the connection to our database hosted by AWS.

sql connection1

Step 8: Create Table
Once we had a connection and a cursor, we wrote SQL queries in Python. PostgreSQL queries from Python using the psycopg2 library need four elements:

Establish Connection
Establish Cursor
Execute Cursor
Commit Connection
Step 9: Loading data into a Postgres table from CSV
Once we had our tables, we copied data from CSV files with psycopg2 using the copy_from() method.

SQL connection 2

Step 10: Importing data from AWS database to jupyter notebook for Machine Learning (Code included in the picture above)
We used pandas to read in the SQL database to jupyter notebook. We used the final_customers_sql file to create our machine learning models

SQL_todb

Machine Learning:
Link to Machine Learning jupyter notebook

Goal: To create a Machine Learning model to predict review score. We converted review score to binary variable by using the following code:
Picture1

(See Feature Engineering and Data Preprocessing for Machine Learning in Database section above)
Description of how data was split into training and testing sets:

The goal of the training and testing sets is to create the machine learning model to predict review score. We took the review score and used it as a target column, then made it into y data. After that, we put x and y into training and testing to make X_train, X_test and y_train, and y_test. We split the data by making a 75/25 split where 25% of the data was used for testing. X is everything except review score and y is review score.

training_testing_split

Resampling

Our data has more positive reviews than negative reviews, as the pie chart below shows.

pie chart

We used Random undersampling to resample our data using the following code:

resample

Model Choice:

As our target variable is binary, we choose a classification model. We created 5 different models and compared their accuracies. We got accuracy >70% for all the models except for Artificial Neural Network that gave us only 50% of accuracy. Random forest performed the best for our data and we used this model for our web app.

Summary of Machine Learning Models Performance

Linear Logistic Regression: We got accuracy of 74%. Logistic regression is easier to implement and interpret, and very efficient to train. But it is tough to obtain complex relationships using logistic regression and it over fits the model.

K-NN model: We were hoping to receive higher accuracy with K-NN but we got lower accuracy of 67% than logistic regression. One benefit of KNN algorithm is that it doesn’t require training before making predictions, new data can be added seamlessly which will not impact the accuracy of the algorithm.

Decision Tree: We got accuracy of ~92%. Decision Tree algorithm is very intuitive and easy to understand but a small change in the data can change the prediction big time.

Ensemble-Random Forest: As this combines various decision tree models, it gave us the best value of accuracy of ~98% . The Random Forest doesn’t over fit the model but it makes algorithms to run slow.

confusion_matrix

Artificial Neural Network: We were hoping to get the highest accuracy with this one but it gave us accuracy of 50%. ANN can overfit the data and takes a lot of time to run.
Here is a graph that shows the comparison of all the models we tested:
model comparison

Feature Importance
We used Random Forest feature importance technique (as Random Forest has the maximum accuracy ~98% among all the models, we tried to find importance score for input features based on how useful they are at predicting a target variable). As the picture below shows, the most important feature is the "time estimate to delivery" which is the difference between estimate delivery date and actual delivery date and the least important feature is the "order status."

feature importance

Predictive Web App
We used Streamlit to create an interactive web app to predict the review score for various combinations of the features. We used Random Forest classifier model to create the prediction.

Link to Predictive Web App

Predictive App

Dashboard
Link to dashboard on Google Slides

Link to video of dashboard interactivity

Link to Dashboard on Tableau

This report starts by telling the story of Olist, how they began, where there customers are at, what products sell the most and ways to improve their services. Olist is a company that begins its online presence in the Fall of 2016 and builds a robust presence in 2017 and 2018. We review the products that they help distribute and how they're performing around the country. As part of their quality control review they send out surveys for feedback on their services, we were able to use their star rating to see how customers felt about the services provided. Finally part of the service Olist provides is logistics and delivery is largely discussed in these reviews, we attempt to draw this picture in our visualizations.

Visualization Tools
The team decided to move forward with Tableau for our dashboard needs. The data, as previously described, was cleaned, merged and further processed via Python, postgresSQL, and AWS. The dataset that we've been working with include some of the changes made for the machine learning models, changing strings into integers. Also, in assistance to our Tableau learning journey, aside from the module, were youtube videos for guidance on connecting worksheets for interactive connections and overlapping maps to demonstrate the data.

Sales Dashboard
Olist Sales Review

In this board we see the popularity of items being sold on Olist, most popular item are home goods and least popular are security services. We used cluster or bubble chart to review this and a map to locate places where these items were sold most frequently.

Review Dashboard
Review_Dashboard The next dashboard illustrates the reviews and how most categories were rated. Overall Olist provides a good service with most items receiving reviews of 3.5 and greater. There are negative reviews and there will be seen in areas where delivering large ticket items may not be as easy as delivering smaller items. Most of our machine learning visualizations and results are located in this frame as well. The logistics might be different. For this reason we also reviewed freight costs and services.

Interactive elements:

line graphs that can be filtered by year
Maps that can be filtered by customer review scores, product category, and city
Conclusion
From this analysis, we concluded that for our future analysis, we would hope to analyze the most recent orders with the orders we’ve analyzed from 2016-2018. With the more recent data, we would compare the olist orders with orders place on the other eCommerce platforms in Brazil. We would also like to compare the orders placed per region while also looking at the population, per capita income, access to online banking, and access to internet since about 20 million Brazilians do not have access to internet.








