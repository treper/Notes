####Spark SQL

sql语句


	hiveContext.hql("create table temp_temp4 as select cnt,count(1) as ct from (select user_id,count(distinct ds) as cnt from pms.pms_user_data group by user_id) dayct group by cnt order by ct desc")



create table temp_temp4 as select cnt,count(1) as ct from (select end_user_id,count(distinct order_id) as cnt from  default.so_item group by end_user_id) dayct group by cnt order by ct desc

29,753,977

select count(distinct order_id) from default.so_item;
201，079，605


select tmp.cnt,count(distinct user_id) as c from (select count(distinct ds) as cnt,user_id from pms_user_data group by user_id) tmp group by tmp.cnt order by c desc




join得到表，pms_user_buy_data 耗时8231.047 seconds

####Spark SQL performance test

将hive表的hdfs文件转换为`parquet`格式

	val sqlContext = new org.apache.spark.sql.SQLContext(sc)
	import sqlContext.createSchemaRDD
	val hiveContext = new org.apache.spark.sql.hive.HiveContext(sc)
	hiveContext.hql("select * from pms.pms_user_buy_data").saveAsParquetFile("hdfs://yhd-jqhadoop2.int.yihaodian.com:8022/user/pms/xq/pms_user_buy_data.parquet")



|SQL|SQL|hive|spark on hdfs|spark on parquet|spark on cached parquet|
|:-:|:-:|:-:|:-:|:-:|:-:|
|Select||280|99|74|29|
|GroupBy|587|362|239|177|
|Join|8231||||



spark on hdfs sql

	group by

	create table pms.temp_ubd_gp as select city_id,gender,promotion,product_id,count(1) as cnt from pms.pms_user_buy_data where city_id is not null and gender is not null and product_id is not null and promotion is not null group by city_id,gender,promotion,product_id order by cnt desc

    cost 362 sec

17:50:57 17:55:56

18:09:16 13:15 

	join

	create table pms.temp_ubd_gp_cn as select a.city_id,a.gender,b.product_cname,a.cnt from pms.temp_ubd_gp a join default.product b on a.product_id=b.id order by a.cnt desc

	cost 355 sec



join

create table pms.pms_user_buy_data1 as select b.end_user_id as end_user_id,case when j.is_leaf=2 then j.id when j.is_leaf=1 then j.parent_so_id else null end as root_so_id,c.is_yihaodian as is_yihaodian,b.is_promote as is_promote,b.order_item_price as order_item_price,b.order_item_num as order_item_num,b.product_id as product_id,a.l1 as c1,a.l2 as c2,a.l3 as c3,a.l4 as c4,k.product_brand_id as product_brand_id,d.gender as gender,case when d.gender='f' and d.younglady='1' then 'lady' when d.gender='f' and d.younglady='0' then 'mum' else null end as younglady,g.promotion as promotion,split(b.ds,'-')[1] as month,b.ds as day,d.city as city_id,i.city as city,i.city_level as city_level,i.province as province,h.season as season,h.change_season as change_season,h.feel as feel,h.weather as weather from pms.pms_bought b left outer join default.so j on (j.id=b.so_id) left outer join default.product k on (k.id=b.product_id and b.product_id is not null) left outer join pms.product_category a on(a.product_id=b.product_id and a.product_id is not null and b.end_user_id is not null and a.l3 is not null) left outer join pms.merchant c on (c.id=b.merchant_id) left outer join pms.temp_userprofile_demo d on (d.user=b.end_user_id and d.user is not null) left outer join pms.city_discretize i on (i.id=d.city) left outer join pms.date_discretize g on (g.day=b.ds) left outer join pms.weather_discretize h on (h.day=b.ds and h.city=d.city);

cost 9578.762 seconds

