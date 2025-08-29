# NoSQL Activity

## Software System Development Lab Activity 3

**Date:** 21 August  
**Lecture:** NoSQL  
**Database:** sample_supplies Collection: sales  
**Total Marks:** 20  
**Deadline:** 22 August, 5 PM  

### Instructions
- Use MongoDB Shell mongosh only. No scripts and no code files.
- Atlas option: load Sample Data to get sample_supplies.sales. Local option: import the JSON mirror below. Do not write to sales.
- First clone sales into sales_work. All writes go to sales_work. Reads in Tasks 5 to 10 must use sales.
- For each question k run exactly one command in mongosh. Copy that command into qk_query.txt and the exact one line JSON output into qk_output.txt.
- Always wrap your expression with printjsononeline(...) so the output is a single JSON value on one line.
- Work individually. Plagiarism results in zero.

## Dataset Setup

### Atlas Setup
Open your cluster, click Collections, then Load Sample Dataset. Use sample_supplies.sales.  

### Local Setup
Download JSON and import:

```bash
curl -L -o sales.json \
https://raw.githubusercontent.com/neelabalan/mongodb-sample-dataset/main/sample_supplies/sales.json

mongoimport --uri "mongodb://localhost:27017" \
--db sample_supplies --collection sales \
--file sales.json --jsonArray
```

## Initialize MongoDB

### Connect to MongoDB
```bash
mongosh
use sample_supplies
```

### Clone Base Collection to Working Copy
Run this command once to create sales_work:

```javascript
printjsononeline((function(){
db.sales_work?.drop();
db.sales.aggregate([
{$match:{}},
{$merge:{into:"sales_work", whenMatched:"fail", whenNotMatched:"insert"}}
]);
return {src: db.sales.countDocuments({}), work: db.sales_work.countDocuments({})};
})())
```

## MongoDB Query Patterns

### 1. Array from Find Query
```javascript
printjsononeline(
db.sales.find({FILTER}, {PROJ}).sort({SORT}).limit(N).toArray()
)
```

### 2. Array from Aggregation
```javascript
printjsononeline(
db.sales.aggregate([ STAGES... ]).toArray()
)
```

### 3. Single Object or Number
```javascript
printjsononeline({count: db.sales.countDocuments({FILTER})})
```

## Submission Format
Create a folder `<roll_number>_lab3` and submit it as a zip file. For each question k add:
- `qk_query.txt`: One line containing the exact command you executed.
- `qk_output.txt`: One line containing the exact JSON printed by mongosh.

**Example:**
```bash
# q1_query.txt
printjsononeline({count: db.sales.countDocuments({})})

# q1_output.txt
{"count":16050}
```

Write any assumptions or interpretations in a README.txt.

## Context
You are a new data analyst at SupplyChainX, a global office supplies retailer. The ops lead wants quick changes in a sandbox copy and fast readouts from the live data.

## Tasks Overview
Ten tasks at 2 marks each:
- **Tasks 1-4:** Target `sales_work` collection (CRUD operations)
- **Tasks 5-10:** Target `sales` collection (Analytics queries)
- Numeric checks use absolute tolerance 1e-6

---

<details>
<summary><strong>Task 1. Create [2 Marks]</strong></summary>

**Scenario:** Ops lead asked you to record a training order so downstream tools see a fresh shape.

**Task:** Insert one document into sales_work with your details:
- orderId: LAB_<ROLL>
- saleDate: ISO date of today
- items: one element with name of your choice, tags ["lab","custom"], price: 10.0, quantity: 1
- storeLocation: "Training"
- customer: set email to your email, age to your age, satisfaction to 4
- couponUsed: false, purchaseMethod: "Online"

**Output:** Insert result object with acknowledged and insertedId.

**Example Query:**
```javascript
printjsononeline(db.sales_work.insertOne({
  orderId: "LAB_12345",
  saleDate: new ISODate(),
  items: [{name: "Test Item", tags: ["lab","custom"], price: 10.0, quantity: 1}],
  storeLocation: "Training",
  customer: {email: "student@example.com", age: 20, satisfaction: 4},
  couponUsed: false,
  purchaseMethod: "Online"
}))
```
</details>

<details>
<summary><strong>Task 2. Read [2 Marks]</strong></summary>

**Scenario:** Review team wants to confirm the row landed with the right keys.

**Task:** From sales_work, fetch your document by orderId. Project orderId, storeLocation, purchaseMethod, and first item name as item0. Exclude _id.

**Output:** A single object.

**Example Query:**
```javascript
printjsononeline(db.sales_work.findOne(
  {orderId: "LAB_12345"},
  {_id: 0, orderId: 1, storeLocation: 1, purchaseMethod: 1, "item0": "$items.0.name"}
))
```
</details>

<details>
<summary><strong>Task 3. Update [2 Marks]</strong></summary>

**Scenario:** Review team flagged that you undercounted quantities.

**Task:** On sales_work, for your orderId increment items.0.quantity by 2 and set customer.satisfaction to 5.

**Output:** Update result object with matchedCount and modifiedCount.

**Example Query:**
```javascript
printjsononeline(db.sales_work.updateOne(
  {orderId: "LAB_12345"},
  {$inc: {"items.0.quantity": 2}, $set: {"customer.satisfaction": 5}}
))
```
</details>

<details>
<summary><strong>Task 4. Delete [2 Marks]</strong></summary>

**Scenario:** Sandbox cleanup request.

**Task:** Delete your inserted row from sales_work by orderId.

**Output:** Delete result object with deletedCount.

**Example Query:**
```javascript
printjsononeline(db.sales_work.deleteOne({orderId: "LAB_12345"}))
```
</details>

<details>
<summary><strong>Task 5. Top Stores by Revenue [2 Marks]</strong></summary>

**Scenario:** The ops lead wants a quick league table of stores.

**Task:** On sales, compute revenue per storeLocation as sum of items.price * items.quantity. Return top 5 sorted by revenue desc then storeLocation asc. Convert revenue to a JSON number.

**Output:** Array {storeLocation, revenue}.

**Example Query:**
```javascript
printjsononeline(db.sales.aggregate([
{$unwind:"$items"},
{$group:{_id:"$storeLocation", rev:{$sum:{$multiply:["$items.price","$items.quantity"]}}}},
{$project:{_id:0,storeLocation:"$_id",revenue:{$toDouble:"$rev"}}},
{$sort:{revenue:-1,storeLocation:1}},
{$limit:5}
]).toArray())
```
</details>

<details>
<summary><strong>Task 6. Monthly Revenue, 2015 [2 Marks]</strong></summary>

**Scenario:** Finance wants a 2015 month series for reconciliation.

**Task:** On sales, for orders in 2015 return {month: "YYYY-MM", revenue} sorted by month asc. Only months present in data should appear. Convert revenue to a JSON number.

**Output:** Array of objects.

**Example Query:**
```javascript
printjsononeline(db.sales.aggregate([
{$match: {saleDate: {$gte: ISODate("2015-01-01"), $lt: ISODate("2016-01-01")}}},
{$unwind: "$items"},
{$group: {
  _id: {$dateToString: {format: "%Y-%m", date: "$saleDate"}},
  revenue: {$sum: {$multiply: ["$items.price", "$items.quantity"]}}
}},
{$project: {_id: 0, month: "$_id", revenue: {$toDouble: "$revenue"}}},
{$sort: {month: 1}}
]).toArray())
```
</details>

<details>
<summary><strong>Task 7. Basket Size by Channel [2 Marks]</strong></summary>

**Scenario:** Product Manager suspects phone orders contain fewer distinct items.

**Task:** On sales, for each purchaseMethod compute average number of distinct item names per order. Round to 2 decimals. Sort by avg_distinct_items desc then purchaseMethod asc.

**Output:** Array {purchaseMethod, avg_distinct_items}.

**Example Query:**
```javascript
printjsononeline(db.sales.aggregate([
{$project: {
  purchaseMethod: 1,
  distinctItems: {$size: {$setUnion: ["$items.name", []]}}
}},
{$group: {
  _id: "$purchaseMethod",
  avg_distinct_items: {$avg: "$distinctItems"}
}},
{$project: {
  _id: 0,
  purchaseMethod: "$_id",
  avg_distinct_items: {$round: ["$avg_distinct_items", 2]}
}},
{$sort: {avg_distinct_items: -1, purchaseMethod: 1}}
]).toArray())
```
</details>

<details>
<summary><strong>Task 8. Tags Dictionary [2 Marks]</strong></summary>

**Scenario:** Marketing is cleaning tag taxonomy.

**Task:** On sales, list all distinct items.tags as a sorted array of strings.

**Output:** Array of strings sorted asc.

**Example Query:**
```javascript
printjsononeline(db.sales.aggregate([
{$unwind: "$items"},
{$unwind: "$items.tags"},
{$group: {_id: "$items.tags"}},
{$sort: {_id: 1}},
{$group: {_id: null, tags: {$push: "$_id"}}},
{$project: {_id: 0, tags: 1}}
]).toArray()[0].tags)
```
</details>

<details>
<summary><strong>Task 9. Payment Channel Quality Proxy [2 Marks]</strong></summary>

**Scenario:** Support asks which channel yields happier customers.

**Task:** On sales, using customer.satisfaction where present, return average satisfaction per purchaseMethod for orders since 2015. Round to 2 decimals. Sort by avg_satisfaction desc then purchaseMethod asc.

**Output:** Array {purchaseMethod, avg_satisfaction, n}.

**Example Query:**
```javascript
printjsononeline(db.sales.aggregate([
{$match: {
  saleDate: {$gte: ISODate("2015-01-01")},
  "customer.satisfaction": {$exists: true}
}},
{$group: {
  _id: "$purchaseMethod",
  avg_satisfaction: {$avg: "$customer.satisfaction"},
  n: {$sum: 1}
}},
{$project: {
  _id: 0,
  purchaseMethod: "$_id",
  avg_satisfaction: {$round: ["$avg_satisfaction", 2]},
  n: 1
}},
{$sort: {avg_satisfaction: -1, purchaseMethod: 1}}
]).toArray())
```
</details>

<details>
<summary><strong>Task 10. Top Items by Revenue [2 Marks]</strong></summary>

**Scenario:** Buying wants the revenue leaders.

**Task:** On sales, return the top 10 item names by total revenue sum of price * quantity. Convert revenue to a JSON number. Sort by revenue desc then name asc.

**Output:** Array {name, revenue}.

**Example Query:**
```javascript
printjsononeline(db.sales.aggregate([
{$unwind: "$items"},
{$group: {
  _id: "$items.name",
  revenue: {$sum: {$multiply: ["$items.price", "$items.quantity"]}}
}},
{$project: {
  _id: 0,
  name: "$_id",
  revenue: {$toDouble: "$revenue"}
}},
{$sort: {revenue: -1, name: 1}},
{$limit: 10}
]).toArray())
```
</details>

---

## Example Output Format

**q5_query.txt:**
```javascript
printjsononeline(db.sales.aggregate([{$unwind:"$items"},{$group:{_id:"$storeLocation",rev:{$sum:{$multiply:["$items.price","$items.quantity"]}}}},{$project:{_id:0,storeLocation:"$_id",revenue:{$toDouble:"$rev"}}},{$sort:{revenue:-1,storeLocation:1}},{$limit:5}]).toArray())
```

**q5_output.txt:**
```json
[{"storeLocation":"Denver","revenue":12345.67}, {"storeLocation":"Seattle","revenue":11234.56}]
```