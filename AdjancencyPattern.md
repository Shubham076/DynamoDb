
# DynamoDB Adjacency List Pattern with GSI for Invoices and Bills

## Overview

This guide demonstrates modeling and querying many-to-many relationships between invoices and bills in Amazon DynamoDB. It utilizes an adjacency list pattern and a Global Secondary Index (GSI) to efficiently manage complex relational data.

## Table Design

The main table is designed to store both entities and their relationships, facilitating direct queries from invoices to associated bills.

### Attributes

-   **PK (Partition Key)**: Combines the entity type and ID, e.g., `INVOICE#<InvoiceID>` or `BILL#<BillID>`.
-   **SK (Sort Key)**: Represents either the entity itself or its relationship, enabling diverse data access patterns within a single table structure.

### Example Data

| PK            | SK            | Additional Attributes |
|---------------|---------------|-----------------------|
| `INVOICE#123` | `INVOICE#123` | Invoice details       |
| `INVOICE#123` | `BILL#789`    | Bill reference        |
| `BILL#789`    | `BILL#789`    | Bill details          |
                          
## Global Secondary Index (GSI)

The GSI inversely indexes the data to efficiently support querying from bills to find all associated invoices.

### GSI Structure

-   **GSI Partition Key**: Utilizes what is the SK in the main table to allow bill-centric queries.
-   **GSI Sort Key**: Utilizes what is the PK in the main table, facilitating the listing of all invoices for a bill.

## Use Cases and Query Examples

### Querying All Bills for an Invoice

To find all bills related to a specific invoice, query the main table using the invoice's ID as the PK.


```js// Query for all bills associated with Invoice#123
const params = {
  TableName: "InvoicesBills",
  KeyConditionExpression: "PK = :invoiceId AND begins_with(SK, :skPrefix)",
  ExpressionAttributeValues: {
    ":invoiceId": "INVOICE#123",
    ":skPrefix": "BILL#"
  }
};
``` 

### Querying All Invoices Containing a Specific Bill

To find all invoices that a specific bill is part of, query the GSI using the bill's ID as the PK.


```js// GSI query for all invoices containing Bill#789
const gsiParams = {
  TableName: "InvoicesBills",
  IndexName: "BillInvoiceIndex",
  KeyConditionExpression: "PK = :billId AND begins_with(SK, :skPrefix)",
  ExpressionAttributeValues: {
    ":billId": "BILL#789",
    ":skPrefix": "INVOICE#"
  }
};
``` 

## Implementation Steps

1.  **Create Main Table**: Define the PK and SK, aligning with the entity and relationship structure.
2.  **Create GSI**: Invert the main table's PK and SK for the GSI's keys to support reverse lookup patterns.
3.  **Data Entry**: Populate the table with invoices, bills, and their relationships as designed.
4.  **Query Data**: Utilize the main table and GSI for querying relationships, adjusting parameters as needed for specific use cases.

## Conclusion

This pattern highlights DynamoDB's capability to manage and efficiently query complex relationships, like those between invoices and bills, using an adjacency list pattern complemented by a GSI. This approach ensures scalable, cost-effective, and performant data management for applications requiring sophisticated relational data handling.
