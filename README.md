
# Customer Lifetime Value with SQL and Python

<img width="742" height="602" src="https://www.sqlservertutorial.net/wp-content/uploads/SQL-Server-Sample-Database.png" data-src="https://www.sqlservertutorial.net/wp-content/uploads/SQL-Server-Sample-Database.png" alt="" class="wp-image-96 ls-is-cached lazyloaded" data-srcset="https://www.sqlservertutorial.net/wp-content/uploads/SQL-Server-Sample-Database.png 742w, https://www.sqlservertutorial.net/wp-content/uploads/SQL-Server-Sample-Database-300x243.png 300w" data-sizes="(max-width: 742px) 100vw, 742px" sizes="(max-width: 742px) 100vw, 742px" srcset="https://www.sqlservertutorial.net/wp-content/uploads/SQL-Server-Sample-Database.png 742w, https://www.sqlservertutorial.net/wp-content/uploads/SQL-Server-Sample-Database-300x243.png 300w">

<p>LTV = customer lifetime value<p>
<p>ARPU = average monthly recurring revenue per user <p>
<p>churn_rate = the rate at which we are losing customers (inverse of retention)<p>
<p>
<p>
<img alt="" class="oy tw t u v ip aj c" width="300" height="73" role="presentation" src="https://miro.medium.com/max/375/0*Z2Z5j3DO2d126NuA.png" srcset="https://miro.medium.com/max/345/0*Z2Z5j3DO2d126NuA.png 276w, https://miro.medium.com/max/375/0*Z2Z5j3DO2d126NuA.png 300w" sizes="300px">



```python
from sqlalchemy.engine import URL
import sqlalchemy
server = '*' 
database = 'bikes' 
username = '*' 
password = '*' 
connection_string='DRIVER={ODBC Driver 17 for SQL Server};SERVER='+server+';DATABASE='+database+';UID='+username+';PWD='+ password

connection_url = URL.create("mssql+pyodbc", query={"odbc_connect": connection_string})

sqlalchemy.create_engine(connection_url)
```




    Engine(mssql+pyodbc://?odbc_connect=DRIVER%3D%7BODBC+Driver+17+for+SQL+Server%7D%3BSERVER%3Dlocalhost%5CSQLEXPRESS%3BDATABASE%3Dbikes%3BUID%3Dsa%3BPWD%3DPg123456789)




```python
%load_ext sql
```


```python
%sql mssql+pyodbc://?odbc_connect=DRIVER%3D%7BODBC+Driver+17+for+SQL+Server%7D%3BSERVER%3Dlocalhost%5CSQLEXPRESS%3BDATABASE%3Dpractice%3BUID%3Dsa%3BPWD%3DPg123456789
```


```python
%config SqlMagic.displaycon = False
```

# analyzing a bike store orders during 2016

### monthly purchase of customers:


```python
%%sql
SELECT top 10
customer_id,MONTH(order_date) as month,sum((quantity*list_price)*(1-discount)) as price
  FROM (select * from [bikes].[sales].[orders] where year(order_date)=2016) as orders2016
  inner join [bikes].[sales].[order_items] on order_items.order_id=orders2016.order_id group by customer_id,MONTH(order_date)
```

    Done.
    




<table>
    <tr>
        <th>customer_id</th>
        <th>month</th>
        <th>price</th>
    </tr>
    <tr>
        <td>57</td>
        <td>1</td>
        <td>4205.9641</td>
    </tr>
    <tr>
        <td>60</td>
        <td>1</td>
        <td>7199.9820</td>
    </tr>
    <tr>
        <td>80</td>
        <td>1</td>
        <td>6822.5814</td>
    </tr>
    <tr>
        <td>91</td>
        <td>1</td>
        <td>6816.9225</td>
    </tr>
    <tr>
        <td>94</td>
        <td>1</td>
        <td>9442.5048</td>
    </tr>
    <tr>
        <td>164</td>
        <td>1</td>
        <td>1139.9810</td>
    </tr>
    <tr>
        <td>175</td>
        <td>1</td>
        <td>1349.9820</td>
    </tr>
    <tr>
        <td>236</td>
        <td>1</td>
        <td>4157.9724</td>
    </tr>
    <tr>
        <td>252</td>
        <td>1</td>
        <td>8587.3970</td>
    </tr>
    <tr>
        <td>258</td>
        <td>1</td>
        <td>437.0907</td>
    </tr>
</table>



### average revenue per user(ARPU):


```python
%%sql
with purchases as
(SELECT
customer_id,MONTH(order_date) as month,sum((quantity*list_price)*(1-discount)) as price
  FROM (select * from [bikes].[sales].[orders] where year(order_date)=2016) as orders2016
  inner join [bikes].[sales].[order_items] on order_items.order_id=orders2016.order_id group by customer_id,MONTH(order_date))
select avg(price) as ARPU from purchases
```

    Done.
    




<table>
    <tr>
        <th>ARPU</th>
    </tr>
    <tr>
        <td>3834.721212</td>
    </tr>
</table>



### table of orders for every cusomer consecutively


```python
%%sql
with purchase as 
(SELECT customer_id,month(order_date) as order_month
  FROM [bikes].[sales].[orders] group by customer_id,month(order_date) )
  select top 10 customer_id,order_month,lead(order_month,1) over (partition by customer_id order by order_month) as next_purchase from purchase
```

    Done.
    




<table>
    <tr>
        <th>customer_id</th>
        <th>order_month</th>
        <th>next_purchase</th>
    </tr>
    <tr>
        <td>1</td>
        <td>1</td>
        <td>2</td>
    </tr>
    <tr>
        <td>1</td>
        <td>2</td>
        <td>3</td>
    </tr>
    <tr>
        <td>1</td>
        <td>3</td>
        <td>4</td>
    </tr>
    <tr>
        <td>1</td>
        <td>4</td>
        <td>5</td>
    </tr>
    <tr>
        <td>1</td>
        <td>5</td>
        <td>8</td>
    </tr>
    <tr>
        <td>1</td>
        <td>8</td>
        <td>9</td>
    </tr>
    <tr>
        <td>1</td>
        <td>9</td>
        <td>10</td>
    </tr>
    <tr>
        <td>1</td>
        <td>10</td>
        <td>11</td>
    </tr>
    <tr>
        <td>1</td>
        <td>11</td>
        <td>12</td>
    </tr>
    <tr>
        <td>1</td>
        <td>12</td>
        <td>None</td>
    </tr>
</table>



### <p>churned=cusomers who not return from one month to the next <p> returned=cusomers who return from one month to the next



```python
%%sql
with purchase as 
(SELECT customer_id,month(order_date) as order_month
  FROM [bikes].[sales].[orders] group by customer_id,month(order_date) ),
customer_lags as
  (select customer_id,order_month,lead(order_month,1) over (partition by customer_id order by order_month) as next_purchase from purchase)
select top 10 customer_id,order_month+1 as month,case when next_purchase-order_month<2 then 'returned' else 'churned' end as 'status' from customer_lags where order_month <= 11

```

    Done.
    




<table>
    <tr>
        <th>customer_id</th>
        <th>month</th>
        <th>status</th>
    </tr>
    <tr>
        <td>1</td>
        <td>2</td>
        <td>returned</td>
    </tr>
    <tr>
        <td>1</td>
        <td>3</td>
        <td>returned</td>
    </tr>
    <tr>
        <td>1</td>
        <td>4</td>
        <td>returned</td>
    </tr>
    <tr>
        <td>1</td>
        <td>5</td>
        <td>returned</td>
    </tr>
    <tr>
        <td>1</td>
        <td>6</td>
        <td>churned</td>
    </tr>
    <tr>
        <td>1</td>
        <td>9</td>
        <td>returned</td>
    </tr>
    <tr>
        <td>1</td>
        <td>10</td>
        <td>returned</td>
    </tr>
    <tr>
        <td>1</td>
        <td>11</td>
        <td>returned</td>
    </tr>
    <tr>
        <td>1</td>
        <td>12</td>
        <td>returned</td>
    </tr>
    <tr>
        <td>2</td>
        <td>2</td>
        <td>returned</td>
    </tr>
</table>



### count number of customers in each category by month


```python
%%sql
with purchase as 
(SELECT customer_id,month(order_date) as order_month
  FROM [bikes].[sales].[orders] group by customer_id,month(order_date) ),
customer_lags as
  (select customer_id,order_month,lead(order_month,1) over (partition by customer_id order by order_month) as next_purchase from purchase),
customer_status as (select customer_id,order_month+1 as month,case when next_purchase-order_month<2 then 'returned' else 'churned' end as 'status' from customer_lags where order_month <= 11)
select top 10 month,status,count(distinct customer_id) as count from customer_status group by month,status order by 1
```

    Done.
    




<table>
    <tr>
        <th>month</th>
        <th>status</th>
        <th>count</th>
    </tr>
    <tr>
        <td>2</td>
        <td>returned</td>
        <td>395</td>
    </tr>
    <tr>
        <td>2</td>
        <td>churned</td>
        <td>363</td>
    </tr>
    <tr>
        <td>3</td>
        <td>returned</td>
        <td>440</td>
    </tr>
    <tr>
        <td>3</td>
        <td>churned</td>
        <td>295</td>
    </tr>
    <tr>
        <td>4</td>
        <td>churned</td>
        <td>296</td>
    </tr>
    <tr>
        <td>4</td>
        <td>returned</td>
        <td>561</td>
    </tr>
    <tr>
        <td>5</td>
        <td>returned</td>
        <td>405</td>
    </tr>
    <tr>
        <td>5</td>
        <td>churned</td>
        <td>542</td>
    </tr>
    <tr>
        <td>6</td>
        <td>churned</td>
        <td>342</td>
    </tr>
    <tr>
        <td>6</td>
        <td>returned</td>
        <td>262</td>
    </tr>
</table>



### pivot table


```python
%%sql
with purchase as 
(SELECT customer_id,month(order_date) as order_month
  FROM [bikes].[sales].[orders] group by customer_id,month(order_date) ),
customer_lags as
  (select customer_id,order_month,lead(order_month,1) over (partition by customer_id order by order_month) as next_purchase from purchase),
customer_status as (select customer_id,order_month+1 as month,case when next_purchase-order_month<2 then 'returned' else 'churned' end as 'status' from customer_lags where order_month <= 11),
churn as  
(select month,status,count(distinct customer_id) as counts from customer_status group by month,status)
select top 10 * from
(
select * from churn
) AS SourceTable  
PIVOT  
(  
  sum(counts)
  FOR status IN ([returned],[churned])  
) AS PivotTable
```

    Done.
    




<table>
    <tr>
        <th>month</th>
        <th>returned</th>
        <th>churned</th>
    </tr>
    <tr>
        <td>2</td>
        <td>395</td>
        <td>363</td>
    </tr>
    <tr>
        <td>3</td>
        <td>440</td>
        <td>295</td>
    </tr>
    <tr>
        <td>4</td>
        <td>561</td>
        <td>296</td>
    </tr>
    <tr>
        <td>5</td>
        <td>405</td>
        <td>542</td>
    </tr>
    <tr>
        <td>6</td>
        <td>262</td>
        <td>342</td>
    </tr>
    <tr>
        <td>7</td>
        <td>250</td>
        <td>334</td>
    </tr>
    <tr>
        <td>8</td>
        <td>267</td>
        <td>338</td>
    </tr>
    <tr>
        <td>9</td>
        <td>291</td>
        <td>354</td>
    </tr>
    <tr>
        <td>10</td>
        <td>307</td>
        <td>348</td>
    </tr>
    <tr>
        <td>11</td>
        <td>256</td>
        <td>415</td>
    </tr>
</table>



### monthly churn rate


```python
%%sql
with purchase as 
(SELECT customer_id,month(order_date) as order_month
  FROM [bikes].[sales].[orders] group by customer_id,month(order_date) ),
customer_lags as
  (select customer_id,order_month,lead(order_month,1) over (partition by customer_id order by order_month) as next_purchase from purchase),
customer_status as (select customer_id,order_month+1 as month,case when next_purchase-order_month<2 then 'returned' else 'churned' end as 'status' from customer_lags where order_month <= 11),
churn as  
(select month,status,count(distinct customer_id) as counts from customer_status group by month,status)
select *,cast(churned as float)/(cast(returned as float)+cast(churned as float)) as churn_rate from
(
select top 10 * from churn
) AS SourceTable  
PIVOT  
(  
  sum(counts)
  FOR status IN ([returned],[churned])  
) AS PivotTable
```

    Done.
    




<table>
    <tr>
        <th>month</th>
        <th>returned</th>
        <th>churned</th>
        <th>churn_rate</th>
    </tr>
    <tr>
        <td>3</td>
        <td>440</td>
        <td>295</td>
        <td>0.4013605442176871</td>
    </tr>
    <tr>
        <td>4</td>
        <td>None</td>
        <td>296</td>
        <td>None</td>
    </tr>
    <tr>
        <td>6</td>
        <td>None</td>
        <td>342</td>
        <td>None</td>
    </tr>
    <tr>
        <td>7</td>
        <td>250</td>
        <td>None</td>
        <td>None</td>
    </tr>
    <tr>
        <td>8</td>
        <td>267</td>
        <td>338</td>
        <td>0.5586776859504132</td>
    </tr>
    <tr>
        <td>10</td>
        <td>307</td>
        <td>None</td>
        <td>None</td>
    </tr>
    <tr>
        <td>11</td>
        <td>None</td>
        <td>415</td>
        <td>None</td>
    </tr>
    <tr>
        <td>12</td>
        <td>226</td>
        <td>None</td>
        <td>None</td>
    </tr>
</table>



### avergare churn rate


```python
%%sql
with purchase as 
(SELECT customer_id,month(order_date) as order_month
  FROM [bikes].[sales].[orders] group by customer_id,month(order_date) ),
customer_lags as
  (select customer_id,order_month,lead(order_month,1) over (partition by customer_id order by order_month) as next_purchase from purchase),
customer_status as (select customer_id,order_month+1 as month,case when next_purchase-order_month<2 then 'returned' else 'churned' end as 'status' from customer_lags where order_month <= 11),
churn as  
(select month,status,count(distinct customer_id) as counts from customer_status group by month,status)
select avg(cast(churned as float)/(cast(returned as float)+cast(churned as float))) as average_churn_rate from
(
select top 10 * from churn
) AS SourceTable  
PIVOT  
(  
  sum(counts)
  FOR status IN ([returned],[churned])  
) AS PivotTable
```

    Done.
    




<table>
    <tr>
        <th>average_churn_rate</th>
    </tr>
    <tr>
        <td>0.5256821197661526</td>
    </tr>
</table>



#  LTV
It is a simple calculation, then, to estimate LTV: we have average ARPU and avergae churn, so we just divide one by the other!
<p>\$3834.721212/0.5256 = $7295.89<p>
