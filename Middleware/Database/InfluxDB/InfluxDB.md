# InfluxDB

时序数据库

## 三大特性

- 基于时间序列
- 可度量性
- 基于事件

## 数据模型

### Measurement（SQL Table）

相当于关系型数据库中表的概念。

### Tags（indexed）

标签，相当于主键。

### Field（not indexed）

字段。

### Point（SQL Record）

表中的一条数据。

## 时间线（Series）

一个数据源采集的指标，随着时间的流逝而源源不断地产生数据。这样形成的数据线称为时间线。

Series = Measurement + Tags
