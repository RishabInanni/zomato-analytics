<h1>user_behavior_analysis.sql:</h1>

what is total amount each customer spent on zomato

select a.userid, sum(b.price) total_amt_spend from sales a inner join product b on 
a.product_id =b.product_id
group by a.userid;

<h1>visit_frequency.sql:</h1>

How many days has each customers visited zomato

select userid , count(distinct created_date) distinct_date from sales group by userid;

<h1>first_purchase_by_customer.sql:</h1>

what was the first product purchased by each customer

select* from(
select *,rank() over(partition by userid  order by created_date  )rnk from sales
) a where rnk =1;

<h1>most_purchased_item.sql:</h1>

what is the most purchased item on menu and how many times was it purchased by all the customers;

select userid,count(product_id) cnt  from sales where product_id=
(select product_id  from sales group by product_id order by count(product_id) desc limit 1) group by userid;


<h1>most_popular_item.sql:</h1>

which item was most popular for each customer

select* from(
select *, rank() over (partition by userid order by cnt desc)rnk from
(select userid,product_id,count(product_id) cnt from sales group by userid,product_id)a)b
where rnk =1;

<h1>first_purchase_after_membership.sql:</h1>

which item was purchased first by customer after they became member

select * from
(select c.*,rank() over(partition by userid order by created_date)rnk from
(select a.userid,a.created_date,a.product_id,b.gold_signup_date from sales a inner join goldusers_signup b on a.userid=b.userid and created_date>=gold_signup_date)c)d where rnk =1;

<h1>last_purchase_before_membership.sql:</h1>

which item was purchased just before the customer became the member

select * from(
select c.*, rank() over (partition by userid order by created_date desc) rnk from
(
select a.userid,a.created_date,a.product_id,b.gold_signup_date from sales a inner join goldusers_signup b on a.userid=b.userid and created_date<= gold_signup_date)c)d
where rnk =1;

</h1>total_amt_spent_before_membership.sql:</h1>

what is the total amount spent for each member before they became member

select userid, count(created_date) order_purchased ,sum(price) total_amt_spend from(
select c.*,d.price from
(
select a.userid,a.created_date,a.product_id from sales a inner join goldusers_signup b on a.userid=b.userid and created_date<=gold_signup_date)c inner join product d on c.product_id =d.product_id)e
group by userid;







<h1>points_collected_by_customers.sql:</h1>

if buying each product generates points eg: for p1 rs5 = 1 zomato point,for p1 rs10 = 5 zomato point,for p3 rs 2 = 1 zomato point
calculate points collected by each customers and for which product most points has been given till date

select userid,sum(total_points)*2.5 total_money_earned from
(select e.*,amt/points total_points from
(select d.*,case when product_id=1 then 5 when product_id =2 then 2 when product_id=3 then 5 else 0 end as points from
(select c.userid,c.product_id,sum(price) amt from
(select a.*,b.price from sales a inner join product b on a.product_id=b.product_id )c
group by userid,product_id)d)e)f  group by userid order by userid asc;

<h1>total_points_by_products.sql:</h1>

total no of points by each products

select product_id,sum(total_points) total_points_earned from
(select e.*,amt/points total_points from
(select d.*,case when product_id=1 then 5 when product_id =2 then 2 when product_id=3 then 5 else 0 end as points from
(select c.userid,c.product_id,sum(price) amt from
(select a.*,b.price from sales a inner join product b on a.product_id=b.product_id )c
group by userid,product_id)d)e)f  group by product_id order by product_id asc;





<h1>most_points_generating_product.sql:</h1>

product having highest no of points

select * from
(select *, rank() over(order by total_points_earned desc) rnk from
(select product_id,sum(total_points) total_points_earned from
(select e.*,amt/points total_points from
(select d.*,case when product_id=1 then 5 when product_id =2 then 2 when product_id=3 then 5 else 0 end as points from
(select c.userid,c.product_id,sum(price) amt from
(select a.*,b.price from sales a inner join product b on a.product_id=b.product_id )c
group by userid,product_id)d)e)f  group by product_id order by product_id asc)g)h where rnk =1;

<h1>points_earned_in_first_year.sql:</h1>

in the first 1 year after customer joins the gold programme including their joininng date irrespective of what customers have purchased they earn 5 zomato points for every 10 rs spennt who earne more?and what was their price erning in 1st year

select c.*,d.price*0.5 total_points_earned from
(select a.*,b.gold_signup_date from sales a inner join goldusers_signup b on a.userid=b.userid and created_date >=gold_signup_date and created_date<= gold_signup_date + INTERVAL '1 year')c
inner join product d on c.product_id=d.product_id;


<h1>transaction_rank_by_date.sql:</h1>

rank all the transaction of the customer

select *, rank() over(partition by userid order by created_date asc) rnk from sales;





<h1>transaction_rank_gold_members.sql:</h1>

rank all the transaction for each member when they are a zomato gold member for every non gold member they are ranked as na

SELECT
e.*,
CASE WHEN rnk = 0 THEN 'na' ELSE CAST(rnk AS VARCHAR) END AS rnkk
FROM (
SELECT
c.*,
RANK() OVER (PARTITION BY userid ORDER BY created_date DESC) AS rnk
FROM (
SELECT
a.userid,
a.created_date,
a.product_id,
b.gold_signup_date
FROM
sales a
LEFT JOIN goldusers_signup b ON a.userid = b.userid AND a.created_date > b.gold_signup_date
) c
) e;

