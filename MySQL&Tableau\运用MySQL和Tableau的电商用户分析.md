# 运用MySQL和Tableau的电商用户分析

# 一、项目简介

本项目基于公开美妆电商用户行为数据，围绕用户活跃、购买转化、留存复购、用户价值及商品流量与转化表现开展分析。使用MySQL完成数据清洗和字段标准化，并通过条件聚合、关联查询、子查询、视图及窗口函数构建转化漏斗、次日留存、5日留存、复购率等指标；进一步使用Tableau搭建分析看板，识别用户核心活跃时段、关键流失环节，为用户召回和商品转化优化提供数据支持。

# 二、数据来源

公开数据集：https://tianchi\.aliyun\.com/dataset/196209

# 三、数据清洗

user\_id（用户id）

item\_id（商品id）

behavior\_type（用户行为标签，1：浏览；2：收藏；3：加购物车；4：购买）

item\_category（商品类别id）

date（日期）

hour（时间）

user\_geohash（用户所在省份）

## 3\.1 修改列名

```SQL
ALTER TABLE customers_beauty_data    
    CHANGE f1 user_id int,    
    CHANGE f2 item_id int,    
    CHANGE f3 behavior_type varchar(5),    
    CHANGE f4 item_category int,    
    CHANGE f5 date varchar(255),    
    CHANGE f6 hour int,    
    CHANGE f7 user_geohash varchar(255)
```

## 3\.2 空值检查

```SQL
SELECT count(*) as 空值行   
FROM customers_beauty_data    
WHERE    
    user_id is null    
    or item_id is null    
    or item_category is null    
    or behavior_type is null    
    or date is null    
    or hour is null    
    or user_geohash is null
```

![image\.png](图片和附件/image%2012.png)

## 3\.3 去除重复项

```SQL
SELECT    
    user_id,    
    item_id,    
    date,    
    hour,    
    behavior_type,    
    user_geohash    
FROM    
    customers_beauty_data    
group by    
    user_id,    
    item_id,    
    date,    
    hour,    
    behavior_type,    
    user_geohash    
having    
    count(*) > 1
```

![image\.png](图片和附件/image%201.png)

创建一个新表（不含重复值）来替换掉原表，替换完后再删除新表

```SQL
CREATE TABLE temp_table as SELECT DISTINCT * FROM customers_beauty_data;    
TRUNCATE TABLE customers_beauty_data;    
INSERT into customers_beauty_data SELECT * from temp_table;    
DROP table temp_table;
```

# 四、数据分析

## 4\.1 用户角度

### 4\.1\.1 浏览深度

PV是浏览量，UV是浏览人数，PV/UV记为浏览深度

**日期维度**

```SQL
CREATE TABLE date_pv_uv    
(    
    date char(10),    
    pv_date int(9),    
    uv_date int(9),    
    pvuv_date DECIMAL(10,3)    
);    
INSERT INTO date_pv_uv    
SELECT    
    date,    
    count(IF(behavior_type=1,1,NULL)) pv_date,    
    count(DISTINCT user_id) uv_date,    
    round(count(IF(behavior_type=1,1,NULL)) /count(DISTINCT user_id),3) pvuv_date      
FROM    
    customers_beauty_data    
GROUP BY date  
ORDER by date
```

**时间维度**

```SQL
CREATE TABLE hour_pv_uv    
(    
    hour char(10),    
    pv_hour int(9),    
    uv_hour int(9),    
    pvuv_hour DECIMAL(10,3)    
);    
INSERT INTO hour_pv_uv    
SELECT    
    hour,    
    count(IF(behavior_type=1,1,NULL)) pv_hour,    
    count(DISTINCT user_id) uv_hours,    
    round(count(IF(behavior_type=1,1,NULL)) /count(DISTINCT user_id),3) pvuv_hours    
FROM    
    customers_beauty_data    
GROUP BY hour
ORDER by hour
```

### 4\.1\.2 行为时间序列

统计每天每时进行浏览、收藏、加入购物车、购买这四种行为的人数各有多少

```SQL
create table df_timeseries    
(    
    date char(10),    
    hour int(9),    
    pv int(9),    
    cart int(9),    
    fav int(9),    
    buy int(9)    
);    
insert into df_timeseries    
select    
    date,    
    hour,    
    count(if(behavior_type = 1,1,null)) as pv,    
    count(if(behavior_type = 2,1,null)) as fav,    
    count(if(behavior_type = 3,1,null)) as cart,    
    count(if(behavior_type = 4,1,null)) as buy    
from    
    customers_beauty_data    
group by    
    date,    
    hour    
order by    
    date,    
    hour;
```

### 4\.1\.3 地区分布

```SQL
CREATE TABLE df_geohash_distribution(
    user_geohash varchar(25),
    num_people int(10)
);

insert into df_geohash_distribution
select user_geohash,count(*) num_people
from customers_beauty_data
where behavior_type=4
group by user_geohash
```

![image\.png](图片和附件/image%208.png)

```SQL
create table df_timeseries    
  (    
    dates char(10),    
    hours int(9),    
    PV int(9),    
    CART int(9),    
    FAV int(9),    
    BUY int(9)    
  );    
insert into df_timeseries    
select    
  dates,    
  hours,    
  count(if(behavior_type = 'pv',1,null)) as PV,    
  count(if(behavior_type = 'cart',1,null)) as CART,    
  count(if(behavior_type = 'fav',1,null)) as FAV,    
  count(if(behavior_type = 'buy',1,null)) as BUY    
from    
  userbehavior    
group by    
  dates,    
  hours    
order by    
  dates asc,    
  hours asc;
```

### 4\.1\.4 复购率

统计不同用户在不同省份的购买次数

```SQL
CREATE TABLE buy_times    
(    
    user_id int, 
    user_geohash varchar(255),    
    times int(10)    
);    
INSERT INTO buy_times    
SELECT    
    user_id,    
    user_geohash,
    count(user_id) times    
FROM    
    customers_beauty_data    
WHERE    
    behavior_type=4    
GROUP BY    
    user_id,
    user_geohash                
order by 
    user_id,
    times
```

计算复购率

```SQL
SELECT 
    total_buyers,
    repeat_buyers,
    CONCAT(ROUND(100 * repeat_buyers / total_buyers, 2), '%') AS rate
FROM 
(
    SELECT 
        COUNT(DISTINCT user_id) AS total_buyers,
        COUNT(DISTINCT CASE WHEN total_times >= 2 THEN user_id END) AS repeat_buyers
    FROM (
        SELECT user_id, SUM(times) AS total_times
        FROM buy_times
        GROUP BY user_id
    ) totals 
) repurchase;
```

![image\.png](图片和附件/image%204.png)

### 4\.1\.5 转化率

按用户、省份、商品分组，统计各行为次数，生成浏览 / 收藏 / 加购 / 购买的 0\-1 行为序列路径

```SQL
CREATE VIEW customer_behavior_sequence AS    
SELECT    
    user_id,
    user_geohash,         
    item_id,
    COUNT(IF(behavior_type = 1, 1, NULL)) AS pv_times,      
    COUNT(IF(behavior_type = 2, 1, NULL)) AS fav_times,      
    COUNT(IF(behavior_type = 3, 1, NULL)) AS cart_times,      
    COUNT(IF(behavior_type = 4, 1, NULL)) AS buy_times,
    CONCAT(
        IF(COUNT(IF(behavior_type = 1, 1, NULL)) > 0, 1, 0),  
        IF(COUNT(IF(behavior_type = 2, 1, NULL)) > 0, 1, 0),  
        IF(COUNT(IF(behavior_type = 3, 1, NULL)) > 0, 1, 0),  
        IF(COUNT(IF(behavior_type = 4, 1, NULL)) > 0, 1, 0)   
    ) AS path  
FROM customers_beauty_data      
GROUP BY      
    user_id,    
    user_geohash,    
    item_id;
```

![image\.png](图片和附件/image.png)

统计【浏览→浏览\-收藏/加购→浏览\-收藏/加购\-购买】三层转化漏斗的行为频次

```SQL
CREATE TABLE df_customers_behavior    
(    
    behavior VARCHAR(25),    
    num int(10)
); 

INSERT INTO df_customers_behavior
SELECT '浏览', COUNT(*) FROM customer_behavior_sequence 
WHERE path IN ('1000','1001','1010','1011','1100','1101','1110','1111')
UNION ALL
SELECT '浏览-收藏/加购', COUNT(*) from customer_behavior_sequence 
WHERE path IN ('1010','1100','1110','1011','1101','1111')   
UNION ALL
SELECT '浏览-收藏/加购-购买', COUNT(*) from customer_behavior_sequence 
WHERE path IN ('1101','1011','1111')

```

![image\.png](图片和附件/image%2010.png)

### 4\.1\.6 留存率

计算次日留存和五日留存

```SQL
CREATE TABLE retention_rate   
(    
    date VARCHAR(25),    
    retention_1 FLOAT,
    retention_5 FLOAT
);    

INSERT INTO retention_rate     

SELECT    
    cbd1.date,    
    COUNT(CASE WHEN cbd2.date = DATE_ADD(cbd1.date, INTERVAL 1 DAY) THEN cbd2.user_id END) 
    / COUNT(DISTINCT cbd1.user_id) AS retention_1,
    
    COUNT(CASE WHEN cbd2.date = DATE_ADD(cbd1.date, INTERVAL 5 DAY) THEN cbd2.user_id END) 
    / COUNT(DISTINCT cbd1.user_id) AS retention_5        
FROM    
(    
    SELECT distinct user_id,date    
    FROM customers_beauty_data    
) cbd1    
LEFT JOIN    
(    
    SELECT DISTINCT user_id,date    
    FROM customers_beauty_data    
) cbd2    
ON cbd1.user_id=cbd2.user_id      
GROUP BY cbd1.date    
ORDER BY cbd1.date
```

![image\.png](图片和附件/image%2013.png)

### 4\.1\.7 RFM模型

R：最近一次消费的时间间隔

F：一定时间内的消费频率

M：一定时间内的消费金额

**消费时间间隔**

```SQL
create view r as
select user_id,max(date) as buy_date
from customers_beauty_data
where behavior_type=4
group by user_id
```

**消费频率**

```SQL
create view f as
select user_id,count(user_id) as buy_times
from customers_beauty_data
where behavior_type=4
group by user_id
```

**建立用户RFM模型**

购买日期范围为2023\-11\-18\-2023\-12\-18，购买次数有1，2，3，4，5，6，7，8，9，10，11，12，13，14，15，16，17，19，20，21，22，33，44，49，82

```SQL
CREATE TABLE df_rfm_result AS
SELECT
    t.user_id,
    t.buy_date AS recency,
    t.r_score,
    AVG(t.r_score) OVER () AS avg_r,
    t.buy_times AS frequency,
    t.f_score,
    AVG(t.f_score) OVER () AS avg_f
FROM (
    SELECT
        r.user_id,
        r.buy_date,
        f.buy_times,
        CASE
            WHEN r.buy_date BETWEEN '2023-12-12' AND '2023-12-18' THEN 100
            WHEN r.buy_date BETWEEN '2023-12-05' AND '2023-12-11' THEN 80
            WHEN r.buy_date BETWEEN '2023-11-28' AND '2023-12-04' THEN 60
            WHEN r.buy_date BETWEEN '2023-11-21' AND '2023-11-27' THEN 40
            ELSE 20
        END AS r_score,
        CASE
            WHEN f.buy_times >= 20 THEN 100
            WHEN f.buy_times BETWEEN 10 AND 19 THEN 80
            WHEN f.buy_times BETWEEN 5 AND 9 THEN 60
            WHEN f.buy_times BETWEEN 2 AND 4 THEN 40
            ELSE 20
        END AS f_score
    FROM r
    JOIN f USING (user_id)
) t;
```

**用户分类**

```SQL
create table df_rfm_categoryresult    
(    
    user_class varchar(5),    
    user_class_num int(9)    
);    
insert into df_rfm_categoryresult    
select    
    user_class,    
    count(*) as user_class_num    
from    
(    
select *,    
case    
    when (f_score >= avg_f and r_score >= avg_r) then '价值用户'    
    when (f_score >= avg_f and r_score < avg_r) then '保持用户'    
    when (f_score < avg_f and r_score >= avg_r) then '发展用户'    
    else '挽留用户'    
end 
as user_class    
from df_rfm_result         
) as g    
group by user_class
```

## 4\.2 商品角度

### 4\.2\.1 商品热度

**人气排行**

```SQL
CREATE TABLE df_category_popularity
(
    item_category int(10),      
    popularity int(20)      
);
INSERT INTO df_category_popularity
SELECT
    item_category,
    COUNT(item_category) AS popularity  
FROM customers_beauty_data
WHERE behavior_type = 1  
GROUP BY item_category
ORDER BY popularity DESC  
LIMIT 10; 
```

**热销品类**

```SQL
CREATE TABLE df_popular_category
(
    item_category int(10),
    category_hot int(20)
);
INSERT INTO df_popular_category
SELECT
    item_category,
    count(item_category) category_hot
FROM customers_beauty_data
WHERE behavior_type=4
GROUP BY item_category
ORDER BY category_hot DESC
LIMIT 10
```

### 4\.2\.2 商品特征

统计每个商品种类被浏览、收藏、加入购物车、购买的次数

```SQL
CREATE TABLE df_category_count    
(    
    item_category int(10),    
    pv int(10),    
    fav int(10),    
    cart int(10),    
    buy int(10)    
);    
INSERT INTO df_category_count    
SELECT    
    item_category,    
    COUNT(if(behavior_type=1,1,null)) pv,    
    COUNT(if(behavior_type=2,1,null)) fav,    
    COUNT(if(behavior_type=3,1,null)) cart,    
    COUNT(if(behavior_type=4,1,null)) buy    
FROM
    customers_beauty_data    
GROUP BY    
    item_category 
```

# 五、数据可视化

## 5\.1 流量类指标

### 5\.1\.1 用户每日浏览深度

![image\.png](图片和附件/image%202.png)

**现象描述**

11月下旬至12月初，PV 和 PV/UV 整体处于相对平稳状态。进入12月后开始小幅增长；12月8日以后增长明显，并在12月12日达到观察期峰值。活动结束后，PV 和 PV/UV 快速回落，并逐步恢复至常态水平。

相比之下，UV在整个观察期内波动较小，仅在12月12日前后出现较明显变化。该差异说明活动期间流量增长主要来自用户人均浏览次数增加。

**业务解释**

“双十二”预热和集中促销提高了用户的浏览意愿。PV显著增长而浏览UV相对稳定，意味着活动的“促活和加深浏览”效果较强，但“外部拉新”效果相对有限。

**运营建议**

1. 将活动运营拆分为预热期、活动期和返场期。预热期重点引导收藏和加购，活动日重点刺激支付，返场期用于召回未购买用户。

2. 在商品详情页增加肤质适配、成分说明、真实评价和常见问题，降低用户反复跳转比较的成本。

3. 若目标是拉新，应增加站外渠道来源和新老用户标签，单独评估 KOL、短视频和信息流广告带来的新增UV与首购率。

### 5\.1\.2 用户每时浏览深度

![image\.png](图片和附件/image%207.png)

**现象描述**

- 01:00—08:00：PV、UV和PV/UV均处于全天低位；

- 09:00—17:00：指标波动相对有限，整体保持稳定；

- 18:00以后：PV和PV/UV快速上升，在22:00左右达到峰值，随后回落。

**业务解释**

用户活跃时间与日常作息高度一致。工作和学习时段内，用户访问相对分散；晚间休闲时间更充足，用户更容易浏览美妆内容、观看直播、比较产品并产生消费。22:00附近是平台最重要的流量窗口。

**运营建议**

1. 将18:00—23:00设置为核心营销时段，集中安排直播、优惠券发放、消息推送和限时促销。

2. 17:00—18:00可作为预热时段，提前发送“开播提醒”、“购物车降价”和“库存提醒”。

3. 对已经收藏或加购但未购买的用户，可在21:00—22:30触发个性化召回。

4. 凌晨时段不建议进行大规模付费投放，可保留低成本自动化触达和客服机器人。

## 5\.2 用户行为分析

![image\.png](图片和附件/image%2011.png)

![image\.png](图片和附件/image%2014.png)

**现象描述**

从日期维度看，12月初以前，浏览、收藏、加购和购买行为整体保持相对平稳，未出现明显的大幅波动。进入12月后，各类用户行为开始逐渐增长，并在12月8日以后出现较明显的上升趋势。12月12日前后，各项用户行为均达到阶段性高峰。其中，浏览行为增长最为明显，收藏、加购和购买行为也同步上升。12月12日以后，各项行为快速回落，并逐渐恢复至日常水平。

从小时维度看，凌晨1:00—8:00，各项行为均处于全天较低水平；9:00—17:00，浏览、收藏、加购和购买行为逐渐恢复并保持相对稳定；18:00以后，各项行为明显增加，并在21:00—22:00前后达到较高水平；22:00以后，用户活跃度逐渐下降。

整体来看，18:00—23:00是用户浏览和购买行为较为集中的时间段，也是平台重要的流量与成交时段。此外，浏览行为的数量明显高于收藏、加购和购买行为，说明平台获得了较多商品曝光，但从浏览到购买的过程中仍存在较明显的用户流失。

**业务解释**

在活动预热阶段，用户通常会提前浏览商品、查看评价、对比价格，并通过收藏或加入购物车的方式保留购买意向，浏览、收藏和加购行为会率先增加。活动正式开始后，部分前期已经浏览、收藏或加购的用户完成购买，从而推动购买行为同步增长。活动结束后，各项行为迅速下降。平台能够依靠大型促销活动在短时间内获得较高流量，但活动带来的增长持续时间有限，日常运营对用户的吸引力仍有提升空间。

**运营建议**

1. 围绕促销周期开展分阶段运营

在活动预热阶段，应重点提升用户的收藏和加购意愿，通过预约提醒、优惠券预领取、商品预售和提前加购等方式积累潜在购买用户；在活动正式开展阶段，应重点促进用户完成支付，通过发送限时优惠、库存提醒和优惠倒计时信息，缩短用户决策时间；在活动结束后，应及时开展用户召回，减少活动流量的快速流失。

2. 重点利用晚间流量高峰

3. 加强浏览至购买环节的转化

## 5\.3 用户留存分析

![image\.png](图片和附件/image%2015.png)

**现象描述**

次日留存率整体高于5日留存率。用户在发生浏览、收藏、加购或购买行为后，短期内再次访问平台的可能性相对较高，但随着时间间隔延长，能够持续回访的用户比例逐渐下降。

**业务解释**

次日留存高于5日留存，平台具有一定的短期用户召回能力。但5日留存相对较低，说明部分用户的访问行为具有临时性。促销活动、优惠券或热门商品能够在短期内提高用户活跃度。很大一部分原因与美妆产品本身也有很大联系，用户不一定在短时间内重复购买。

**运营建议**

1. 建立分阶段用户召回机制。 对首次活跃用户在次日推送商品浏览记录、收藏商品动态和使用攻略；在第3至第5天推送优惠提醒、同类商品推荐或内容种草，延长用户活跃周期。 

2. 对不同用户行为进行留存运营。 对仅浏览用户推送商品内容和评价；对收藏用户推送降价提醒；对加购未购买用户提供限时优惠；对已购买用户推送使用教程、搭配建议和补货提醒。  

## 5\.4 用户价值定位分析

![image\.png](图片和附件/image%209.png)

**运营建议**

1. 价值用户重点维护。 提供会员等级、专属客服、新品优先试用、生日礼遇和定向补货提醒，增强用户忠诚度。

2. 保持用户作为重点召回对象。 根据其历史购买品类和购买间隔发送补货提醒、专属优惠券或新品信息。 

3. 发展用户重点促进第二次购买。 首次购买后的7至30天是用户培养的重要阶段，可通过关联商品推荐、复购券、使用教程和评价返积分等方式促进再次购买。 

4. 挽留用户控制营销成本。 采用短信、普通优惠券等方式进行低成本批量唤醒。多次触达仍无响应的用户，应减少高频推送，避免增加营销成本并造成用户反感。 

## 5\.5 购买地区分析

![image\.png](图片和附件/image%206.png)

**运营建议**

1. 针对地域特征进行差异化选品。 结合气候、季节和消费偏好调整商品，例如在北方干燥地区重点推广保湿修护产品，在高日照地区加强防晒类产品推广。

## 5\.6 商品热度分析

![image\.png](图片和附件/image%205.png)

![image\.png](图片和附件/image%2016.png)

**现象描述**

人气排行反映用户最经常浏览的品类，热销排行则反映实际购买次数较多的品类。

**运营建议**

1. 对高浏览、高购买的核心优势品类保持搜索及推荐位置，并通过关联商品推荐提高连带购买。 

2. 对高关注低转化的品类优化主图、试色内容、成分说明、用户评价和价格优惠。 

3. 对低曝光高购买品类增加首页推荐力度，通过搜索曝光、直播展示和内容种草增加流量，有助于带来更高增量。 

4. 对低曝光低购买的品类进行扶持、调整或下架。

## 5\.7 商品特征分析

![image\.png](图片和附件/image%203.png)

**业务解释**

第一象限商品同时具有较高的浏览量和购买量，其用户关注度高，实际交易量也多；第二象限商品浏览量相对较低，但购买次数较高。该部分商品虽然获得的曝光有限，却具有较强的成交能力。此类商品可能是目标用户需求明确的小众商品，也可能是老用户复购；第三象限商品的浏览量和购买量均处于较低水平；第四象限商品有较高浏览量，但购买次数相对较低，该部分商品能够吸引用户点击和关注，但未能有效推动用户完成购买。

**运营建议**

1. 对核心热销商品应持续增加曝光并保持供应稳定性。

2. 对高转化潜力商品应重点解决曝光不足的问题，向具有相关需求的用户进行精准推送。

3. 对长期表现不佳的商品优化卖点、价格和目标人群，必要时采取降价清仓、组合销售、福利赠品或下架处理。

4. 对高关注低转化商品，应重点优化商品详情页、用户评价、价格优惠和购买流程，并通过增加同类商品或平价替代品推荐，降低用户决策成本，促进浏览流量向实际购买转化。

# 六、仪表板搭建

![仪表板\.png](图片和附件/仪表板.png)



