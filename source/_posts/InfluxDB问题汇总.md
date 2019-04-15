---
title: InfluxDB问题汇总
date: 2019-04-12 15:44:30
tags: 问题记录
---

最近在试用InfluxDB，遇到一些问题，这里汇总一下：

### Continuous Query 时间偏移问题

##### 背景

公司业务系统需要使用东八区的时间进行业务查询，所以最初保存`influx point`的时候，做了时间处理，后来因为需要对大量的数据进行重采样，自然而然想到`CQ`这个功能，遇到了这个`time offset`不生效的问题。

##### 重现

假设已经生成性能日志表`perflog`，`temp`为所在数据库，构建如下的`CQ`:

```sql
CREATE CONTINUOUS QUERY cq_2m ON temp 
BEGIN 
        SELECT mean(rate) AS rate INTO temp.autogen.cq_2m 
        FROM temp.autogen.perflog 
        GROUP BY time(2m,8h) fill(0) 
END
```

其中，`GROUP BY time（2m,8h）`表示，每两分钟记录一次`rate`字段的平均值，存储到表`cq_2m`中，执行时间偏移八小时，这个语法在[influx官方文档](<https://docs.influxdata.com/influxdb/v1.7/query_language/continuous_queries/#basic-syntax>)的`Example 4`中有提到：

> Use an [offset interval](https://docs.influxdata.com/influxdb/v1.7/query_language/data_exploration/#advanced-group-by-time-syntax) in the `GROUP BY time()` clause to alter both the CQ’s default execution time and preset time boundaries.

然而，这个`8h`的参数只偏移了执行时间，并没有作用到`where`条件的时间区间，测试日志如下：

```
2019-03-27T07:04:00.029446Z info Executing continuous query {"log_id": "0ERSB7o0000", "service": "continuous_querier", "trace_id": "0ERVFR7G000", "op_name": "continuous_querier_execute", "name": "cq_2m", "db_instance": "temp", "start": "2019-03-27T07:02:00.000000Z", "end": "2019-03-27T07:04:00.000000Z"}
2019-03-27T07:04:00.030061Z info Executing query {"log_id": "0ERSB7o0000", "service": "query", "query": "SELECT mean(rate) AS rate INTO temp.autogen.cq_2m FROM temp.autogen.perflog WHERE time >= '2019-03-27T07:02:00Z' AND time < '2019-03-27T07:04:00Z' GROUP BY time(2m,8h) fill(0)"}
```

当时是北京时间`15:04:00`，但是`where time`的区间还是提前了八个小时，就是取自`influx`自带的`now()`函数，这个函数使用的是`UTC`时间。

##### 解决

提了[issue](<https://github.com/influxdata/influxdb/issues/12926>)，并没有满意的答复，只能曲线救国。

思路有两种：

1. 统一坐标系
2. 用`quartz`代替`CQ`进行重采样

因为2比较简单，工作量比1小，就选了2。:sweat_smile:















