# Amazon DynamoDB Setup Guide

## Problem Statement

A company is building a real-time data analytics application that processes a large number of transactions in an e-commerce environment. They need a database that can scale efficiently to handle high volumes of traffic and provide low-latency reads and writes.

## Step-by-Step Instructions for Creating a DynamoDB Instance

### Step 1: Open the AWS Management Console

1. Navigate to the [AWS Management Console](https://aws.amazon.com/console/).
2. Sign in to your AWS account.

### Step 2: Create a DynamoDB Table

1. In the AWS Management Console, search for **DynamoDB** and select it.

2. Click on **Create table**.

![Create Table Button](images/create_table.png)

3. Enter the table name as `Transactions`.

4. Define the primary key:
   - **Partition Key**: `TransactionID` (String)
   - **Sort Key**: `Timestamp` (Number)

![Create Table Configuration](images/create_table_button.png)

5. Configure the read and write capacity. You can use the **Capacity calculator** to estimate your needs.

![Capacity Calculator](images/capacity_calculator.png)

6. Optionally, enable **Deletion protection** to prevent accidental deletion of the table.

![Deletion Protection](images/deletion_protection.png)

7. Click **Create** to create the table.

![Creating Table](images/creating_table.png)

### Step 3: Insert Data into the Table

1. After the table is created, navigate to the **Items** tab.

![Items Tab](images/items_tab.png)

2. Click on **Create item**.

![Create Item Button](images/create_item_button.png)

3. Add the following attributes:
   - `TransactionID` (String)
   - `Timestamp` (Number)
   - `CustomerID` (String)
   - `Amount` (Number)
   - `PaymentMethod` (String)

![Create Item Form](images/create_item_form.png)

4. Click **Create item** to save the record.

### Step 4: Verify the Table Creation

1. Go back to the **Tables** tab to see your newly created table.

![Tables Overview](images/tables_overview.png)

2. Ensure the status shows as **Active**.

### Step 5: Query Data

1. In the **Items** tab, use the **Query** button to retrieve records based on the `TransactionID`.

![Query Button](images/query_button.png)

## AWS CLI Commands for DynamoDB Setup

### Create Table

Run the following command to create the DynamoDB table:

bash
aws dynamodb create-table 

--table-name Transactions 

--attribute-definitions AttributeName=TransactionID,AttributeType=S AttributeName=Timestamp,AttributeType=N 

--key-schema AttributeName=TransactionID,KeyType=HASH AttributeName=Timestamp,KeyType=RANGE 

--provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5


### Insert Data

Use the following command to insert a record into the DynamoDB table:

bash
aws dynamodb put-item 

--table-name Transactions 

--item '{
"TransactionID": {"S": "T12345"},
"Timestamp": {"N": "1665500000"},
"CustomerID": {"S": "C56789"},
"Amount": {"N": "100"},
"PaymentMethod": {"S": "CreditCard"}
}'


### Query Data

To query the table for records by `TransactionID`, use this command:

```bash
aws dynamodb query 

--table-name Transactions 

--key-condition-expression "TransactionID = :transId" 

--expression-attribute-values '{":transId": {"S": "T12345"}}'
```

To filter results by `CustomerID`, use:

```bash
aws dynamodb query 

--table-name Transactions 

--key-condition-expression "TransactionID = :transId" 

--filter-expression "CustomerID = :custId" 

--expression-attribute-values '{":transId": {"S": "T12345"}, ":custId": {"S": "C56789"}}'
```

## Conclusion

You have successfully set up a DynamoDB instance, created a table, inserted data, and queried the records. This setup will support your real-time data analytics application effectively.

