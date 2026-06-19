# 编写 Druid Spec 及其注意事项

> 本文参考 Druid 官方文档。

Apache Druid 是一个集时间序列数据库、数据仓库和全文检索系统特点于一体的分析性数据平台（OLAP）。Druid 作为一个高可用、高性能和多特性的 OLAP 平台，使用场景丰富。

![Druid 使用场景](
https://magebyte.oss-cn-shenzhen.aliyuncs.com/druid/usage.png)

许多互联网公司基于 Druid 搭建 OLAP 数据分析和 BI 平台。如：

- [快手万亿级实时 OLAP 平台的建设与实践](https://www.infoq.cn/article/IWfHmTig_KNAeEJKF8eS)
- [Druid 在有赞的实践](https://zhuanlan.zhihu.com/p/25593670)
- [Druid 在小米公司的技术实践](https://zhuanlan.zhihu.com/p/25593670)

此前 Druid 系列文章已经详解过 Druid 的特性、使用场景、架构和实现原理。可以参考:

- [时间序列数据库(TSDB)初识与选择](https://mp.weixin.qq.com/s/9ckUy3Lz9GHTNPauNlpV0w)
- [十分钟了解 Apache Druid](https://mp.weixin.qq.com/s/dGBfQdmD7niW32BXWb2B0g)
- [Apache Druid 的集群设计与工作流程](https://mp.weixin.qq.com/s/wPDdXU3dIvt-yZ-u5AmC9g)
- [Apache Druid 底层存储设计(列存储与全文检索)](https://mp.weixin.qq.com/s/5mb0efBkwfyre6FKrzLuhg)

本文将指导读者完整定义一个完整 `Spec`，并指出关键注意事项。`Spec` 是 Druid 数据摄入的配置信息，使用 `json` 格式，使用 Druid 时可以通过界面配置，最后生成 `Spec` 文件，也可以直接编写 `Spec` 文件，然后上传配置。无论使用哪种方式，深入了解 `Spec` 的编写既是开始使用 Druid 的第一步，也是深入了解 Druid 各种概念，继而深入了解 Druid 原理的必经之路。

## 示例数据

假设我们有以下网络流量数据：

- `srcIP`: 发送端 IP 地址
- `srcPort`: 发送端端口号
- `dstIP`: 接收端 IP 地址
- `dstPort`: 接收端端口号
- `protocol`: IP 协议号
- `packets`: 传输的包的数量
- `bytes`: 传输字节数
- `cost`: 传输耗费的时间

```json
{"ts":"2018-01-01T01:01:35Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":2000, "dstPort":3000, "protocol": 6, "packets":10, "bytes":1000, "cost": 1.4}
{"ts":"2018-01-01T01:01:51Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":2000, "dstPort":3000, "protocol": 6, "packets":20, "bytes":2000, "cost": 3.1}
{"ts":"2018-01-01T01:01:59Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":2000, "dstPort":3000, "protocol": 6, "packets":30, "bytes":3000, "cost": 0.4}
{"ts":"2018-01-01T01:02:14Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":40, "bytes":4000, "cost": 7.9}
{"ts":"2018-01-01T01:02:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":50, "bytes":5000, "cost": 10.2}
{"ts":"2018-01-01T01:03:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":60, "bytes":6000, "cost": 4.3}
{"ts":"2018-01-01T02:33:14Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8", "srcPort":4000, "dstPort":5000, "protocol": 17, "packets":100, "bytes":10000, "cost": 22.4}
{"ts":"2018-01-01T02:33:45Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8", "srcPort":4000, "dstPort":5000, "protocol": 17, "packets":200, "bytes":20000, "cost": 34.5}
{"ts":"2018-01-01T02:35:45Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8", "srcPort":4000, "dstPort":5000, "protocol": 17, "packets":300, "bytes":30000, "cost": 46.3}
```

将上面 JSON 内容保存到 `$druid_root\quickstart\`目录下的 `ingestion-tutorial-data.json` 文件中。

下面我们开始编写一个 `Spect` 将上面的数据写入 Druid。

在本教程中，我们将使用本地批处理`indexing`任务。如果使用其他任务类型，摄入规范的某些地方将会不一样，我们将在教程中指出。

下面将详细讲解 `Spec` 配置，你将了解以下内容：

![内容](
https://magebyte.oss-cn-shenzhen.aliyuncs.com/druid/menu.png)

## 定义 schema

Druid 摄入 spec 的核心元素是 `dataSchema` 。`dataSchema` 定义如何将输入的数据解析成 Druid 能够存储的列集合。

我们从一个空的`dataSchema` 开始，并按教程一步步添加字段。

在`quickstart/` 目录下创建 `ingestion-tutorial-index.json` 文件，将以下内容写入文件：

```json
"dataSchema" : {}
```

随着教程的进行，我们将不断的修改此 spec 文件。

### 数据源名称

数据源名称通过`dataSchema` 下的 `dataSource` 参数指定。`dataSource` 类似于 RDBMS 的 Table Name，写入的数据通过此名称查询，如：`select * from $dataSource`。

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
}
```

让我们将教程中的数据源命名为 `ingestion-tutorial`。

### 时间列

`dataSchema` 需要知道如何从输入的数据中提取主时间字段。Druid 的数据必须有时间字段，Druid 底层按时间分 segment 来存储数据，详情可以参考[《Apache Druid 的集群设计与工作流程》](https://mp.weixin.qq.com/s/wPDdXU3dIvt-yZ-u5AmC9g)。

我们数据中的时间戳列是"ts"，它是一个 `ISO 8601` 规范的时间戳，我们将配置此字段的 `timestampSpec`信息加到 `dataSchema` 下：

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  }
}
```

### 列类型

现在，我们已经定义了时间列，让我们看一下其他列的定义。

Druid 支持以下列类型：String，Long，Float，Double。下面章节中我们将看到这些类型如何被使用。

在我们讲如何定义其他非时间列之前，先讨论一下 `rollup`。

### Rollup

在摄入数据时，我们需要考虑是否需要 rollup。

- 如果开启 rollup，需要将输入数据列分成两种类型，维度(dimension)和指标(metric)。维度是 rollup 的 grouping 列（用于 group by，filtering），指标是被聚合计算的列。
- 如果不开启 rollup，所有列都被视为维度，将不会进行预聚合。

在此教程中，我们开启 rollup。在 `dataSchema` 的 `granularitySpec`中指定：

```json
"dataSchema" : {
 "dataSource" : "ingestion-tutorial",
 "timestampSpec" : {
   "format" : "iso",
   "column" : "ts"
 },
 "granularitySpec" : {
   "rollup" : true
 }
}
```

#### 选择维度和指标

在此教程中，我们按以下方式划分维度列和指标列：

- Dimensions: srcIP, srcPort, dstIP, dstPort, protocol
- Metrics: packets, bytes, cost

这些维度是一组属性，用以标识一组网络流量数据，而指标代表按此维度组合的网络流量的实际情况。

让我们看看如何在 spec 中定义维度和指标吧。

#### 维度

维度由 `dataSchema` 中的 `dimensionsSpec` 参数指定。

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "dimensionsSpec" : {
    "dimensions": [
      "srcIP",
      { "name" : "srcPort", "type" : "long" },
      { "name" : "dstIP", "type" : "string" },
      { "name" : "dstPort", "type" : "long" },
      { "name" : "protocol", "type" : "string" }
    ]
  },
  "granularitySpec" : {
    "rollup" : true
  }
}
```

每个维度都有一个 `name` 和 一个 `type` ，其中 `type` 可以是"long", "float", "double", or "string"。

注意，`srcIP` 是一个 "string" 维度。对于字符串维度，只需要指定维度的名称就可以了，因为它的类型默认为"string"。

也请注意， `protocol` 在输入数据中是数字类型，但我们以 "string" 列类型提取它，所以 Druid 在摄入数据时会将其强制由 long 类型转换成 string 类型。

##### Strings vs Numbers

数字类型的数据应该作为数字维度还是字符串维度？

数字维度相对于字符串维度有以下优势和劣势：

- 优势：数字需要更小的存储空间，并且在读取该列时需要更小的开销。
- 劣势：数字维度没有索引，所以按此列 filter 的操作会比字符串类型的维度(这种维度有索引)更慢。

#### 指标

指标通过 `dataSchema` 中的 `metricsSpec` 参数指定：

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "dimensionsSpec" : {
    "dimensions": [
      "srcIP",
      { "name" : "srcPort", "type" : "long" },
      { "name" : "dstIP", "type" : "string" },
      { "name" : "dstPort", "type" : "long" },
      { "name" : "protocol", "type" : "string" }
    ]
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "rollup" : true
  }
}
```

在定义指标时，必须指定当前列在 rollup 时应该执行的聚合类型。

这里，我们在 `packets` 和 `bytes` 两个指标列上定义了 long 类型的 sum 聚合，在 `cost` 列上定义了一个 double 的 sum 聚合。

注意 `metricsSpec` 与 `dimensionSpec` 和 `parseSpec` 的嵌套层级不一样。它和 `dataSchema` 中的 `parser`在同一嵌套层级。

注意，我们也定义了一个 `count` 聚合器。这个计数聚合器将统计原始数据摄入的行数。

### No rollup

如果我们不使用 rollup，将在 `dimensionsSpec` 中指定所有列，如：

```json
      "dimensionsSpec" : {
        "dimensions": [
          "srcIP",
          { "name" : "srcPort", "type" : "long" },
          { "name" : "dstIP", "type" : "string" },
          { "name" : "dstPort", "type" : "long" },
          { "name" : "protocol", "type" : "string" },
          { "name" : "packets", "type" : "long" },
          { "name" : "bytes", "type" : "long" },
          { "name" : "srcPort", "type" : "double" }
        ]
      },
```

### 定义粒度

此时，我们已经完成了 `parser` 和 `metricSpec` 的定义，并几乎要完成 Spec 的 `dataSchema` 。

我们还需要在 `granularitySpec` 中设置一些额外的参数：

- granularitySpec 类型：支持两种类型——`uniform` 和 `arbitrary` 。在本教程中，我们将使用 `uniform`，这样所有的 segment 将有统一的时间范围大小(本示例中，所有 segment 覆盖一个小时的数据量)。
- segment 粒度：设置单个 segment 应该包含多大时间范围的数据，如：`DAY`，`WEEK` 。
- 时间列中时间戳的 buckting 粒度(称为查询粒度 `queryGranularity` )。

#### Segment 粒度

segment 粒度通过 `granularitySpec` 中的 `segmentGranularity` 属性配置。此文档中，我们将创建 hourly 粒度的 segment：

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "dimensionsSpec" : {
    "dimensions": [
      "srcIP",
      { "name" : "srcPort", "type" : "long" },
      { "name" : "dstIP", "type" : "string" },
      { "name" : "dstPort", "type" : "long" },
      { "name" : "protocol", "type" : "string" }
    ]
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "type" : "uniform",
    "segmentGranularity" : "HOUR",
    "rollup" : true
  }
}
```

我们的输入数据包含两个小时的事件，所以此任务将生成两个 segment。

#### 查询粒度

查询粒度通过 `granularitySpec` 中的 `queryGranularity` 属性配置。此教程中，我们使用 minute 级粒度：

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "dimensionsSpec" : {
    "dimensions": [
      "srcIP",
      { "name" : "srcPort", "type" : "long" },
      { "name" : "dstIP", "type" : "string" },
      { "name" : "dstPort", "type" : "long" },
      { "name" : "protocol", "type" : "string" }
    ]
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "type" : "uniform",
    "segmentGranularity" : "HOUR",
    "queryGranularity" : "MINUTE",
    "rollup" : true
  }
}
```

为了查看查询粒度配置的效果，让我们从原始输入数据中查看这一行：

```json
{"ts":"2018-01-01T01:03:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":60, "bytes":6000, "cost": 4.3}
```

当使用 minute 粒度摄入这行数据时，Druid 将把这行数据的时间戳 floor 成 minute 桶的时间：

```json
{"ts":"2018-01-01T01:03:00Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":60, "bytes":6000, "cost": 4.3}
```

#### 定义时间范围

对于批量任务，必须定义时间范围。在时间范围之外的输入数据将不被摄入。

这个时间范围也在 `granularitySpec` 中指定：

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "dimensionsSpec" : {
    "dimensions": [
      "srcIP",
      { "name" : "srcPort", "type" : "long" },
      { "name" : "dstIP", "type" : "string" },
      { "name" : "dstPort", "type" : "long" },
      { "name" : "protocol", "type" : "string" }
    ]
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "type" : "uniform",
    "segmentGranularity" : "HOUR",
    "queryGranularity" : "MINUTE",
    "intervals" : ["2018-01-01/2018-01-02"],
    "rollup" : true
  }
}
```

## 定义任务类型

现在我们已经完成了 `dataSchema` 的定义。接下来要做的就是将我们创建的 `dataSchema` 放入一个数据摄入任务中，并指定输入的源。

`dataSchema` 适用于所有类型的任务，但每种任务类型都有自生的规范格式。在本教程中，我们使用本地批量数据摄入任务类型(the native ingestion task)：

```json
{
  "type" : "index_parallel",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "ingestion-tutorial",
      "timestampSpec" : {
        "format" : "iso",
        "column" : "ts"
      },
      "dimensionsSpec" : {
        "dimensions": [
          "srcIP",
          { "name" : "srcPort", "type" : "long" },
          { "name" : "dstIP", "type" : "string" },
          { "name" : "dstPort", "type" : "long" },
          { "name" : "protocol", "type" : "string" }
        ]
      },
      "metricsSpec" : [
        { "type" : "count", "name" : "count" },
        { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
        { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
        { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "HOUR",
        "queryGranularity" : "MINUTE",
        "intervals" : ["2018-01-01/2018-01-02"],
        "rollup" : true
      }
    }
  }
}
```

## 定义输入源

现在，让我们来定义我们自己的输入源，它在 `ioConfig` 对象中指定。每种任务类型都有自己的 `ioConfig` 类型。为了读取输入数据，我们需要指定一个 `inputSource`。我们之前保存的网络流量数据需要从一个本地文件读取，其配置如下：

```json
    "ioConfig" : {
      "type" : "index_parallel",
      "inputSource" : {
        "type" : "local",
        "baseDir" : "quickstart/",
        "filter" : "ingestion-tutorial-data.json"
      }
    }
```

### 定义数据格式

因为我们的数据是 JSON 字符串形式的，我们使用 `inputFormat` `json` 格式化数据（还支持 csv、protobuf 等数据类型）：

```json
    "ioConfig" : {
      "type" : "index_parallel",
      "inputSource" : {
        "type" : "local",
        "baseDir" : "quickstart/",
        "filter" : "ingestion-tutorial-data.json"
      },
      "inputFormat" : {
        "type" : "json"
      }
    }
```

```json
{
  "type" : "index_parallel",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "ingestion-tutorial",
      "timestampSpec" : {
        "format" : "iso",
        "column" : "ts"
      },
      "dimensionsSpec" : {
        "dimensions": [
          "srcIP",
          { "name" : "srcPort", "type" : "long" },
          { "name" : "dstIP", "type" : "string" },
          { "name" : "dstPort", "type" : "long" },
          { "name" : "protocol", "type" : "string" }
        ]
      },
      "metricsSpec" : [
        { "type" : "count", "name" : "count" },
        { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
        { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
        { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "HOUR",
        "queryGranularity" : "MINUTE",
        "intervals" : ["2018-01-01/2018-01-02"],
        "rollup" : true
      }
    },
    "ioConfig" : {
      "type" : "index_parallel",
      "inputSource" : {
        "type" : "local",
        "baseDir" : "quickstart/",
        "filter" : "ingestion-tutorial-data.json"
      },
      "inputFormat" : {
        "type" : "json"
      }
    }
  }
}
```

## 其他调整

每一个摄取任务都有 `tuningConfig` 配置项，它允许用户调整各种摄取参数。

举例来说，让我们添加一个`tuningConfig`，以设置本次批量摄取任务的目标 segment 大小：

```json
    "tuningConfig" : {
      "type" : "index_parallel",
      "maxRowsPerSegment" : 5000000
    }
```

## 最终 spec

现在我们已经定义完成了一个摄取规范，它现在看起来如下所示：

```json
{
  "type" : "index_parallel",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "ingestion-tutorial",
      "timestampSpec" : {
        "format" : "iso",
        "column" : "ts"
      },
      "dimensionsSpec" : {
        "dimensions": [
          "srcIP",
          { "name" : "srcPort", "type" : "long" },
          { "name" : "dstIP", "type" : "string" },
          { "name" : "dstPort", "type" : "long" },
          { "name" : "protocol", "type" : "string" }
        ]
      },
      "metricsSpec" : [
        { "type" : "count", "name" : "count" },
        { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
        { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
        { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "HOUR",
        "queryGranularity" : "MINUTE",
        "intervals" : ["2018-01-01/2018-01-02"],
        "rollup" : true
      }
    },
    "ioConfig" : {
      "type" : "index_parallel",
      "inputSource" : {
        "type" : "local",
        "baseDir" : "quickstart/",
        "filter" : "ingestion-tutorial-data.json"
      },
      "inputFormat" : {
        "type" : "json"
      }
    },
    "tuningConfig" : {
      "type" : "index_parallel",
      "maxRowsPerSegment" : 5000000
    }
  }
}
```

## 提交任务并查询数据

在包根目录下，运行下面命令：

```shell
bin/post-index-task --file quickstart/ingestion-tutorial-index.json --url http://localhost:8081
```

脚本执行完成后，我们来查询数据。

让我们运行 `bin/dsql` 并发送一个 `select * from "ingestion-tutorial";` 语句，查询已经被写入的数据：

```mysql
$ bin/dsql
Welcome to dsql, the command-line client for Druid SQL.
Type "\h" for help.
dsql> select * from "ingestion-tutorial";

┌──────────────────────────┬───────┬──────┬───────┬─────────┬─────────┬─────────┬──────────┬─────────┬─────────┐
│ __time                   │ bytes │ cost │ count │ dstIP   │ dstPort │ packets │ protocol │ srcIP   │ srcPort │
├──────────────────────────┼───────┼──────┼───────┼─────────┼─────────┼─────────┼──────────┼─────────┼─────────┤
│ 2018-01-01T01:01:00.000Z │  6000 │  4.9 │     3 │ 2.2.2.2 │    3000 │      60 │ 6        │ 1.1.1.1 │    2000 │
│ 2018-01-01T01:02:00.000Z │  9000 │ 18.1 │     2 │ 2.2.2.2 │    7000 │      90 │ 6        │ 1.1.1.1 │    5000 │
│ 2018-01-01T01:03:00.000Z │  6000 │  4.3 │     1 │ 2.2.2.2 │    7000 │      60 │ 6        │ 1.1.1.1 │    5000 │
│ 2018-01-01T02:33:00.000Z │ 30000 │ 56.9 │     2 │ 8.8.8.8 │    5000 │     300 │ 17       │ 7.7.7.7 │    4000 │
│ 2018-01-01T02:35:00.000Z │ 30000 │ 46.3 │     1 │ 8.8.8.8 │    5000 │     300 │ 17       │ 7.7.7.7 │    4000 │
└──────────────────────────┴───────┴──────┴───────┴─────────┴─────────┴─────────┴──────────┴─────────┴─────────┘
Retrieved 5 rows in 0.12s.

dsql>
```

如果觉得阅读后对你有帮助，希望分享、点赞、在看三连哦。

关注 【码哥字节】解锁更多硬核。

**推荐阅读**

以下几篇文章阅读量与读者反馈都很好，推荐大家阅读：

- [数据库系统设计概述](https://mp.weixin.qq.com/s/HoSul-GjX6FmulY6ugdmgQ)
- [不可不知的软件架构模式](https://mp.weixin.qq.com/s/77OIESNbJtF6jNkSLq2fyg)
- [Tomcat 架构原理解析到架构设计借鉴](https://mp.weixin.qq.com/s/fU5Jj9tQvNTjRiT9grm6RA)
- [Tomcat 高并发之道原理拆解与性能调优](https://mp.weixin.qq.com/s/0jj7QfQCxEIBS2PM58H4bw)

公众号后台回复 ”加群“，加入读者技术群，里面有阿里、腾讯的小伙伴一起探讨技术。

![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/wechat/Snip20200314_5.png)
