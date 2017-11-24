#### 使用BI Studio平台对电子商务类业务进行报表展示

本文章综合电商类应用场景，对电商类业务的流量获取、行为浏览到交易评价，再到售后，展示了这类业务最关心的图表，挖掘报表价值。

报表的最终呈现如下图所示

![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-overview1.png)
![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-overview2.png)
![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-overview3.png)


##### 关注的指标

1. 基础运营指标

主要的关注的事件有：
* 登录app
* 注册账户
* 退出app
* 搜索商品
* 添加商品到购物车
* 提交订单
* 取消订单
* 支付订单
* 申请退货

根据以上事件可以得出：

* 注册率、app新增用户、活跃用户(时间序列图)
* top 10订单城市（饼状图）
* top 10订单商品（饼状图）
* 可以得出用户对产品的评论标签云（标签云）

![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-ratio.png)
![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-add.png)
![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-active.png)
![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-city.png)
![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-order.png)
![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-comment.png)


2. 用户分群

* 不同年龄-购买力的分布图（气泡图）
* 用户属性分析（饼状图）

![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-age-buy.png)
![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-user.png)


3. 商品热力分析

* 不同商品的销量的热力分析（热力图）
![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-product.png)


4. 自定义查询

bi studio提供sql lab用以开放用户自定义查询的能力。
![](http://pandora-kibana.qiniu.com/bi-demo-ecommerce-sqllab.png)



