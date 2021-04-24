### 每组找前x类型

> 表: `customers`
>
> ```
> +---------------+---------+
> | Column Name   | Type    |
> +---------------+---------+
> | customer_id   | int     |
> | name          | varchar |
> +---------------+---------+
> customer_id 是该表主键。
> 该表包含消费者的id和姓名.
> ```
>
> 表: `orders`
>
> ```
> +---------------+---------+
> | Column Name   | Type    |
> +---------------+---------+
> | order_id      | int     |
> | order_date    | date    |
> | customer_id   | int     |
> | product_id    | int     |
> +---------------+---------+
> order_id 是该表主键。
> 该表包含消费者产生的订单编号，订单日期，顾客id和商品id。
> 不会有商品被相同的用户在一天内下单超过一次。
> ```
>
> 编写一个sql语句，找到每个用户最近三笔订单。若用户订单少于3笔，则返回该用户的全部订单，结果返回用户名customer_name，订单编号order_id和订单日期order_date，以custromer_name升序，order_date降序排列。

* 思路：子查询，要算出某人成绩在第几名，可以转换成：算出他一共比多少人成绩高

```mysql
select c.name as customer_name,o.order_id as order_id,o.order_date as order_date from customers c ,orders o
where c.customer_id=o.customer_id and (
  select count(1) from orders o1 
  where o1.customer_id=c.customer_id and o1.order_date>=o.order_date
)<=3
order by customer_name,o.order_date desc
```

### 缺省值处理

> 一位用户，既可以作为卖家也可以作为买家参与一场交易，以下为相关的表结构，
>
> 表: `users`
>
> ```
> +----------------+---------+
> | Column Name    | Type    |
> +----------------+---------+ 
> | user_id        | int     |
> | join_date      | date    |
> | favorite_brand | varchar |
> +----------------+---------+
> user_id 是该表的主键
> 表中包含一位某网站用户的个人id，注册时间和最喜欢的品牌。
> ```
>
> 表: `orders`
>
> ```
> ---------------+---------+
> | Column Name   | Type    |
> +---------------+---------+
> | order_id      | int     |
> | order_date    | date    |
> | item_id       | int     |
> | buyer_id      | int     |
> | seller_id     | int     |
> +---------------+---------+
> order_id 是该表的主键
> 该表包含订单的id，日期，商品id，买方id和卖方id
> ```
>
> 编写一个sql语句，查询每个用户的注册日期以及在2019年作为买家的订单总数，结果返回用户编号user_id，注册日期join_date和订单数量orders_in_2019，以user_id升序排列。

* 坑：这里要求查询每个用户，但是不一定没个用户在orders表里，left join以后可能会有null值
* 思路：coalesce函数进行处理

```mysql
select u.user_id,u.join_date,coalesce(o.cnt,0) as orders_in_2019 from users u left join 
(select buyer_id,count(order_id) as cnt from orders where order_date like '2019%'  group by buyer_id) o
on u.user_id=o.buyer_id
order by u.user_id
```

