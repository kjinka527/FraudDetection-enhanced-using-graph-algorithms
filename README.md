**********Fraud Detection using Graph Algorithms and Machine Learning*********
__________________________________________________________________________________________
** Dataset **

In this project we used BankSim dataset available in Kaggle. 

Number of records - 594643
Fraudulent records - 7200 (1.21%)

Some Fields are:
customerID, merchantID, amount, age, gender, category, Zip code, fraud
__________________________________________________________________________________________
*** System Specifications ***

Python 3.0

neo4j comminuty server 3.5.5
__________________________________________________________________________________________

** Details **

We have built Support Vector Machine, Decision Tree, Random Forest, Extreme Gradient Boosting, CatBoost models
Inorder to further improve our prediction model, we added graph features by projecting the data into graph database-Neo4j
By using following queries, we were able to construct new features and extract to .CSV file



CREATE CONSTRAINT ON (c:Customer) ASSERT c.id IS UNIQUE;
CREATE CONSTRAINT ON (b:Bank) ASSERT b.id IS UNIQUE;


USING PERIODIC COMMIT
LOAD CSV WITH HEADERS FROM
'file:///data.csv' AS line
WITH line,
SPLIT(line.customer, "'") AS customerID,
SPLIT(line.merchant, "'") AS merchantID,
SPLIT(line.age, "'") AS customerAge,
SPLIT(line.gender, "'") AS customerGender,
SPLIT(line.zipcodeOri, "'") AS customerZip,
SPLIT(line.zipMerchant, "'") AS merchantZip,
SPLIT(line.category, "'") AS transCategory


MERGE (customer:Customer {id: customerID[1], age: customerAge[1], gender: customerGender[1], zipCode: customerZip[1]})

MERGE (bank:Bank {id: merchantID[1], zipCode: merchantZip[1]})

CREATE (transaction:Transaction {amount: line.amount, fraud: line.fraud, category: transCategory[1], step: line.step})-[:WITH]->(bank)
CREATE (customer)-[:MAKE]->(transaction);

__________________________________________________________________________________________


*** Create Relationships using below query ***

MATCH (c1:Customer)-[:MAKE]->(t1:Transaction)-[:WITH]->(b1:Bank)
WITH c1, b1
MERGE (p1:Placeholder {id: b1.id})

MATCH (c1:Customer)-[:MAKE]->(t1:Transaction)-[:WITH]->(b1:Bank)
WITH c1, b1
MERGE (p1:Placeholder {id: c1.id})

MATCH (c1:Customer)-[:MAKE]->(t1:Transaction)-[:WITH]->(b1:Bank)
WITH c1, b1, count(*) as cnt
MATCH (p1:Placeholder {id:c1.id})
WITH c1, b1, p1, cnt
MATCH (p2:Placeholder {id: b1.id})
WITH c1, b1, p1, p2, cnt
CREATE (p1)-[:Payes {cnt: cnt}]->(p2)

__________________________________________________________________________________________

*** Call inbuilt graph Algorithms ***

** PAGERANK ALGORITHM**
CALL algo.pageRank('Placeholder', 'Payes', {writeProperty: 'pagerank'})

** LABEL PROPAGATION ALGORITHM **
CALL algo.labelPropagation('Placeholder', 'Payes', 'OUTGOING',
 {write:true, partitionProperty: "community", weightProperty: "count"})

** DEGREE ALGORITHM **
MATCH (p:Placeholder)
SET p.degree = apoc.node.degree(p, 'Payes')

__________________________________________________________________________________________

*** Extract these new features (Pagerank for merchant and customer, community labels for customer and merchant, Page Rank for merchant and customer) to csv using below query
CALL apoc.export.csv.query("MATCH (p:Placeholder) RETURN p.id AS id, p.degree AS degree, p.pagerank as pagerank, p.community AS community ", "results.csv", {})

	1. split the results.csv into customer_features.csv and merchant_features.csv with corresponding data
	2. use these files to merge these new features with original dataset.
	3. Refer BankFraudDetection.ipynb for detailed code execution

__________________________________________________________________________________________
