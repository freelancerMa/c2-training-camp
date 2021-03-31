                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               snap_report_database                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## 报告时间段: ```2021-03-31 10:15:40.405542``` ~ ```2021-03-31 11:17:59.148349```    
   
 ## 一、数据库性能分析
   
 ### 1. 当前数据库 TOP 10 SQL : total_cpu_time
   
 calls | total_ms | rows | shared_blks_hit | shared_blks_read | shared_blks_dirtied | shared_blks_written | local_blks_hit | local_blks_read | local_blks_dirtied | shared_blks_written | temp_blks_read | temp_blks_written | blk_read_time | blk_write_time | query
 ---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
 2 | 112.552 | 0 | 1288 | 236 | 80 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | ```CREATE EXTENSION IF NOT EXISTS pg_stat_statements;```
 1 | 61.104 | 0 | 2381 | 57 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | ```create table snap_pg_many_indexes_rel as select 1::int8 snap_id, now() snap_ts, current_database(), t2.nspname, t1.relname, pg_relation_size(t1.oid), t3.idx_cnt from pg_class t1, pg_namespace t2, (select indrelid,count(*) idx_cnt from pg_index group by 1 having count(*)>4) t3 where t1.oid=t3.indrelid and t1.relnamespace=t2.oid and pg_relation_size(t1.oid)/1024/1024.0>10 order by t3.idx_cnt desc;```
 1 | 56.382 | 0 | 2680 | 18 | 4 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | ```create table snap_pg_table_bloat as select 1::int8 snap_id, now() snap_ts, \r   current_database() AS db, schemaname, tablename, reltuples::bigint AS tups, relpages::bigint AS pages, otta,\r   ROUND(CASE WHEN otta=0 OR sml.relpages=0 OR sml.relpages=otta THEN 0.0 ELSE sml.relpages/otta::numeric END,1) AS tbloat,\r   CASE WHEN relpages < otta THEN 0 ELSE relpages::bigint - otta END AS wastedpages,\r   CASE WHEN relpages < otta THEN 0 ELSE bs*(sml.relpages-otta)::bigint END AS wastedbytes,\r   CASE WHEN relpages < otta THEN $$0 bytes$$::text ELSE (bs*(relpages-otta))::bigint &#124;&#124; $$ bytes$$ END AS wastedsize,\r   iname, ituples::bigint AS itups, ipages::bigint AS ipages, iotta,\r   ROUND(CASE WHEN iotta=0 OR ipages=0 OR ipages=iotta THEN 0.0 ELSE ipages/iotta::numeric END,1) AS ibloat,\r   CASE WHEN ipages < iotta THEN 0 ELSE ipages::bigint - iotta END AS wastedipages,\r   CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta) END AS wastedibytes,\r   CASE WHEN ipages < iotta THEN $$0 bytes$$ ELSE (bs*(ipages-iotta))::bigint &#124;&#124; $$ bytes$$ END AS wastedisize,\r   CASE WHEN relpages < otta THEN\r     CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta::bigint) END\r     ELSE CASE WHEN ipages < iotta THEN bs*(relpages-otta::bigint)\r       ELSE bs*(relpages-otta::bigint + ipages-iotta::bigint) END\r   END AS totalwastedbytes\r FROM (\r   SELECT\r     nn.nspname AS schemaname,\r     cc.relname AS tablename,\r     COALESCE(cc.reltuples,0) AS reltuples,\r     COALESCE(cc.relpages,0) AS relpages,\r     COALESCE(bs,0) AS bs,\r     COALESCE(CEIL((cc.reltuples*((datahdr+ma-\r       (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::float)),0) AS otta,\r     COALESCE(c2.relname,$$?$$) AS iname, COALESCE(c2.reltuples,0) AS ituples, COALESCE(c2.relpages,0) AS ipages,\r     COALESCE(CEIL((c2.reltuples*(datahdr-12))/(bs-20::float)),0) AS iotta \r   FROM\r      pg_class cc\r   JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname <> $$information_schema$$\r   LEFT JOIN\r   (\r     SELECT\r       ma,bs,foo.nspname,foo.relname,\r       (datawidth+(hdr+ma-(case when hdr%ma=0 THEN ma ELSE hdr%ma END)))::numeric AS datahdr,\r       (maxfracsum*(nullhdr+ma-(case when nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2\r     FROM (\r       SELECT\r         ns.nspname, tbl.relname, hdr, ma, bs,\r         SUM((1-coalesce(null_frac,0))*coalesce(avg_width, 2048)) AS datawidth,\r         MAX(coalesce(null_frac,0)) AS maxfracsum,\r         hdr+(\r           SELECT 1+count(*)/8\r           FROM pg_stats s2\r           WHERE null_frac<>0 AND s2.schemaname = ns.nspname AND s2.tablename = tbl.relname\r         ) AS nullhdr\r       FROM pg_attribute att \r       JOIN pg_class tbl ON att.attrelid = tbl.oid\r       JOIN pg_namespace ns ON ns.oid = tbl.relnamespace \r       LEFT JOIN pg_stats s ON s.schemaname=ns.nspname\r       AND s.tablename = tbl.relname\r       AND s.inherited=false\r       AND s.attname=att.attname,\r       (\r         SELECT\r           (SELECT current_setting($$block_size$$)::numeric) AS bs,\r             CASE WHEN SUBSTRING(SPLIT_PART(v, $$ $$, 2) FROM $$#"[0-9]+.[0-9]+#"%$$ for $$#$$)\r               IN ($$8.0$$,$$8.1$$,$$8.2$$) THEN 27 ELSE 23 END AS hdr,\r           CASE WHEN v ~ $$mingw32$$ OR v ~ $$64-bit$$ THEN 8 ELSE 4 END AS ma\r         FROM (SELECT version() AS v) AS foo\r       ) AS constants\r       WHERE att.attnum > 0 AND tbl.relkind=$$r$$\r       GROUP BY 1,2,3,4,5\r     ) AS foo\r   ) AS rs\r   ON cc.relname = rs.relname AND nn.nspname = rs.nspname\r   LEFT JOIN pg_index i ON indrelid = cc.oid\r   LEFT JOIN pg_class c2 ON c2.oid = i.indexrelid\r ) AS sml order by wastedbytes desc limit 10;```
 2 | 50.842 | 0 | 778 | 18 | 6 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | ```create table snap_pg_tbs_size as select 1::int8 snap_id, now() snap_ts, spcname, pg_tablespace_location(oid), pg_size_pretty(pg_tablespace_size(oid)) from pg_tablespace where spcname<>'pg_global' order by pg_tablespace_size(oid) desc;```
 2 | 36.012 | 0 | 900 | 12 | 4 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | ```create table snap_pg_stat_database as select 1::int8 snap_id, now() snap_ts, datname,round(100*(xact_rollback::numeric/(case when xact_commit > 0 then xact_commit else 1 end + xact_rollback)),2)&#124;&#124;$$ %$$ rollback_ratio, round(100*(blks_hit::numeric/(case when blks_read>0 then blks_read else 1 end + blks_hit)),2)&#124;&#124;$$ %$$ hit_ratio, blk_read_time, blk_write_time, conflicts, deadlocks from pg_stat_database;```
 2 | 31.526 | 0 | 1486 | 46 | 28 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | ```create table snap_list (id serial8 primary key, snap_ts timestamp, snap_level text);```
 1 | 26.34 | 0 | 2464 | 2 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | ```create table snap_pg_index_bloat as select 1::int8 snap_id, now() snap_ts, \r   current_database() AS db, schemaname, tablename, reltuples::bigint AS tups, relpages::bigint AS pages, otta,\r   ROUND(CASE WHEN otta=0 OR sml.relpages=0 OR sml.relpages=otta THEN 0.0 ELSE sml.relpages/otta::numeric END,1) AS tbloat,\r   CASE WHEN relpages < otta THEN 0 ELSE relpages::bigint - otta END AS wastedpages,\r   CASE WHEN relpages < otta THEN 0 ELSE bs*(sml.relpages-otta)::bigint END AS wastedbytes,\r   CASE WHEN relpages < otta THEN $$0 bytes$$::text ELSE (bs*(relpages-otta))::bigint &#124;&#124; $$ bytes$$ END AS wastedsize,\r   iname, ituples::bigint AS itups, ipages::bigint AS ipages, iotta,\r   ROUND(CASE WHEN iotta=0 OR ipages=0 OR ipages=iotta THEN 0.0 ELSE ipages/iotta::numeric END,1) AS ibloat,\r   CASE WHEN ipages < iotta THEN 0 ELSE ipages::bigint - iotta END AS wastedipages,\r   CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta) END AS wastedibytes,\r   CASE WHEN ipages < iotta THEN $$0 bytes$$ ELSE (bs*(ipages-iotta))::bigint &#124;&#124; $$ bytes$$ END AS wastedisize,\r   CASE WHEN relpages < otta THEN\r     CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta::bigint) END\r     ELSE CASE WHEN ipages < iotta THEN bs*(relpages-otta::bigint)\r       ELSE bs*(relpages-otta::bigint + ipages-iotta::bigint) END\r   END AS totalwastedbytes\r FROM (\r   SELECT\r     nn.nspname AS schemaname,\r     cc.relname AS tablename,\r     COALESCE(cc.reltuples,0) AS reltuples,\r     COALESCE(cc.relpages,0) AS relpages,\r     COALESCE(bs,0) AS bs,\r     COALESCE(CEIL((cc.reltuples*((datahdr+ma-\r       (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::float)),0) AS otta,\r     COALESCE(c2.relname,$$?$$) AS iname, COALESCE(c2.reltuples,0) AS ituples, COALESCE(c2.relpages,0) AS ipages,\r     COALESCE(CEIL((c2.reltuples*(datahdr-12))/(bs-20::float)),0) AS iotta \r   FROM\r      pg_class cc\r   JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname <> $$information_schema$$\r   LEFT JOIN\r   (\r     SELECT\r       ma,bs,foo.nspname,foo.relname,\r       (datawidth+(hdr+ma-(case when hdr%ma=0 THEN ma ELSE hdr%ma END)))::numeric AS datahdr,\r       (maxfracsum*(nullhdr+ma-(case when nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2\r     FROM (\r       SELECT\r         ns.nspname, tbl.relname, hdr, ma, bs,\r         SUM((1-coalesce(null_frac,0))*coalesce(avg_width, 2048)) AS datawidth,\r         MAX(coalesce(null_frac,0)) AS maxfracsum,\r         hdr+(\r           SELECT 1+count(*)/8\r           FROM pg_stats s2\r           WHERE null_frac<>0 AND s2.schemaname = ns.nspname AND s2.tablename = tbl.relname\r         ) AS nullhdr\r       FROM pg_attribute att \r       JOIN pg_class tbl ON att.attrelid = tbl.oid\r       JOIN pg_namespace ns ON ns.oid = tbl.relnamespace \r       LEFT JOIN pg_stats s ON s.schemaname=ns.nspname\r       AND s.tablename = tbl.relname\r       AND s.inherited=false\r       AND s.attname=att.attname,\r       (\r         SELECT\r           (SELECT current_setting($$block_size$$)::numeric) AS bs,\r             CASE WHEN SUBSTRING(SPLIT_PART(v, $$ $$, 2) FROM $$#"[0-9]+.[0-9]+#"%$$ for $$#$$)\r               IN ($$8.0$$,$$8.1$$,$$8.2$$) THEN 27 ELSE 23 END AS hdr,\r           CASE WHEN v ~ $$mingw32$$ OR v ~ $$64-bit$$ THEN 8 ELSE 4 END AS ma\r         FROM (SELECT version() AS v) AS foo\r       ) AS constants\r       WHERE att.attnum > 0 AND tbl.relkind=$$r$$\r       GROUP BY 1,2,3,4,5\r     ) AS foo\r   ) AS rs\r   ON cc.relname = rs.relname AND nn.nspname = rs.nspname\r   LEFT JOIN pg_index i ON indrelid = cc.oid\r   LEFT JOIN pg_class c2 ON c2.oid = i.indexrelid\r ) AS sml order by wastedibytes desc limit 10;```
 2 | 18.458 | 0 | 820 | 26 | 8 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | ```create table snap_pg_stat_activity as select 1::int8 snap_id, now() snap_ts, state, count(*) from pg_stat_activity group by 1,2,3;```
 2 | 15.802 | 0 | 354 | 4 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | ```create table snap_pg_conn_stats as select 1::int8 snap_id, now() snap_ts, max_conn,used,res_for_super,max_conn-used-res_for_super res_for_normal from (select count(*) used from pg_stat_activity) t1,(select setting::int res_for_super from pg_settings where name=$$superuser_reserved_connections$$) t2,(select setting::int max_conn from pg_settings where name=$$max_connections$$) t3;```
 2 | 14.426 | 0 | 750 | 18 | 12 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | ```create table snap_pg_db_role_setting as select 1::int8 snap_id, now() snap_ts, * from pg_db_role_setting;```
   
 #### 建议
   
 检查SQL是否有优化空间, 配合auto_explain插件在csvlog中观察LONG SQL的执行计划是否正确.  
   
 ### 2. TOP 10 size 表统计信息
   
 current_database | nspname | relname | relkind | pg_relation_size | seq_scan | seq_tup_read | idx_scan | idx_tup_fetch | n_tup_ins | n_tup_upd | n_tup_del | n_tup_hot_upd | n_live_tup | n_dead_tup
 ---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
 ```test``` | ```pg_catalog``` | ```pg_proc``` | r | 580 kB | 8 | 22584 | 940 | 1141 | 12 | 1 | 0 | 1 | 6.0000000000000000 | 0.50000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_attribute``` | r | 452 kB | 279 | 13356 | 13423 | 34750 | 880 | 1 | 0 | 1 | 1648.5000000000000000 | 0.50000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_depend``` | r | 448 kB | 48 | 353184 | 1326 | 1194 | 257 | 0 | 0 | 0 | 128.5000000000000000 | 0.00000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_description``` | r | 272 kB | 0 | 0 | 37 | 4 | 1 | 2 | 0 | 2 | 0.50000000000000000000 | 1.00000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_collation``` | r | 232 kB | 0 | 0 | 32 | 18 | 0 | 0 | 0 | 0 | 0.00000000000000000000 | 0.00000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_statistic``` | r | 148 kB | 2 | 768 | 1287 | 1426 | 50 | 82 | 0 | 4 | 25.0000000000000000 | 40.5000000000000000
 ```test``` | ```pg_catalog``` | ```pg_operator``` | r | 120 kB | 0 | 0 | 614 | 1923 | 0 | 0 | 0 | 0 | 0.00000000000000000000 | 0.00000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_rewrite``` | r | 96 kB | 12 | 1404 | 195 | 193 | 1 | 0 | 0 | 0 | 0.50000000000000000000 | 0.00000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_class``` | r | 88 kB | 17352 | 4276741 | 14251 | 13891 | 89 | 24 | 0 | 22 | 202.5000000000000000 | 2.5000000000000000
 ```test``` | ```pg_catalog``` | ```pg_type``` | r | 84 kB | 42 | 15666 | 1496 | 869 | 111 | 0 | 0 | 0 | 235.0000000000000000 | 0.00000000000000000000
   
 #### 说明
   
 seq_scan, 全表扫描次数  
   
 seq_tup_read, 全表扫描实际一共读取了多少条记录, 如果平均每次读取的记录数不多, 可能是limit语句造成的  
   
 idx_scan, 索引扫描次数  
   
 idx_tup_fetch, 索引扫描实际获取的记录数, 如果平均每次读取记录数很多, 说明数据库倾向使用索引扫描, 建议观察随机IO的性能看情况调整  
   
 n_tup_ins, 统计周期内, 插入了多少条记录  
   
 n_tup_upd, 统计周期内, 更新了多少条记录  
   
 n_tup_hot_upd, 统计周期内, HOT更新(指更新后的记录依旧在当前PAGE)了多少条记录  
   
 n_live_tup, 该表有多少可用数据  
   
 n_dead_tup, 该表有多少垃圾数据  
   
 #### 建议
   
 经验值: 单表超过10GB, 并且这个表需要频繁更新 或 删除+插入的话, 建议对表根据业务逻辑进行合理拆分后获得更好的性能, 以及便于对膨胀索引进行维护; 如果是只读的表, 建议适当结合SQL语句进行优化.  
   
 ### 3. 全表扫描统计 , 平均实际扫描记录数排名前10的表
   
 current_database | nspname | relname | relkind | pg_relation_size | seq_scan | seq_tup_read | idx_scan | idx_tup_fetch | n_tup_ins | n_tup_upd | n_tup_del | n_tup_hot_upd | n_live_tup | n_dead_tup
 ---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
 ```test``` | ```pg_catalog``` | ```pg_depend``` | r | 448 kB | 48 | 353184 | 1326 | 1194 | 257 | 0 | 0 | 0 | 128.5000000000000000 | 0.00000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_proc``` | r | 580 kB | 8 | 22584 | 940 | 1141 | 12 | 1 | 0 | 1 | 6.0000000000000000 | 0.50000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_type``` | r | 84 kB | 42 | 15666 | 1496 | 869 | 111 | 0 | 0 | 0 | 235.0000000000000000 | 0.00000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_class``` | r | 88 kB | 17352 | 4276741 | 14251 | 13891 | 89 | 24 | 0 | 22 | 202.5000000000000000 | 2.5000000000000000
 ```test``` | ```pg_catalog``` | ```pg_cast``` | r | 16 kB | 10 | 2130 | 2291 | 274 | 0 | 0 | 0 | 0 | 0.00000000000000000000 | 0.00000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_statistic``` | r | 148 kB | 2 | 768 | 1287 | 1426 | 50 | 82 | 0 | 4 | 25.0000000000000000 | 40.5000000000000000
 ```test``` | ```pg_catalog``` | ```pg_rewrite``` | r | 96 kB | 12 | 1404 | 195 | 193 | 1 | 0 | 0 | 0 | 0.50000000000000000000 | 0.00000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_attribute``` | r | 452 kB | 279 | 13356 | 13423 | 34750 | 880 | 1 | 0 | 1 | 1648.5000000000000000 | 0.50000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_index``` | r | 32 kB | 205 | 3973 | 7011 | 6888 | 23 | 0 | 0 | 0 | 11.5000000000000000 | 0.00000000000000000000
 ```test``` | ```pg_catalog``` | ```pg_ts_dict``` | r | 8192.0000000000000000 bytes | 2 | 32 | 0 | 0 | 0 | 0 | 0 | 0 | 0.00000000000000000000 | 0.00000000000000000000
   
 #### 说明
   
 seq_scan, 全表扫描次数  
   
 seq_tup_read, 全表扫描实际一共读取了多少条记录, 如果平均每次读取的记录数不多, 可能是limit语句造成的  
   
 idx_scan, 索引扫描次数  
   
 idx_tup_fetch, 索引扫描实际获取的记录数, 如果平均每次读取记录数很多, 说明数据库倾向使用索引扫描, 建议观察随机IO的性能看情况调整  
   
 n_tup_ins, 统计周期内, 插入了多少条记录  
   
 n_tup_upd, 统计周期内, 更新了多少条记录  
   
 n_tup_hot_upd, 统计周期内, HOT更新(指更新后的记录依旧在当前PAGE)了多少条记录  
   
 n_live_tup, 该表有多少可用数据  
   
 n_dead_tup, 该表有多少垃圾数据  
   
 #### 建议
   
 平均扫描的记录数如果很多, 建议找到SQL, 并针对性的创建索引(统计分析需求除外).  
   
 ### 4. 未命中buffer , 热表统计
   
 current_database | schemaname | relname | heap_blks_read | heap_blks_hit | idx_blks_read | idx_blks_hit | toast_blks_read | toast_blks_hit | tidx_blks_read | tidx_blks_hit
 ---|---|---|---|---|---|---|---|---|---|---
 ```test``` | ```pg_catalog``` | ```pg_proc``` | 278 | 1295 | 186 | 1815 | 5 | 22 | 4 | 46
 ```test``` | ```pg_catalog``` | ```pg_rewrite``` | 64 | 261 | 20 | 199 | 63 | 58 | 20 | 186
 ```test``` | ```pg_catalog``` | ```pg_statistic``` | 83 | 1061 | 25 | 1636 | 12 | 16 | 10 | 25
 ```test``` | ```pg_catalog``` | ```pg_constraint``` | 18 | 147 | 40 | 15 | 0 | 0 | 0 | 0
 ```test``` | ```pg_catalog``` | ```pg_db_role_setting``` | 0 | 0 | 26 | 37657 | 0 | 0 | 0 | 0
 ```test``` | ```pg_catalog``` | ```pg_description``` | 11 | 2 | 15 | 66 | 0 | 0 | 0 | 0
 ```test``` | ```pg_catalog``` | ```pg_attrdef``` | 1 | 4 | 12 | 162 | 0 | 0 | 0 | 0
 ```test``` | ```pg_catalog``` | ```pg_trigger``` | 0 | 0 | 12 | 422 | 0 | 0 | 0 | 0
 ```test``` | ```pg_catalog``` | ```pg_shdescription``` | 2 | 0 | 4 | 2 | 0 | 0 | 0 | 0
 ```test``` | ```pg_catalog``` | ```pg_shseclabel``` | 0 | 0 | 4 | 62 | 0 | 0 | 0 | 0
   
 #### 建议
   
 如果热表的命中率很低, 说明需要增加shared buffer, 添加内存.  
   
 ### 5. 未命中&命中buffer , 热表统计
   
 current_database | schemaname | relname | heap_blks_read | heap_blks_hit | idx_blks_read | idx_blks_hit | toast_blks_read | toast_blks_hit | tidx_blks_read | tidx_blks_hit
 ---|---|---|---|---|---|---|---|---|---|---
 ```test``` | ```pg_catalog``` | ```pg_db_role_setting``` | 0 | 0 | 26 | 37657 | 0 | 0 | 0 | 0
 ```test``` | ```pg_catalog``` | ```pg_proc``` | 278 | 1295 | 186 | 1815 | 5 | 22 | 4 | 46
 ```test``` | ```pg_catalog``` | ```pg_statistic``` | 83 | 1061 | 25 | 1636 | 12 | 16 | 10 | 25
 ```test``` | ```pg_catalog``` | ```pg_rewrite``` | 64 | 261 | 20 | 199 | 63 | 58 | 20 | 186
 ```test``` | ```pg_catalog``` | ```pg_trigger``` | 0 | 0 | 12 | 422 | 0 | 0 | 0 | 0
 ```test``` | ```pg_catalog``` | ```pg_seclabel``` | 0 | 0 | 2 | 274 | 0 | 0 | 0 | 0
 ```test``` | ```pg_catalog``` | ```pg_constraint``` | 18 | 147 | 40 | 15 | 0 | 0 | 0 | 0
 ```test``` | ```pg_catalog``` | ```pg_attrdef``` | 1 | 4 | 12 | 162 | 0 | 0 | 0 | 0
 ```test``` | ```pg_catalog``` | ```pg_description``` | 11 | 2 | 15 | 66 | 0 | 0 | 0 | 0
 ```test``` | ```pg_catalog``` | ```pg_shseclabel``` | 0 | 0 | 4 | 62 | 0 | 0 | 0 | 0
   
 #### 建议
   
 如果热表的命中率很低, 说明需要增加shared buffer, 添加内存.  
   
 ### 6. 未命中 , 热索引统计
   
 current_database | schemaname | relname | indexrelname | idx_blks_read | idx_blks_hit
 ---|---|---|---|---|---
 ```test``` | ```pg_catalog``` | ```pg_attribute``` | ```pg_attribute_relid_attnum_index``` | 161 | 32774
 ```test``` | ```pg_catalog``` | ```pg_proc``` | ```pg_proc_proname_args_nsp_index``` | 94 | 415
 ```test``` | ```pg_catalog``` | ```pg_proc``` | ```pg_proc_oid_index``` | 92 | 1400
 ```test``` | ```pg_catalog``` | ```pg_class``` | ```pg_class_oid_index``` | 85 | 31457
 ```test``` | ```pg_catalog``` | ```pg_amproc``` | ```pg_amproc_fam_proc_index``` | 72 | 13339
 ```test``` | ```pg_catalog``` | ```pg_class``` | ```pg_class_relname_nsp_index``` | 65 | 1452
 ```test``` | ```pg_catalog``` | ```pg_database``` | ```pg_database_datname_index``` | 52 | 18724
 ```test``` | ```pg_catalog``` | ```pg_database``` | ```pg_database_oid_index``` | 52 | 37266
 ```test``` | ```pg_catalog``` | ```pg_operator``` | ```pg_operator_oprname_l_r_n_index``` | 49 | 556
 ```test``` | ```pg_catalog``` | ```pg_index``` | ```pg_index_indexrelid_index``` | 48 | 10375
   
 #### 建议
   
 如果热索引的命中率很低, 说明需要增加shared buffer, 添加内存.  
   
 ### 7. 未命中&命中buffer , 热索引统计
   
 current_database | schemaname | relname | indexrelname | idx_blks_read | idx_blks_hit
 ---|---|---|---|---|---
 ```test``` | ```pg_catalog``` | ```pg_authid``` | ```pg_authid_oid_index``` | 32 | 48725
 ```test``` | ```pg_catalog``` | ```pg_db_role_setting``` | ```pg_db_role_setting_databaseid_rol_index``` | 26 | 37657
 ```test``` | ```pg_catalog``` | ```pg_database``` | ```pg_database_oid_index``` | 52 | 37266
 ```test``` | ```pg_catalog``` | ```pg_attribute``` | ```pg_attribute_relid_attnum_index``` | 161 | 32774
 ```test``` | ```pg_catalog``` | ```pg_class``` | ```pg_class_oid_index``` | 85 | 31457
 ```test``` | ```pg_catalog``` | ```pg_database``` | ```pg_database_datname_index``` | 52 | 18724
 ```test``` | ```pg_catalog``` | ```pg_amproc``` | ```pg_amproc_fam_proc_index``` | 72 | 13339
 ```test``` | ```pg_catalog``` | ```pg_index``` | ```pg_index_indexrelid_index``` | 48 | 10375
 ```test``` | ```pg_catalog``` | ```pg_opclass``` | ```pg_opclass_oid_index``` | 48 | 8772
 ```test``` | ```pg_catalog``` | ```pg_depend``` | ```pg_depend_reference_index``` | 43 | 3103
   
 #### 建议
   
 如果热索引的命中率很低, 说明需要增加shared buffer, 添加内存.  
   
 ### 8. 上次巡检以来未使用，或者使用较少的索引
   
 current_database | schemaname | relname | indexrelname | idx_scan | idx_tup_read | idx_tup_fetch | pg_size_pretty
 ---|---|---|---|---|---
   
 #### 建议
   
 建议和应用开发人员确认后, 删除不需要的索引.  
   
 ### 9. 索引数超过4并且SIZE大于10MB的表
   
 current_database | schemaname | relname | pg_size_pretty | idx_cnt
 ---|---|---|---|---
   
 #### 建议
   
 索引数量太多, 影响表的增删改性能, 建议检查是否有不需要的索引.  
   
 建议检查pg_stat_all_tables(n_tup_ins,n_tup_upd,n_tup_del,n_tup_hot_upd), 如果确实非常频繁, 建议检查哪些索引是不需要的.  
   
 ## 二、数据库空间使用分析
   
 ### 1. 用户对象占用空间的柱状图
   
 snap_ts | current_database | this_buk_no | rels_in_this_buk | buk_min | buk_max
 ---|---|---|---|---|---
 ```2021-03-31 11:15:40.405542+08``` | ```test``` | 1 | 8 | 0 bytes | 0 bytes
 ```2021-03-31 11:15:40.405542+08``` | ```test``` | 6 | 17 | 8192 bytes | 8192 bytes
 ```2021-03-31 11:15:40.405542+08``` | ```test``` | 10 | 2 | 16 kB | 16 kB
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | 1 | 15 | 0 bytes | 0 bytes
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | 2 | 21 | 8192 bytes | 8192 bytes
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | 4 | 1 | 16 kB | 16 kB
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | 7 | 2 | 32 kB | 32 kB
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | 9 | 2 | 40 kB | 40 kB
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | 10 | 1 | 48 kB | 48 kB
   
 #### 建议
   
 纵览用户对象大小的柱状分布图, 单容量超过10GB的对象(指排除TOAST的空间还超过10GB)，建议分区, 目前建议使用pg_pathman插件.  
   
 ## 三、数据库垃圾分析
   
 ### 1. 表膨胀分析
   
 snap_ts | db | schemaname | tablename | tups | pages | otta | tbloat | wastedpages | wastedbytes | wastedsize | iname | itups | ipages | iotta | ibloat | wastedipages | wastedibytes | wastedisize | totalwastedbytes
 ---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_description``` | 3838 | 34 | 32 | 1.1 | 2 | 16384 | 16384 bytes | pg_description_o_c_o_index | 3838 | 21 | 24 | 0.9 | 0 | 0 | 0 bytes | 16384
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_collation``` | 925 | 29 | 27 | 1.1 | 2 | 16384 | 16384 bytes | pg_collation_oid_index | 925 | 5 | 25 | 0.2 | 0 | 0 | 0 bytes | 16384
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_collation``` | 925 | 29 | 27 | 1.1 | 2 | 16384 | 16384 bytes | pg_collation_name_enc_nsp_index | 925 | 7 | 25 | 0.3 | 0 | 0 | 0 bytes | 16384
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_conversion``` | 132 | 3 | 2 | 1.5 | 1 | 8192 | 8192 bytes | pg_conversion_oid_index | 132 | 2 | 2 | 0.0 | 0 | 0 | 0 bytes | 8192
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_rewrite``` | 117 | 12 | 11 | 1.1 | 1 | 8192 | 8192 bytes | pg_rewrite_oid_index | 117 | 2 | 11 | 0.2 | 0 | 0 | 0 bytes | 8192
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_amproc``` | 546 | 5 | 4 | 1.3 | 1 | 8192 | 8192 bytes | pg_amproc_fam_proc_index | 546 | 5 | 3 | 1.7 | 2 | 16384 | 16384 bytes | 24576
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_operator``` | 775 | 15 | 14 | 1.1 | 1 | 8192 | 8192 bytes | pg_operator_oid_index | 775 | 5 | 12 | 0.4 | 0 | 0 | 0 bytes | 8192
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_operator``` | 775 | 15 | 14 | 1.1 | 1 | 8192 | 8192 bytes | pg_operator_oprname_l_r_n_index | 775 | 5 | 12 | 0.4 | 0 | 0 | 0 bytes | 8192
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_amproc``` | 546 | 5 | 4 | 1.3 | 1 | 8192 | 8192 bytes | pg_amproc_oid_index | 546 | 4 | 3 | 1.3 | 1 | 8192 | 8192 bytes | 16384
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_rewrite``` | 117 | 12 | 11 | 1.1 | 1 | 8192 | 8192 bytes | pg_rewrite_rel_rulename_index | 117 | 2 | 11 | 0.2 | 0 | 0 | 0 bytes | 8192
   
 #### 建议
   
 根据浪费的字节数, 设置合适的autovacuum_vacuum_scale_factor, 大表如果频繁的有更新或删除和插入操作, 建议设置较小的autovacuum_vacuum_scale_factor来降低浪费空间.  
   
 同时还需要打开autovacuum, 根据服务器的内存大小, CPU核数, 设置足够大的autovacuum_work_mem 或 autovacuum_max_workers 或 maintenance_work_mem, 以及足够小的 autovacuum_naptime.  
   
 同时还需要分析是否对大数据库使用了逻辑备份pg_dump, 系统中是否经常有长SQL, 长事务. 这些都有可能导致膨胀.  
   
 使用pg_reorg或者vacuum full可以回收膨胀的空间.  
   
 参考: https://github.com/digoal/blog/blob/master/201504/20150429_02.md.  
   
 otta评估出的表实际需要页数, iotta评估出的索引实际需要页数.  
   
 bs数据库的块大小.  
   
 tbloat表膨胀倍数, ibloat索引膨胀倍数, wastedpages表浪费了多少个数据块, wastedipages索引浪费了多少个数据块.  
   
 wastedbytes表浪费了多少字节, wastedibytes索引浪费了多少字节.  
   
 ### 2. 索引膨胀分析
   
 snap_ts | db | schemaname | tablename | tups | pages | otta | tbloat | wastedpages | wastedbytes | wastedsize | iname | itups | ipages | iotta | ibloat | wastedipages | wastedibytes | wastedisize | totalwastedbytes
 ---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_depend``` | 7329 | 55 | 54 | 1.0 | 1 | 8192 | 8192 bytes | pg_depend_reference_index | 7329 | 42 | 34 | 1.2 | 8 | 65536 | 65536 bytes | 73728
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_depend``` | 7329 | 55 | 54 | 1.0 | 1 | 8192 | 8192 bytes | pg_depend_depender_index | 7329 | 41 | 34 | 1.2 | 7 | 57344 | 57344 bytes | 65536
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_amproc``` | 546 | 5 | 4 | 1.3 | 1 | 8192 | 8192 bytes | pg_amproc_fam_proc_index | 546 | 5 | 3 | 1.7 | 2 | 16384 | 16384 bytes | 24576
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_ts_config_map``` | 304 | 2 | 2 | 0.0 | 0 | 0 | 0 bytes | pg_ts_config_map_index | 304 | 4 | 2 | 2.0 | 2 | 16384 | 16384 bytes | 16384
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_attrdef``` | 0 | 0 | 0 | 0.0 | 0 | 0 | 0 bytes | pg_attrdef_oid_index | 0 | 1 | 0 | 0.0 | 1 | 8192 | 8192 bytes | 8192
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_attrdef``` | 0 | 0 | 0 | 0.0 | 0 | 0 | 0 bytes | pg_attrdef_adrelid_adnum_index | 0 | 1 | 0 | 0.0 | 1 | 8192 | 8192 bytes | 8192
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_constraint``` | 2 | 1 | 1 | 0.0 | 0 | 0 | 0 bytes | pg_constraint_oid_index | 2 | 2 | 1 | 2.0 | 1 | 8192 | 8192 bytes | 8192
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_user_mapping``` | 0 | 0 | 0 | 0.0 | 0 | 0 | 0 bytes | pg_user_mapping_user_server_index | 0 | 1 | 0 | 0.0 | 1 | 8192 | 8192 bytes | 8192
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```pg_catalog``` | ```pg_authid``` | 2 | 1 | 1 | 0.0 | 0 | 0 | 0 bytes | pg_authid_oid_index | 2 | 2 | 1 | 2.0 | 1 | 8192 | 8192 bytes | 8192
 ```2021-03-31 11:17:59.148349+08``` | ```test``` | ```__rds_pg_stats__``` | ```snap_list``` | 0 | 0 | 0 | 0.0 | 0 | 0 | 0 bytes | snap_list_pkey | 0 | 1 | 0 | 0.0 | 1 | 8192 | 8192 bytes | 8192
   
 #### 建议
   
 如果索引膨胀太大, 会影响性能, 建议重建索引, create index CONCURRENTLY ... .  
   
 ### 3. 垃圾记录 TOP 10 表分析
   
 snap_ts | database | schemaname | tablename | n_dead_tup
 ---|---|---|---|---
   
 #### 建议
   
 通常垃圾过多, 可能是因为无法回收垃圾, 或者回收垃圾的进程繁忙或没有及时唤醒, 或者没有开启autovacuum, 或在短时间内产生了大量的垃圾.  
   
 可以等待autovacuum进行处理, 或者手工执行vacuum table.  
   
 ### 4. 未引用的大对象分析
   
 snap_ts | database | pg_size_pretty
 ---|---|---|---|---
   
 #### 建议
   
 如果大对象没有被引用时, 建议删除, 否则就类似于内存泄露, 使用vacuumlo可以删除未被引用的大对象, 例如: vacuumlo -l 1000 $db -w或者我写的调用vacuumlo()函数.  
   
 应用开发时, 注意及时删除不需要使用的大对象, 使用lo_unlink 或 驱动对应的API.  
   
 参考 http://www.postgresql.org/docs/9.4/static/largeobjects.html  
   
 ## 四、数据库安全或潜在风险分析
   
 ### 1. 表年龄前100
   
 snap_ts | database | rolname | nspname | relkind | relname | age | age_remain
 ---|---|---|---|---|---|---|---
   
 #### 建议
   
 表的年龄正常情况下应该小于vacuum_freeze_table_age, 如果剩余年龄小于2亿, 建议人为干预, 将LONG SQL或事务杀掉后, 执行vacuum freeze.  
   
 ### 2. unlogged table和hash index
   
 snap_ts | database | rolname | nspname | relname
 ---|---|---|---|---
   
 snap_ts | database | idx
 ---|---|---
   
 #### 建议
   
 unlogged table和hash index不记录XLOG, 无法使用流复制或者log shipping的方式复制到standby节点, 如果在standby节点执行某些SQL, 可能导致报错或查不到数据.  
   
 在数据库CRASH后无法修复unlogged table和hash index, 不建议使用.  
   
 PITR对unlogged table和hash index也不起作用.  
   
 ### 3. 剩余可使用次数不足1000万次的序列检查
   
 snap_ts | database | rolname | nspname | relname | times_remain
 ---|---|---|---|---|---
   
 #### 建议
   
 序列剩余使用次数到了之后, 将无法使用, 报错, 请开发人员关注.  
   
 ### 4. 锁等待分析
   
 snap_ts | locktype | r_mode | r_user | r_db | relation | r_pid | r_page | r_tuple | r_xact_start | r_query_start | r_locktime | r_query | w_mode | w_pid | w_page | w_tuple | w_xact_start | w_query_start | w_locktime | w_query
 ---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
   
 #### 建议
   
 锁等待状态, 反映业务逻辑的问题或者SQL性能有问题, 建议深入排查持锁的SQL.  
   
(342 rows)

