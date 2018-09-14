# 针对于MYSQL进行分表

- 首先看一下这张表

lagou_position_source(源数据表)



![](https://img2018.cnblogs.com/blog/1409892/201809/1409892-20180913194600445-1521362007.png)
 



- 严重的数据冗余

4914591    西安        产品运营专员    数据服务,分类信息    5    8    3-5年    大专    运营|编辑|客服类 - 运营    全职    五险一金,周末双休    2018-07-26 15:05:05    
2018-07-26 07:29:00    105728    北佳信息    陕西北佳信息技术有限责任公司    150-500人    不需要融资

4914592    杭州        销售管理实习生    信息安全,数据服务    2    3    应届毕业生    本科    市场|商务|销售类 - 销售    实习    全员MAC,独角兽公司    2018-07-26 15:05:05    2018-07-26 07:29:00    6502    同盾科技    同盾科技有限公司    500-2000人    C轮

4914589    杭州        省区经理（浙江）    移动互联网,医疗健康    6    10    不限    大专    市场|商务|销售类 - 销售    全职    双休,工作灵活,大平台    2018-07-26 15:04:44    2018-07-26 07:29:00    284208    神中科技    杭州神中科技有限公司    50-150人    天使轮

4914587    北京        风控算法工程师    电子商务,金融    20    40    不限    本科    开发|测试|运维类 - 人工智能    全职    六险一金,团队氛围好    2018-07-26 15:04:43    2018-07-26 07:29:00    19243    趣店    趣分期（北京）信息技术有限公司    500-2000人    上市公司

4914588    上海        大数据分析    移动互联网,硬件    18    30    3-5年    本科    开发|测试|运维类 - 数据开发    全职    五险一金,正式编制,交通便利,节假日福利    2018-07-26 15:04:43    2018-07-26 07:29:00    146293    闻善科技    深圳闻善科技有限公司    50-150人    不需要融资
不满足范式

- 所有我们需要进行分表，同事迁移里面的数据

思路：首先源数据表 lagou_position_source

暂且我们直观的可以分出城市区域表(city)、公司表(company)、和职位表(position)

我们以简单粗暴的方式创建

### 首先来一张公司表

	drop table IF EXISTS lagou_company;
	create table lagou_company
    select  distinct company_id,company_short_name,
      company_size,company_full_name,financestage
    from lagou_position_new;
查询出来的数据

 ![](https://img2018.cnblogs.com/blog/1409892/201809/1409892-20180913200432407-272069060.png)
- 成功的分出来一张公司表

- 现在来分析城市表（根据原有数据是有局限的）

- 原因只是显示公司所在区域，下次添加一条记录，我需要地区根本无从选择

- 所以在此需要有一个完整的区域表

### 区域表
 ![](https://img2018.cnblogs.com/blog/1409892/201809/1409892-20180913200925018-1347649137.png)


此表也是需要分离

得到需要的字段（关联方式为id，parentId）话不多说同理

#### 分离方法

	drop table if exists lagou_city;
	create table lagou_city as
	select s3.id cid,s1.cityName province,s2.cityName city,s3.cityName district from s_provinces s1
	  right join s_provinces s2 on s2.parentId = s1.id
	  right join s_provinces s3 on s3.parentId = s2.id
	union
	select s2.id,s1.cityName,s2.cityName,null from s_provinces s1
	  right  join s_provinces s2 on s2.parentId = s1.id ;
最后得到city表

![](https://img2018.cnblogs.com/blog/1409892/201809/1409892-20180913201239185-1139783445.png)

最后剩下position 既然分离出去了必定要有关联

所以该删的删改加的加原有的city、district改为 position_city表的ID即可

方法
	
	drop table  if exists lagou_position;
	create table lagou_position as
	SELECT p.pid,p.company_id,c.cid,p.position,
	p.field,p.salary_min,p.salary_max,p.workyear,
	p.education,p.ptype,p.pnature,p.advantage,p.published_at,p.updated_at
	FROM (SELECT *
        FROM lagou_position_source
        WHERE district IS NULL) p
    JOIN lagou_city c ON (c.city LIKE concat(p.city, '%') AND c.district IS NULL )
	union
	select p.pid,p.company_id,c.cid,p.position,
	  p.field,p.salary_min,p.salary_max,p.workyear,
	  p.education,p.ptype,p.pnature,p.advantage,p.published_at,p.updated_at
	from (select * from lagou_position_source where district is not null)p
	  join lagou_city c on (c.city like concat(p.city,'%')
	and c.district like concat(p.district,'%'));
	lagou_position表

![](https://img2018.cnblogs.com/blog/1409892/201809/1409892-20180913201712122-1423106758.png)


结论：两个原（lagou_position_source）、（s_provinces）表分离成最后

	lagou_position;
	lagou_city ;
	lagou_company;
 