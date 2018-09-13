##查询出所有广东省的市、县”做思路总结
- 首先得到数据库只有一个表
- 根据表的设计使用的只有一张表
- 如果要得到省直接通过where条件查询即可
- 得到市则需要使用递归的方式连接1次
- 自己关联自己因为里面有字段是关联上一级id
- 如果需要查询区县则需要连接2次
##编写代码

	select s3.id,s1.cityName,s2.cityName,s3.cityName from s_provinces s1
	left join s_provinces s2 on s2.parentId = s1.id
	left join s_provinces s3 on s3.parentId = s2.id #where s1.cityName = '广东省'
	union
	select s2.id,s1.cityName,s2.cityName,null from s_provinces s1
	join s_provinces s2 on s2.parentId = s1.id ;

- 使用union是因为有省有市但有可能区县为空。
###union
union和union all的区别是,union会自动压缩多个结果集合中的重复结果，而union all则将所有的结果全部显示出来，不管是不是重复。

[分表总结](https://www.cnblogs.com/dzcici/p/9642879.html)