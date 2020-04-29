# 对比学习关系型数据库Mysql和图数据库Neo4j

- [对比学习关系型数据库Mysql和图数据库Neo4j](#%E5%AF%B9%E6%AF%94%E5%AD%A6%E4%B9%A0%E5%85%B3%E7%B3%BB%E5%9E%8B%E6%95%B0%E6%8D%AE%E5%BA%93mysql%E5%92%8C%E5%9B%BE%E6%95%B0%E6%8D%AE%E5%BA%93neo4j)
  - [你试那么多次,不如刷一次文档来得快](#%E4%BD%A0%E8%AF%95%E9%82%A3%E4%B9%88%E5%A4%9A%E6%AC%A1%E4%B8%8D%E5%A6%82%E5%88%B7%E4%B8%80%E6%AC%A1%E6%96%87%E6%A1%A3%E6%9D%A5%E5%BE%97%E5%BF%AB)
  - [1. 全表扫描](#1-%E5%85%A8%E8%A1%A8%E6%89%AB%E6%8F%8F)
  - [2. 查询价格最贵的10个商品，只返回商品名字和单价](#2-%E6%9F%A5%E8%AF%A2%E4%BB%B7%E6%A0%BC%E6%9C%80%E8%B4%B5%E7%9A%8410%E4%B8%AA%E5%95%86%E5%93%81%E5%8F%AA%E8%BF%94%E5%9B%9E%E5%95%86%E5%93%81%E5%90%8D%E5%AD%97%E5%92%8C%E5%8D%95%E4%BB%B7)
  - [3. 按照商品名字筛选](#3-%E6%8C%89%E7%85%A7%E5%95%86%E5%93%81%E5%90%8D%E5%AD%97%E7%AD%9B%E9%80%89)
  - [4. 按照商品名字筛选2](#4-%E6%8C%89%E7%85%A7%E5%95%86%E5%93%81%E5%90%8D%E5%AD%97%E7%AD%9B%E9%80%892)
  - [5. 模糊查询和数值过滤](#5-%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2%E5%92%8C%E6%95%B0%E5%80%BC%E8%BF%87%E6%BB%A4)
  - [6. 多表联合查询](#6-%E5%A4%9A%E8%A1%A8%E8%81%94%E5%90%88%E6%9F%A5%E8%AF%A2)
  - [7. 其他](#7-%E5%85%B6%E4%BB%96)
  - [8. 图数据库真正优势在于不用递归也能轻易查出来一棵树](#8-%E5%9B%BE%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9C%9F%E6%AD%A3%E4%BC%98%E5%8A%BF%E5%9C%A8%E4%BA%8E%E4%B8%8D%E7%94%A8%E9%80%92%E5%BD%92%E4%B9%9F%E8%83%BD%E8%BD%BB%E6%98%93%E6%9F%A5%E5%87%BA%E6%9D%A5%E4%B8%80%E6%A3%B5%E6%A0%91)

## 你试那么多次,不如刷一次文档来得快

* https://neo4j.com/docs/cypher-manual/current/
* http://clojureneo4j.info/articles/guides.html

## 1. 全表扫描

```sql
SELECT p.*
FROM products as p;
```
```neo4j
MATCH (p:Product)
RETURN p;
```

## 2. 查询价格最贵的10个商品，只返回商品名字和单价
```sql
SELECT p.ProductName, p.UnitPrice
FROM products as p
ORDER BY p.UnitPrice DESC
LIMIT 10;
```
```neo4j
MATCH (p:Product)
RETURN p.productName, p.unitPrice
ORDER BY p.unitPrice DESC
LIMIT 10;
```

## 3. 按照商品名字筛选
```sql
SELECT p.ProductName, p.UnitPrice
FROM products AS p
WHERE p.ProductName = 'Chocolade';
```
```neo4j
MATCH (p:Product)
WHERE p.productName = "Chocolade"
RETURN p.productName, p.unitPrice;
```

```neo4j
## 其他的写法
MATCH (p:Product {productName:"Chocolade"})
RETURN p.productName, p.unitPrice;
```

## 4. 按照商品名字筛选2
```sql
SELECT p.ProductName, p.UnitPrice
FROM products as p
WHERE p.ProductName IN ('Chocolade','Chai');
```
```neo4j
MATCH (p:Product)
WHERE p.productName IN ['Chocolade','Chai']
RETURN p.productName, p.unitPrice;
```

## 5. 模糊查询和数值过滤
```sql
SELECT p.ProductName, p.UnitPrice
FROM products AS p
WHERE p.ProductName LIKE 'C%' AND p.UnitPrice > 100;
```
```neo4j
MATCH (p:Product)
WHERE p.productName STARTS WITH "C" AND p.unitPrice > 100
RETURN p.productName, p.unitPrice;
```

## 6. 多表联合查询
```sql
SELECT DISTINCT c.CompanyName
FROM customers AS c
JOIN orders AS o ON (c.CustomerID = o.CustomerID)
JOIN order_details AS od ON (o.OrderID = od.OrderID)
JOIN products AS p ON (od.ProductID = p.ProductID)
WHERE p.ProductName = 'Chocolade';
```
```neo4j
MATCH (p:Product {productName:"Chocolade"})<-[:PRODUCT]-(:Order)<-[:PURCHASED]-(c:Customer)
RETURN distinct c.companyName;
```

## 7. 其他

```sql
SELECT e.EmployeeID, count(*) AS Count
FROM Employee AS e
JOIN Order AS o ON (o.EmployeeID = e.EmployeeID)
GROUP BY e.EmployeeID
ORDER BY Count DESC LIMIT 10;
```
```neo4j
MATCH (:Order)<-[:SOLD]-(e:Employee)
RETURN e.name, count(*) AS cnt
ORDER BY cnt DESC LIMIT 10
```

## 8. 图数据库真正优势在于不用递归也能轻易查出来一棵树

关系型数据库需要递归才能查出来一棵树, 但是图数据不用

* [The Minimum Weight Spanning Tree algorithm](https://neo4j.com/docs/graph-algorithms/current/labs-algorithms/minimum-weight-spanning-tree/)

创建一个示例图(这里的数据就像GraphViz的dot描述的文件一样):

```CPYHER
MERGE (a:Place {id:"A"})
MERGE (b:Place {id:"B"})
MERGE (c:Place {id:"C"})
MERGE (d:Place {id:"D"})
MERGE (e:Place {id:"E"})
MERGE (f:Place {id:"F"})
MERGE (g:Place {id:"G"})

MERGE (d)-[:LINK {cost:4}]->(b)
MERGE (d)-[:LINK {cost:6}]->(e)
MERGE (b)-[:LINK {cost:1}]->(a)
MERGE (b)-[:LINK {cost:3}]->(c)
MERGE (a)-[:LINK {cost:2}]->(c)
MERGE (c)-[:LINK {cost:5}]->(e)
MERGE (f)-[:LINK {cost:1}]->(g);

```

最小权重生成树访问与起始节点位于同一连接组件中的所有节点，并返回该组件中所有关系最小的权重的生成树

最小权重生成树算法并写回结果:

```CPYHER
MATCH (n:Place {id:"D"})
CALL algo.spanningTree.minimum('Place', 'LINK', 'cost', id(n),
  {write:true, writeProperty:"MINST"})
YIELD loadMillis, computeMillis, writeMillis, effectiveNodeCount
RETURN loadMillis, computeMillis, writeMillis, effectiveNodeCount;

```

查询最小生成树:

```CPYHER
MATCH path = (n:Place {id:"D"})-[:MINST*]-()
WITH relationships(path) AS rels
UNWIND rels AS rel
WITH DISTINCT rel AS rel
RETURN startNode(rel).id AS source, endNode(rel).id AS destination, rel.cost AS cost
```

![](https://raw.githubusercontent.com/chanshunli/jim-emacs-fun-diff-mysql-neo4j/master/graphviz_dot_to_neo4j.png)
