# 模式设计技巧

## Druid 的数据模型

本文主要讨论对来自其他类型数据库系统的用户的提示，以及常规提示和通用做法。

- Druid 数据存储在 datasources，datasource 类似于传统 RDBMS 中的 table。

- Druid 在向数据源摄取数据时，可以选择 rollup，也可以不 rollup。启用 rollup 功能后，Druid 会在摄取期间部分聚合数据，从而有可能减少数据行数，减少存储空间并提高查询性能。禁用 rollup 功能后，Druid 将为输入数据中为每一行存储一行，而不进行任何预聚合。

- 德鲁伊中的每一行都必须有一个时间戳。数据总是按时间划分，每个查询都有一个时间过滤器。查询结果还可以按时间段（例如分钟，小时，天等）细分。

- 除时间戳列外，Druid 数据源中的所有列均为`维度列`或`指标列`。这遵循 OLAP 数据的[标准命名约定](https://en.wikipedia.org/wiki/Online_analytical_processing#Overview_of_OLAP_systems)。

- 通常，生产数据源具有数十到数百列。

- `维度列`按原样存储，因此可以在查询时对其进行过滤，分组或聚合。它们可以是单个字符串，字符串数组，单个 Long，单个 Doubles 或单个 Float。

- `指标列`是`预先聚合`存储的，因此它们只能在查询时聚合（不能过滤或分组）。它们通常存储为数字（整数或浮点数），但也可以存储为复杂对象，例如[HyperLogLog sketches 或近似分位数]。即使禁用 rollup，也可以在摄取时配置指标，但启用 rollup 时最有用。

## 如果你来自...

### 关系模型

（如 Hive 或 PostgreSQL。）

Druid 数据源通常等效于关系数据库中的表。Druid 的`lookups`行为与`数仓型数据库`的维表相似，但是正如您将在下面看到的那样，如果可以避免，通常建议使用非规范化。

关系数据建模的常见实践规范：将数据分为多个表，这样可以减少或消除数据冗余。例如，在"sales”表中，关系建模的最佳实践需要一个"product id”列，该列是单独的"products”表中的外键，该表又具有"product id”，"product name"，和"product category”列。这样可以避免在"sales”表中引用相同产品的不同行上重复产品名称和类别。

而在 Druid 中，通常使用完全展平的数据源，这些数据源在查询时不需要 join。在" sales”表的示例中，通常在 Druid 中将" product_id”，" product_name”和" product_category”作为维度直接存储在 Druid" sales”数据源中，而无需使用单独的" products”表。完全平面的架构大大提高了性能，因为在查询时消除了 join 的需求。作为额外的速度提升，这还允许 Druid 的查询层直接对压缩的字典编码数据进行操作。也许违反直觉，相对于规范化的架构，这并没有实质性增加存储空间，

在 Druid 中建模关系数据的技巧：

- Druid 数据源没有主键或唯一键。

- 如果需要将两个大型分布式表相互 join，则必须在将数据加载到 Druid 中之前执行此操作。Druid 不支持两个数据源的查询时 join。

- 考虑是否要启用 rollup 以进行预聚合，还是要禁用 rollup 并按原样加载现有数据。Druid 中的 rollup 类似于在关系模型中创建汇总表。

## 时间序列模型

（如 OpenTSDB 或 InfluxDB。）

与时间序列数据库类似，Druid 的数据模型需要时间戳。Druid 不是时间序列数据库，但是它是存储时间序列数据的优秀选择。其灵活的数据模型使它既可以存储时间序列数据，也可以存储非时间序列数据，即使在同一数据源中也是如此。

要在 Druid 中获得最佳的时间序列数据压缩和查询性能，像时间序列数据库通常那样，按 dimension 标准名称进行分区和排序非常重要。

在 Druid 中建模时间序列数据的提示：

- Druid 并不认为数据点是"时间序列”的一部分。取而代之的是，Druid 将每条数据作为摄入的点和聚合的点。

- 创建一个维，以指示数据点所属的 series 的名称。此维度通常称为"metric”或"name”。不要将名为" metric”的维度与 Druid metric 的概念混淆。为了获得最佳性能，请将其首先放在" dimensionsSpec”中的 dimension 列表中。

- 创建其他维度来表示数据的其他属性。在时间序列数据库系统中，这些通常称为"tag”。

- 创建与要查询的聚合类型相对应的`指标`。通常，这包括"sum”，"max”和"min”（long, float, double 类型）。

- 考虑启用 rollup，这将使 Druid 可能将多个点合并到 Druid 数据源中的一行中。

- 如果你预先不知道要有哪些列，可以使用一个空白的维度列表，然后自动检测维度列。

### 日志聚合模型

（例如 Elasticsearch 或 Splunk。）

与日志聚合系统类似，Druid 提供了反向索引以进行快速搜索和过滤。与这些系统相比，Druid 的搜索能力通常较不发达，而其分析能力通常也较发达。Druid 与这些系统之间的主要数据建模差异在于，将数据提取到 Druid 中时，您必须更加明确。Druid 列具有预先特定的类型，而 Druid 暂时不支持嵌套数据。

在 Druid 中建模日志数据的提示：

- 如果你预先不知道要有哪些列，可以使用一个空白的维度列表，然后自动检测维度列。

- 如果你嵌套了数据，请使用`flattenSpec`展平数据。

- 如果您的日志数据主要具有分析用例，请考虑启用 rollup。这将意味着你将失去从 Druid 检索单个事件的能力，但可能会获得更高的压缩并提高查询性能。

> 本文翻译自 Druid [官方文档](https://druid.apache.org/docs/latest/ingestion/schema-design.html)

> 欢迎关注公众号，一起学习 Druid 及更多数据存储相关知识。

![码哥字节](https://magebyte.oss-cn-shenzhen.aliyuncs.com/wechat/%E7%A0%81%E5%93%A5%E5%AD%97%E8%8A%82%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)