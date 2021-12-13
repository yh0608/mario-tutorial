# SQL入门教程

## 1. 数据库表

某公司的商品信息表记录了有哪些商品，订单表中记录了商品销售的流水，商品信息表的“good_id”与订单表中的“good_id”一一对应。

商品表如下

![image-20210929191748907](https://tva1.sinaimg.cn/large/008i3skNly1guxprpw2ggj62hk086wfc02.jpg)

订单表如下

![截屏2021-09-29 下午7.10.41](https://tva1.sinaimg.cn/large/008i3skNly1guxpl7cmzfj62hi0h0q6u02.jpg)

## 2. 入门问题

**知识点：select语句、where限制条件、group by聚合函数**

问题1：检索出id为“1”，支付时间在2021年7月的订单明细

输出结果格式

![image-20210929192303507](https://tva1.sinaimg.cn/large/008i3skNly1guxpx6436tj62i00hctcj02.jpg)

问题2：计算id为“1”的商品在2021年7、8月的销售数量和销售额

输出结果格式

![image-20210929192641793](https://tva1.sinaimg.cn/large/008i3skNly1guxq0ych5aj62hk06it9i02.jpg)

代码

问题1

```sql
select
	*
from demo.order_info
where good_id = 1
and pay_dt between '2021-07-01' and '2021-07-31'
```

问题2

```sql
select
	substr(a.pay_dt,1,7) as `支付月份`,
	count(distinct a.order_id) as `销售数量`,
	sum(a.pay_amt) as `销售额`
from
(
select
	order_id,
	pay_dt,
	pay_amt
from demo.order_info
where good_id = 1
and pay_dt between '2021-07-01' and '2021-08-30'
)a 
group by substr(a.pay_dt,1,7)
```



## 3. 进阶问题

问题3：智能硬件和文具分别在7、8月的销量和销售额

**知识点：join表关联、case when条件函数**

输出结果格式

![image-20211008140007921](https://tva1.sinaimg.cn/large/008i3skNly1gv7v60739mj624e06mjs302.jpg)

```sql
select
    a.category,
		count(distinct case when (b.pay_dt between '2021-07-01' and '2021-07-31') then b.order_id else null end ) as `7月销售数量`,
		count(distinct case when (b.pay_dt between '2021-08-01' and '2021-08-30') then b.order_id else null end ) as `8月销售数量`
from
(
    select
        good_id,
        category
    from demo.good_info
)a
join
(
  select
    good_id,
    order_id,
    user_id,
    pay_dt
  from demo.order_info
  where pay_dt between '2021-07-01' and '2021-08-30'
)b 
on a.good_id = b.good_id
group by a.category
```



问题4：统计2021年每个区域、每月、销售金额TOP１销售人员

**知识点：窗口函数row_number()**

输出结果格式

![image-20211008141153958](https://tva1.sinaimg.cn/large/008i3skNly1gv7vi6m9c2j624g08g75902.jpg)

代码

```sql
select
    b.order_province,
    b.seller,
    b.sum_amt
from 
(
    select
        a.order_province,
        a.seller,
        a.sum_amt,
        row_number() over(partition by a.order_province order by a.sum_amt desc) as rnk
    from
    (
        select
            order_province,
            seller,
            sum(pay_amt) as sum_amt
        from demo.order_info
        where pay_dt between '2021-07-01' and '2021-07-31'
        and order_status = 2
        group by order_province, seller
    )a 
)b 
where b.rnk = 1
```

