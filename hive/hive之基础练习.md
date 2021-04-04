# 1. hive基本业务实操—badou

1. 将orders和order_products_prior建表入hive
2. 每个用户有多少个订单
3. 每个用户平均每个订单是多少商品
4. 每个用户在一周中的购买订单的分布 --列转行 case when
5. 【把days_since_prior_order => date】平均一天（购买那几天）购买商品数量

6. 每个用户最喜爱购买的三个product是什么，最终表结构可以是3个列，或者是一个字符串

