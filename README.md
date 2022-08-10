# influxDB查询学习

```python
from(bucket: "ics")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "REALTIME_01040104_120001")
  |> window(every: 5m)
  |> mean()
  |> duplicate(column: "_stop", as: "_time")
  |> window(every: inf)
```

`from(bucket: "ics")`：设置查询桶

`|>`管道，将上一次的结果输入到当前

`range(start: v.timeRangeStart, stop: v.timeRangeStop)` 时间范围

`|> filter(fn: (r) => r["_measurement"] == "REALTIME_01040104_120001")`将时间范围内的数据输入进来并进行条件筛选

`|> window(every: 5m)`将筛选数据按照时间间隔分割成窗口，如我查询20分钟，按照5分钟一个窗口会将数据切分成4个窗口

`|> mean()`将单个窗口内的数据聚合，本处是5分钟内数据

当每个窗口中的行被聚合时，其输出表仅包含具有聚合值的单个行。所有窗口表仍然是独立的，并且在可视化时，将显示为单个未连接的点。

上述步骤执行完毕后，4个窗口数据将展示为4个点，这时`_time`字段会消失（<font color='red'>在聚合值时，生成的表没有列，因为用于聚合的记录都具有不同的时间戳。聚合函数不推断应为聚合值使用的时间。因此，该列将被删除。</font>）

`  |> duplicate(column: "_stop", as: "_time")` 将每个窗口的`_stop`列复制为`_time`列

`|> window(every: inf)`将上述所有的点（本次是4个点）收集到一个无限的窗口中，相当于取消了4个窗口的隔离，现在4个点都在一个大窗口中。



上述查询等效于下面查询

```python
from(bucket: "ics")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "REALTIME_01040104_120001")
  |> aggregateWindow(every: 5m, fn: mean)
```

`  |> aggregateWindow(every: 5m, fn: mean)`官方描述为:通过将数据分组到固定的时间窗口并对其每个窗口应用聚合或选择器函数来缩减采样数据。翻译：将数据按照5分钟一个窗口切分并计算切分窗口内的平均值，并将切分聚合后的点放入无界窗口中。



获取某段时间内的平均值

```python
from(bucket: "ics")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "REALTIME_01040104_120001")
  |> aggregateWindow(every: 5m, fn: mean)
  |> mean()
```

`|> mean()`将无界窗口中的点位聚合起来求平均值，如果需要某段时间内的单一值，可以使用这种方法

