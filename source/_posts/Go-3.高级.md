---
title:  Go-高级
tags:
  - Go
categories:
  - Go语言
date: 2021-08-25 13:05:00
---


# 36.二进制协议gob及msgpack介绍

<!--more-->

本文主要介绍二进制协议gob及msgpack的基本使用。

最近在写一个gin框架的session服务时遇到了一个问题，Go语言中的json包在序列化空接口存放的数字类型（整型、浮点型等）都序列化成float64类型。

我们构造一个结构体如下：

```go
Copytype s struct {
	data map[string]interface{}
}
```

## json序列化的问题

```go
Copyfunc jsonDemo() {
	var s1 = s{
		data: make(map[string]interface{}, 8),
	}
	s1.data[&quot;count&quot;] = 1
	ret, err := json.Marshal(s1.data)
	if err != nil {
		fmt.Println(&quot;marshal failed&quot;, err)
	}
	fmt.Printf(&quot;%#v\n&quot;, string(ret))
	var s2 = s{
		data: make(map[string]interface{}, 8),
	}
	err = json.Unmarshal(ret, &amp;s2.data)
	if err != nil {
		fmt.Println(&quot;unmarshal failed&quot;, err)
	}
	fmt.Println(s2)
	for _, v := range s2.data {
		fmt.Printf(&quot;value:%v, type:%T\n&quot;, v, v)
	}
}
```

输出结果：

```bash
Copy&quot;{\&quot;count\&quot;:1}&quot;
{map[count:1]}
value:1, type:float64
```

## gob序列化示例

标准库gob是golang提供的“私有”的编解码方式，它的效率会比json，xml等更高，特别适合在Go语言程序间传递数据。

```go
Copyfunc gobDemo() {
	var s1 = s{
		data: make(map[string]interface{}, 8),
	}
	s1.data[&quot;count&quot;] = 1
	// encode
	buf := new(bytes.Buffer)
	enc := gob.NewEncoder(buf)
	err := enc.Encode(s1.data)
	if err != nil {
		fmt.Println(&quot;gob encode failed, err:&quot;, err)
		return
	}
	b := buf.Bytes()
	fmt.Println(b)
	var s2 = s{
		data: make(map[string]interface{}, 8),
	}
	// decode
	dec := gob.NewDecoder(bytes.NewBuffer(b))
	err = dec.Decode(&amp;s2.data)
	if err != nil {
		fmt.Println(&quot;gob decode failed, err&quot;, err)
		return
	}
	fmt.Println(s2.data)
	for _, v := range s2.data {
		fmt.Printf(&quot;value:%v, type:%T\n&quot;, v, v)
	}
}
```

## msgpack

[MessagePack](https://msgpack.org/)是一种高效的二进制序列化格式。它允许你在多种语言(如JSON)之间交换数据。但它更快更小。

## 安装

```bash
Copygo get -u github.com/vmihailenco/msgpack
```

## 示例

```go
Copypackage main

import (
	&quot;fmt&quot;

	&quot;github.com/vmihailenco/msgpack&quot;
)

// msgpack demo

type Person struct {
	Name   string
	Age    int
	Gender string
}

func main() {
	p1 := Person{
		Name:   &quot;沙河娜扎&quot;,
		Age:    18,
		Gender: &quot;男&quot;,
	}
	// marshal
	b, err := msgpack.Marshal(p1)
	if err != nil {
		fmt.Printf(&quot;msgpack marshal failed,err:%v&quot;, err)
		return
	}

	// unmarshal
	var p2 Person
	err = msgpack.Unmarshal(b, &amp;p2)
	if err != nil {
		fmt.Printf(&quot;msgpack unmarshal failed,err:%v&quot;, err)
		return
	}
	fmt.Printf(&quot;p2:%#v\n&quot;, p2) // p2:main.Person{Name:&quot;沙河娜扎&quot;, Age:18, Gender:&quot;男&quot;}
}
```



本文介绍了Go语言版经典的排序算法–快速排序、归并排序和堆排序。

# 37.排序算法

## 快速排序

```go
Copyfunc quickSort(data []int) {
	if len(data) &lt;= 1 {
		return
	}
	base := data[0]
	l, r := 0, len(data)-1
	for i := 1; i &lt;= r; {
		if data[i] &gt; base {
			data[i], data[r] = data[r], data[i]
			r--
		} else {
			data[i], data[l] = data[l], data[i]
			l++
			i++
		}
	}
	quickSort(data[:l])
	quickSort(data[l+1:])
}

func main() {
	s := make([]int, 0, 16)
	for i := 0; i &lt; 16; i++ {
		s = append(s, rand.Intn(100))
	}
	fmt.Println(s)
	quickSort(s)
	fmt.Println(s)
}
```

## 归并排序

```go
Copyfunc mergeSort(data []int) []int {
	length := len(data)
	if length &lt;= 1 {
		return data
	}
	num := length / 2
	left := mergeSort(data[:num])
	right := mergeSort(data[num:])
	return merge(left, right)
}

func merge(left, right []int) (result []int) {
	l, r := 0, 0
	for l &lt; len(left) &amp;&amp; r &lt; len(right) {
		if left[l] &lt; right[r] {
			result = append(result, left[l])
			l++
		} else {
			result = append(result, right[r])
			r++
		}
	}
	result = append(result, left[l:]...)
	result = append(result, right[r:]...)
	return
}

func main() {
	s := make([]int, 0, 16)
	for i := 0; i &lt; 16; i++ {
		s = append(s, rand.Intn(100))
	}
	fmt.Println(s)
	s = mergeSort(s)
	fmt.Println(s)
}
```

## 堆排序

```go
Copyfunc heapSort(array []int) {
	m := len(array)
	s := m / 2
	for i := s; i &gt; -1; i-- {
		heap(array, i, m-1)
	}
	for i := m - 1; i &gt; 0; i-- {
		array[i], array[0] = array[0], array[i]
		heap(array, 0, i-1)
	}
}
func heap(array []int, i, end int) {
	l := 2*i + 1
	if l &gt; end {
		return
	}
	n := l
	r := 2*i + 2
	if r &lt;= end &amp;&amp; array[r] &gt; array[l] {
		n = r
	}
	if array[i] &gt; array[n] {
		return
	}
	array[n], array[i] = array[i], array[n]
	heap(array, n, end)
}
func main() {
	s := make([]int, 0, 16)
	for i := 0; i &lt; 16; i++ {
		s = append(s, rand.Intn(100))
	}
	fmt.Println(s)
	heapSort(s)
	fmt.Println(s)
}
```

protobuf是一种高效的数据格式，平台无关、语言无关、可扩展，可用于 RPC 系统和持续数据存储系统。

# 38.protobuf

## protobuf介绍

`Protobuf`是`Protocol Buffer`的简称，它是Google公司于2008年开源的一种高效的平台无关、语言无关、可扩展的数据格式，目前Protobuf作为接口规范的描述语言，可以作为Go语言RPC接口的基础工具。

## protobuf使用

`protobuf`是一个与语言无关的一个数据协议，所以我们需要先编写IDL文件然后借助专用工具生成指定语言的代码，从而实现数据的序列化与反序列化过程。

大致开发流程如下： 1. IDL编写 2. 生成指定语言的代码 3. 序列化和反序列化

## protobuf语法

[protobuf3语法指南](https://colobu.com/2017/03/16/Protobuf3-language-guide/)

## 编译器安装

## ptotoc

`protobuf`协议编译器是用c++编写的，根据自己的操作系统下载对应版本的`protoc`编译器：https://github.com/protocolbuffers/protobuf/releases，解压后拷贝到`GOPATH/bin`目录下。

## protoc-gen-go

安装生成Go语言代码的工具

```bash
Copygo get -u github.com/golang/protobuf/protoc-gen-go
```

## 编写IDL代码

在`protobuf_demo/address`目录下新建一个名为`person.proto`的文件具体内容如下：

~~~protobuf
Copy// 指定使用protobuf版本
// 此处使用v3版本
syntax = "proto3";

// 包名，通过protoc生成go文件
package address;

// 性别类型
// 枚举类型第一个字段必须为0
enum GenderType {
    SECRET = 0;
    FEMALE = 1;
    MALE = 2;
}

// 人
message Person {
    int64 id = 1;
    string name = 2;
    GenderType gender = 3;
    string number = 4;
}

// 联系簿
message ContactBook {
    repeated Person persons = 1;
}
```

# 生成go语言代码

在protobuf_demo/address目录下执行以下命令。

```bash
address $ protoc --go_out=. ./person.proto 
```

此时在当前目录下会生成一个person.pb.go文件，我们的Go语言代码里就是使用这个文件。
在protobuf_demo/main.go文件中：

```go
package main

import (
	"fmt"
	"io/ioutil"

	"github.com/golang/protobuf/proto"

	"github.com/Q1mi/studygo/code_demo/protobuf_demo/address"
)

// protobuf demo

func main() {
	var cb address.ContactBook

	p1 := address.Person{
		Name:   "小王子",
		Gender: address.GenderType_MALE,
		Number: "7878778",
	}
	fmt.Println(p1)
	cb.Persons = append(cb.Persons, &p1)
	// 序列化
	data, err := proto.Marshal(&p1)
	if err != nil {
		fmt.Printf("marshal failed,err:%v\n", err)
		return
	}
	ioutil.WriteFile("./proto.dat", data, 0644)

	data2, err := ioutil.ReadFile("./proto.dat")
	if err != nil {
		fmt.Printf("read file failed, err:%v\n", err)
		return
	}
	var p2 address.Person
	proto.Unmarshal(data2, &p2)
	fmt.Println(p2)
}
```
~~~

# 39.influxDB

本文介绍了`influxDB`时序数据库及Go语言操作`influxDB`。

[InfluxDB](https://www.influxdata.com/)是一个开源分布式时序、事件和指标数据库。使用Go语言编写，无需外部依赖。其设计目标是实现分布式和水平伸缩扩展。

## 安装

## 下载

https://portal.influxdata.com/downloads/

这里需要注意因为这个网站引用了google的api所以国内点页面的按钮是没反应的，怎么办呢？

按照下图所示，按`F12`打开浏览器的控制台，然后点击`Elements`，按下`Ctrl/Command+F`搜索`releases/influxdb`，按回车查找自己所需版本的下载地址。[![influxdb_01.png](http://www.chenyoude.com/go/influxdb_01.png)](http://www.chenyoude.com/go/influxdb_01.png)

Mac和Linux用户可以点击https://v2.docs.influxdata.com/v2.0/get-started/下载。

## 安装

将上一步的压缩包，解压到本地。

## influxDB介绍

## 名词介绍

| influxDB名词 | 传统数据库概念 |
| ------------ | -------------- |
| database     | 数据库         |
| measurement  | 数据表         |
| point        | 数据行         |

## point

influxDB中的point相当于传统数据库里的一行数据，由时间戳（time）、数据（field）、标签（tag）组成。

| Point属性 | 传统数据库概念                               |
| --------- | -------------------------------------------- |
| time      | 每个数据记录时间，是数据库中的主索引         |
| field     | 各种记录值（没有索引的属性），例如温度、湿度 |
| tags      | 各种有索引的属性，例如地区、海拔             |

## Series

`Series`相当于是 InfluxDB 中一些数据的集合，在同一个 database 中，retention policy、measurement、tag sets 完全相同的数据同属于一个 series，同一个 series 的数据在物理上会按照时间顺序排列存储在一起。

## Go操作influxDB

## 安装

## influxDB 1.x版本

```bash
Copygo get github.com/influxdata/influxdb1-client/v2
```

## influxDB 2.x版本

```bash
Copygo get github.com/influxdata/influxdb-client-go
```

## 基本使用

```go
Copypackage main

import (
	&quot;fmt&quot;
	&quot;log&quot;
	&quot;time&quot;

	client &quot;github.com/influxdata/influxdb1-client/v2&quot;
)

// influxdb demo

func connInflux() client.Client {
	cli, err := client.NewHTTPClient(client.HTTPConfig{
		Addr:     &quot;http://127.0.0.1:8086&quot;,
		Username: &quot;admin&quot;,
		Password: &quot;&quot;,
	})
	if err != nil {
		log.Fatal(err)
	}
	return cli
}

// query
func queryDB(cli client.Client, cmd string) (res []client.Result, err error) {
	q := client.Query{
		Command:  cmd,
		Database: &quot;test&quot;,
	}
	if response, err := cli.Query(q); err == nil {
		if response.Error() != nil {
			return res, response.Error()
		}
		res = response.Results
	} else {
		return res, err
	}
	return res, nil
}

// insert
func writesPoints(cli client.Client) {
	bp, err := client.NewBatchPoints(client.BatchPointsConfig{
		Database:  &quot;test&quot;,
		Precision: &quot;s&quot;, //精度，默认ns
	})
	if err != nil {
		log.Fatal(err)
	}
	tags := map[string]string{&quot;cpu&quot;: &quot;ih-cpu&quot;}
	fields := map[string]interface{}{
		&quot;idle&quot;:   201.1,
		&quot;system&quot;: 43.3,
		&quot;user&quot;:   86.6,
	}

	pt, err := client.NewPoint(&quot;cpu_usage&quot;, tags, fields, time.Now())
	if err != nil {
		log.Fatal(err)
	}
	bp.AddPoint(pt)
	err = cli.Write(bp)
	if err != nil {
		log.Fatal(err)
	}
	log.Println(&quot;insert success&quot;)
}

func main() {
	conn := connInflux()
	fmt.Println(conn)

	// insert
	writesPoints(conn)

	// 获取10条数据并展示
	qs := fmt.Sprintf(&quot;SELECT * FROM %s LIMIT %d&quot;, &quot;cpu_usage&quot;, 10)
	res, err := queryDB(conn, qs)
	if err != nil {
		log.Fatal(err)
	}

	for _, row := range res[0].Series[0].Values {
		for j, value := range row {
			log.Printf(&quot;j:%d value:%v\n&quot;, j, value)
		}
	}
}
```

# 40.Go第三方日志库logrus

日志是程序中必不可少的一个环节，由于Go语言内置的日志库功能比较简洁，我们在实际开发中通常会选择使用第三方的日志库来进行开发。本文介绍了`logrus`这个日志库的基本使用。

## logrus介绍

Logrus是Go（golang）的结构化logger，与标准库logger完全API兼容。

它有以下特点：

- 完全兼容标准日志库，拥有七种日志级别：`Trace`, `Debug`, `Info`, `Warning`, `Error`, `Fatal`and `Panic`。
- 可扩展的Hook机制，允许使用者通过Hook的方式将日志分发到任意地方，如本地文件系统，logstash，elasticsearch或者mq等，或者通过Hook定义日志内容和格式等
- 可选的日志输出格式，内置了两种日志格式JSONFormater和TextFormatter，还可以自定义日志格式
- Field机制，通过Filed机制进行结构化的日志记录
- 线程安全

## 安装

```bash
Copygo get github.com/sirupsen/logrus
```

## 基本示例

使用Logrus最简单的方法是简单的包级导出日志程序:

```go
Copypackage main

import (
  log &quot;github.com/sirupsen/logrus&quot;
)

func main() {
  log.WithFields(log.Fields{
    &quot;animal&quot;: &quot;dog&quot;,
  }).Info(&quot;一条舔狗出现了。&quot;)
}
```

## 进阶示例

对于更高级的用法，例如在同一应用程序记录到多个位置，你还可以创建logrus Logger的实例：

```go
Copypackage main

import (
  &quot;os&quot;
  &quot;github.com/sirupsen/logrus&quot;
)

// 创建一个新的logger实例。可以创建任意多个。
var log = logrus.New()

func main() {
  // 设置日志输出为os.Stdout
  log.Out = os.Stdout

  // 可以设置像文件等任意`io.Writer`类型作为日志输出
  // file, err := os.OpenFile(&quot;logrus.log&quot;, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
  // if err == nil {
  //  log.Out = file
  // } else {
  //  log.Info(&quot;Failed to log to file, using default stderr&quot;)
  // }

  log.WithFields(logrus.Fields{
    &quot;animal&quot;: &quot;dog&quot;,
    &quot;size&quot;:   10,
  }).Info(&quot;一群舔狗出现了。&quot;)
}
```

## 日志级别

Logrus有七个日志级别：`Trace`, `Debug`, `Info`, `Warning`, `Error`, `Fatal`and `Panic`。

```go
Copylog.Trace(&quot;Something very low level.&quot;)
log.Debug(&quot;Useful debugging information.&quot;)
log.Info(&quot;Something noteworthy happened!&quot;)
log.Warn(&quot;You should probably take a look at this.&quot;)
log.Error(&quot;Something failed but I'm not quitting.&quot;)
// 记完日志后会调用os.Exit(1) 
log.Fatal(&quot;Bye.&quot;)
// 记完日志后会调用 panic() 
log.Panic(&quot;I'm bailing.&quot;)
```

## 设置日志级别

你可以在Logger上设置日志记录级别，然后它只会记录具有该级别或以上级别任何内容的条目：

```go
Copy// 会记录info及以上级别 (warn, error, fatal, panic)
log.SetLevel(log.InfoLevel)
```

如果你的程序支持debug或环境变量模式，设置`log.Level = logrus.DebugLevel`会很有帮助。

## 字段

Logrus鼓励通过日志字段进行谨慎的结构化日志记录，而不是冗长的、不可解析的错误消息。

例如，区别于使用`log.Fatalf("Failed to send event %s to topic %s with key %d")`，你应该使用如下方式记录更容易发现的内容:

```go
Copylog.WithFields(log.Fields{
  &quot;event&quot;: event,
  &quot;topic&quot;: topic,
  &quot;key&quot;: key,
}).Fatal(&quot;Failed to send event&quot;)
```

`WithFields`的调用是可选的。

## 默认字段

通常，将一些字段始终附加到应用程序的全部或部分的日志语句中会很有帮助。例如，你可能希望始终在请求的上下文中记录`request_id`和`user_ip`。

区别于在每一行日志中写上`log.WithFields(log.Fields{"request_id": request_id, "user_ip": user_ip})`，你可以向下面的示例代码一样创建一个`logrus.Entry`去传递这些字段。

```go
CopyrequestLogger := log.WithFields(log.Fields{&quot;request_id&quot;: request_id, &quot;user_ip&quot;: user_ip})
requestLogger.Info(&quot;something happened on that request&quot;) # will log request_id and user_ip
requestLogger.Warn(&quot;something not great happened&quot;)
```

## 日志条目

除了使用`WithField`或`WithFields`添加的字段外，一些字段会自动添加到所有日志记录事中:

- time：记录日志时的时间戳
- msg：记录的日志信息
- level：记录的日志级别

## Hooks

你可以添加日志级别的钩子（Hook）。例如，向异常跟踪服务发送`Error`、`Fatal`和`Panic`、信息到StatsD或同时将日志发送到多个位置，例如syslog。

Logrus配有内置钩子。在`init`中添加这些内置钩子或你自定义的钩子：

```go
Copyimport (
  log &quot;github.com/sirupsen/logrus&quot;
  &quot;gopkg.in/gemnasium/logrus-airbrake-hook.v2&quot; // the package is named &quot;airbrake&quot;
  logrus_syslog &quot;github.com/sirupsen/logrus/hooks/syslog&quot;
  &quot;log/syslog&quot;
)

func init() {

  // Use the Airbrake hook to report errors that have Error severity or above to
  // an exception tracker. You can create custom hooks, see the Hooks section.
  log.AddHook(airbrake.NewHook(123, &quot;xyz&quot;, &quot;production&quot;))

  hook, err := logrus_syslog.NewSyslogHook(&quot;udp&quot;, &quot;localhost:514&quot;, syslog.LOG_INFO, &quot;&quot;)
  if err != nil {
    log.Error(&quot;Unable to connect to local syslog daemon&quot;)
  } else {
    log.AddHook(hook)
  }
}
```

意：Syslog钩子还支持连接到本地syslog（例如. “/dev/log” or “/var/run/syslog” or “/var/run/log”)。有关详细信息，请查看[syslog hook README](https://github.com/sirupsen/logrus/blob/master/hooks/syslog/README.md)。

## 格式化

logrus内置以下两种日志格式化程序：

```
logrus.TextFormatter` `logrus.JSONFormatter
```

还支持一些第三方的格式化程序，详见项目首页。

## 记录函数名

如果你希望将调用的函数名添加为字段，请通过以下方式设置：

```go
Copylog.SetReportCaller(true)
```

这会将调用者添加为”method”，如下所示：

~~~json
Copy{"animal":"penguin","level":"fatal","method":"github.com/sirupsen/arcticcreatures.migrate","msg":"a penguin swims by",
"time":"2014-03-10 19:57:38.562543129 -0400 EDT"}
```

注意：，开启这个模式会增加性能开销。

# 线程安全

默认的logger在并发写的时候是被mutex保护的，比如当同时调用hook和写log时mutex就会被请求，有另外一种情况，文件是以appending mode打开的，
此时的并发操作就是安全的，可以用logger.SetNoLock()来关闭它。

# gin框架使用logrus

```go
// a gin with logrus demo

var log = logrus.New()

func init() {
	// Log as JSON instead of the default ASCII formatter.
	log.Formatter = &logrus.JSONFormatter{}
	// Output to stdout instead of the default stderr
	// Can be any io.Writer, see below for File example
	f, _ := os.Create("./gin.log")
	log.Out = f
	gin.SetMode(gin.ReleaseMode)
	gin.DefaultWriter = log.Out
	// Only log the warning severity or above.
	log.Level = logrus.InfoLevel
}

func main() {
	// 创建一个默认的路由引擎
	r := gin.Default()
	// GET：请求方式；/hello：请求的路径
	// 当客户端以GET方法请求/hello路径时，会执行后面的匿名函数
	r.GET("/hello", func(c *gin.Context) {
		log.WithFields(logrus.Fields{
			"animal": "walrus",
			"size":   10,
		}).Warn("A group of walrus emerges from the ocean")
		// c.JSON：返回JSON格式的数据
		c.JSON(200, gin.H{
			"message": "Hello world!",
		})
	})
	// 启动HTTP服务，默认在0.0.0.0:8080启动服务
	r.Run()
}
```
~~~

# 41.Elasticsearch

本文简单介绍了ES、Kibana和Go语言操作ES。

## 介绍

Elasticsearch（ES）是一个基于Lucene构建的开源、分布式、RESTful接口的全文搜索引擎。Elasticsearch还是一个分布式文档数据库，其中每个字段均可被索引，而且每个字段的数据均可被搜索，ES能够横向扩展至数以百计的服务器存储以及处理PB级的数据。可以在极短的时间内存储、搜索和分析大量的数据。通常作为具有复杂搜索场景情况下的核心发动机。

## Elasticsearch能做什么

1. 当你经营一家网上商店，你可以让你的客户搜索你卖的商品。在这种情况下，你可以使用ElasticSearch来存储你的整个产品目录和库存信息，为客户提供精准搜索，可以为客户推荐相关商品。
2. 当你想收集日志或者交易数据的时候，需要分析和挖掘这些数据，寻找趋势，进行统计，总结，或发现异常。在这种情况下，你可以使用Logstash或者其他工具来进行收集数据，当这引起数据存储到ElasticsSearch中。你可以搜索和汇总这些数据，找到任何你感兴趣的信息。
3. 对于程序员来说，比较有名的案例是GitHub，GitHub的搜索是基于ElasticSearch构建的，在github.com/search页面，你可以搜索项目、用户、issue、pull request，还有代码。共有40~50个索引库，分别用于索引网站需要跟踪的各种数据。虽然只索引项目的主分支（master），但这个数据量依然巨大，包括20亿个索引文档，30TB的索引文件。

## Elasticsearch基本概念

## Near Realtime(NRT) 几乎实时

Elasticsearch是一个几乎实时的搜索平台。意思是，从索引一个文档到这个文档可被搜索只需要一点点的延迟，这个时间一般为毫秒级。

## Cluster 集群

群集是一个或多个节点（服务器）的集合， 这些节点共同保存整个数据，并在所有节点上提供联合索引和搜索功能。一个集群由一个唯一集群ID确定，并指定一个集群名（默认为“elasticsearch”）。该集群名非常重要，因为节点可以通过这个集群名加入群集，一个节点只能是群集的一部分。

确保在不同的环境中不要使用相同的群集名称，否则可能会导致连接错误的群集节点。例如，你可以使用logging-dev、logging-stage、logging-prod分别为开发、阶段产品、生产集群做记录。

## Node节点

节点是单个服务器实例，它是群集的一部分，可以存储数据，并参与群集的索引和搜索功能。就像一个集群，节点的名称默认为一个随机的通用唯一标识符（UUID），确定在启动时分配给该节点。如果不希望默认，可以定义任何节点名。这个名字对管理很重要，目的是要确定你的网络服务器对应于你的ElasticSearch群集节点。

我们可以通过群集名配置节点以连接特定的群集。默认情况下，每个节点设置加入名为“elasticSearch”的集群。这意味着如果你启动多个节点在网络上，假设他们能发现彼此都会自动形成和加入一个名为“elasticsearch”的集群。

在单个群集中，你可以拥有尽可能多的节点。此外，如果“elasticsearch”在同一个网络中，没有其他节点正在运行，从单个节点的默认情况下会形成一个新的单节点名为”elasticsearch”的集群。

## Index索引

索引是具有相似特性的文档集合。例如，可以为客户数据提供索引，为产品目录建立另一个索引，以及为订单数据建立另一个索引。索引由名称（必须全部为小写）标识，该名称用于在对其中的文档执行索引、搜索、更新和删除操作时引用索引。在单个群集中，你可以定义尽可能多的索引。

## Type类型

在索引中，可以定义一个或多个类型。类型是索引的逻辑类别/分区，其语义完全取决于你。一般来说，类型定义为具有公共字段集的文档。例如，假设你运行一个博客平台，并将所有数据存储在一个索引中。在这个索引中，你可以为用户数据定义一种类型，为博客数据定义另一种类型，以及为注释数据定义另一类型。

## Document文档

文档是可以被索引的信息的基本单位。例如，你可以为单个客户提供一个文档，单个产品提供另一个文档，以及单个订单提供另一个文档。本文件的表示形式为JSON（JavaScript Object Notation）格式，这是一种非常普遍的互联网数据交换格式。

在索引/类型中，你可以存储尽可能多的文档。请注意，尽管文档物理驻留在索引中，文档实际上必须索引或分配到索引中的类型。

## Shards & Replicas分片与副本

索引可以存储大量的数据，这些数据可能超过单个节点的硬件限制。例如，十亿个文件占用磁盘空间1TB的单指标可能不适合对单个节点的磁盘或可能太慢服务仅从单个节点的搜索请求。

为了解决这一问题，Elasticsearch提供细分你的指标分成多个块称为分片的能力。当你创建一个索引，你可以简单地定义你想要的分片数量。每个分片本身是一个全功能的、独立的“指数”，可以托管在集群中的任何节点。

**Shards分片的重要性主要体现在以下两个特征：**

1. 分片允许你水平拆分或缩放内容的大小
2. 分片允许你分配和并行操作的碎片（可能在多个节点上）从而提高性能/吞吐量 这个机制中的碎片是分布式的以及其文件汇总到搜索请求是完全由ElasticSearch管理，对用户来说是透明的。

在同一个集群网络或云环境上，故障是任何时候都会出现的，拥有一个故障转移机制以防分片和节点因为某些原因离线或消失是非常有用的，并且被强烈推荐。为此，Elasticsearch允许你创建一个或多个拷贝，你的索引分片进入所谓的副本或称作复制品的分片，简称Replicas。

**Replicas的重要性主要体现在以下两个特征：**

1. 副本为分片或节点失败提供了高可用性。为此，需要注意的是，一个副本的分片不会分配在同一个节点作为原始的或主分片，副本是从主分片那里复制过来的。
2. 副本允许用户扩展你的搜索量或吞吐量，因为搜索可以在所有副本上并行执行。

## ES基本概念与关系型数据库的比较

| ES概念                                         | 关系型数据库       |
| ---------------------------------------------- | ------------------ |
| Index（索引）支持全文检索                      | Database（数据库） |
| Type（类型）                                   | Table（表）        |
| Document（文档），不同文档可以有不同的字段集合 | Row（数据行）      |
| Field（字段）                                  | Column（数据列）   |
| Mapping（映射）                                | Schema（模式）     |

## ES API

以下示例使用`curl`演示。

## 查看健康状态

```bash
Copycurl -X GET 127.0.0.1:9200/_cat/health?v
```

输出：

```bash
Copyepoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1564726309 06:11:49  elasticsearch yellow          1         1      3   3    0    0        1             0                  -                 75.0%
```

## 查询当前es集群中所有的indices

```bash
Copycurl -X GET 127.0.0.1:9200/_cat/indices?v
```

输出：

```bash
Copyhealth status index                uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana_task_manager LUo-IxjDQdWeAbR-SYuYvQ   1   0          2            0     45.5kb         45.5kb
green  open   .kibana_1            PLvyZV1bRDWex05xkOrNNg   1   0          4            1     23.9kb         23.9kb
yellow open   user                 o42mIpDeSgSWZ6eARWUfKw   1   1          0            0       283b           283b
```

## 创建索引

```bash
Copycurl -X PUT 127.0.0.1:9200/www
```

输出：

```bash
Copy{&quot;acknowledged&quot;:true,&quot;shards_acknowledged&quot;:true,&quot;index&quot;:&quot;www&quot;}
```

## 删除索引

```bash
Copycurl -X DELETE 127.0.0.1:9200/www
```

输出：

```bash
Copy{&quot;acknowledged&quot;:true}
```

## 插入记录

```bash
Copycurl -H &quot;ContentType:application/json&quot; -X POST 127.0.0.1:9200/user/person -d '
{
	&quot;name&quot;: &quot;dsb&quot;,
	&quot;age&quot;: 9000,
	&quot;married&quot;: true
}'
```

输出：

```bash
Copy{
    &quot;_index&quot;: &quot;user&quot;,
    &quot;_type&quot;: &quot;person&quot;,
    &quot;_id&quot;: &quot;MLcwUWwBvEa8j5UrLZj4&quot;,
    &quot;_version&quot;: 1,
    &quot;result&quot;: &quot;created&quot;,
    &quot;_shards&quot;: {
        &quot;total&quot;: 2,
        &quot;successful&quot;: 1,
        &quot;failed&quot;: 0
    },
    &quot;_seq_no&quot;: 3,
    &quot;_primary_term&quot;: 1
}
```

也可以使用PUT方法，但是需要传入id

```bash
Copycurl -H &quot;ContentType:application/json&quot; -X PUT 127.0.0.1:9200/user/person/4 -d '
{
	&quot;name&quot;: &quot;sb&quot;,
	&quot;age&quot;: 9,
	&quot;married&quot;: false
}'
```

## 检索

Elasticsearch的检索语法比较特别，使用GET方法携带JSON格式的查询条件。

全检索：

```bash
Copycurl -X GET 127.0.0.1:9200/user/person/_search
```

按条件检索：

```bash
Copycurl -H &quot;ContentType:application/json&quot; -X PUT 127.0.0.1:9200/user/person/4 -d '
{
	&quot;query&quot;:{
		&quot;match&quot;: {&quot;name&quot;: &quot;sb&quot;}
	}	
}'
```

ElasticSearch默认一次最多返回10条结果，可以像下面的示例通过size字段来设置返回结果的数目。

```bash
Copycurl -H &quot;ContentType:application/json&quot; -X PUT 127.0.0.1:9200/user/person/4 -d '
{
	&quot;query&quot;:{
		&quot;match&quot;: {&quot;name&quot;: &quot;sb&quot;},
		&quot;size&quot;: 2
	}	
}'
```

## Go操作Elasticsearch

## elastic client

我们使用第三方库https://github.com/olivere/elastic来连接ES并进行操作。

注意下载与你的ES相同版本的client，例如我们这里使用的ES是7.2.1的版本，那么我们下载的client也要与之对应为`github.com/olivere/elastic/v7`。

使用`go.mod`来管理依赖：

```go
Copyrequire (
    github.com/olivere/elastic/v7 v7.0.4
)
```

简单示例：

```go
Copypackage main

import (
	&quot;context&quot;
	&quot;fmt&quot;

	&quot;github.com/olivere/elastic/v7&quot;
)

// Elasticsearch demo

type Person struct {
	Name    string `json:&quot;name&quot;`
	Age     int    `json:&quot;age&quot;`
	Married bool   `json:&quot;married&quot;`
}

func main() {
	client, err := elastic.NewClient(elastic.SetURL(&quot;http://192.168.1.7:9200&quot;))
	if err != nil {
		// Handle error
		panic(err)
	}

	fmt.Println(&quot;connect to es success&quot;)
	p1 := Person{Name: &quot;rion&quot;, Age: 22, Married: false}
	put1, err := client.Index().
		Index(&quot;user&quot;).
		BodyJson(p1).
		Do(context.Background())
	if err != nil {
		// Handle error
		panic(err)
	}
	fmt.Printf(&quot;Indexed user %s to index %s, type %s\n&quot;, put1.Id, put1.Index, put1.Type)
}
```

更多使用详见文档：https://godoc.org/github.com/olivere/elastic

MySQL是常用的关系型数据库，本文介绍了Go语言如何操作MySQL数据库。

# 42.Go操作MySQL

## 连接

Go语言中的`database/sql`包提供了保证SQL或类SQL数据库的泛用接口，并不提供具体的数据库驱动。使用`database/sql`包时必须注入（至少）一个数据库驱动。

我们常用的数据库基本上都有完整的第三方实现。例如：[MySQL驱动](https://github.com/go-sql-driver/mysql)

## 下载依赖

```bash
Copygo get -u github.com/go-sql-driver/mysql
```

## 使用MySQL驱动

```go
Copyfunc Open(driverName, dataSourceName string) (*DB, error)
```

Open打开一个dirverName指定的数据库，dataSourceName指定数据源，一般包至少括数据库文件名和（可能的）连接信息。

```go
Copyimport (
	&quot;database/sql&quot;

	_ &quot;github.com/go-sql-driver/mysql&quot;
)

func main() {
   // DSN:Data Source Name
	dsn := &quot;user:password@tcp(127.0.0.1:3306)/dbname&quot;
	db, err := sql.Open(&quot;mysql&quot;, dsn)
	if err != nil {
		panic(err)
	}
	defer db.Close()
}
```

## 初始化连接

Open函数可能只是验证其参数，而不创建与数据库的连接。如果要检查数据源的名称是否合法，应调用返回值的Ping方法。

返回的DB可以安全的被多个goroutine同时使用，并会维护自身的闲置连接池。这样一来，Open函数只需调用一次。很少需要关闭DB。

```go
Copy// 定义一个全局对象db
var db *sql.DB

// 定义一个初始化数据库的函数
func initDB() (err error) {
	// DSN:Data Source Name
	dsn := &quot;user:password@tcp(127.0.0.1:3306)/test&quot;
	// 不会校验账号密码是否正确
	// 注意！！！这里不要使用:=，我们是给全局变量赋值，然后在main函数中使用全局变量db
	db, err = sql.Open(&quot;mysql&quot;, dsn)
	if err != nil {
		return err
	}
	// 尝试与数据库建立连接（校验dsn是否正确）
	err = db.Ping()
	if err != nil {
		return err
	}
	return nil
}

func main() {
	err := initDB() // 调用输出化数据库的函数
	if err != nil {
		fmt.Printf(&quot;init db failed,err:%v\n&quot;, err)
		return
	}
}
```

其中`sql.DB`是一个数据库（操作）句柄，代表一个具有零到多个底层连接的连接池。它可以安全的被多个go程同时使用。`database/sql`包会自动创建和释放连接；它也会维护一个闲置连接的连接池。

## SetMaxOpenConns

```go
Copyfunc (db *DB) SetMaxOpenConns(n int)
```

`SetMaxOpenConns`设置与数据库建立连接的最大数目。 如果n大于0且小于最大闲置连接数，会将最大闲置连接数减小到匹配最大开启连接数的限制。 如果n<=0，不会限制最大开启连接数，默认为0（无限制）。

## SetMaxIdleConns

```go
Copyfunc (db *DB) SetMaxIdleConns(n int)
```

SetMaxIdleConns设置连接池中的最大闲置连接数。 如果n大于最大开启连接数，则新的最大闲置连接数会减小到匹配最大开启连接数的限制。 如果n<=0，不会保留闲置连接。

## CRUD

## 建库建表

我们先在MySQL中创建一个名为`sql_test`的数据库

Copy

```go
CREATE DATABASE sql_test; ``` 进入该数据库: Copyuse sql_test; ``` 执行以下命令创建一张用于测试的数据表： CopyCREATE TABLE `user` (    `id` BIGINT(20) NOT NULL AUTO_INCREMENT,    `name` VARCHAR(20) DEFAULT '',    `age` INT(11) DEFAULT '0',    PRIMARY KEY(`id`) )ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4; ``` # 查询 # 单行查询 单行查询db.QueryRow()执行一次查询，并期望返回最多一行结果（即Row）。QueryRow总是返回非nil的值，直到返回值的Scan方法被调用时，才会返回被延迟的错误。（如：未找到结果） ```go func (db *DB) QueryRow(query string, args ...interface{}) *Row ``` 具体示例代码： ```go // 查询单条数据示例 func queryRowDemo() { sqlStr := "select id, name, age from user where id=?" var u user // 非常重要：确保QueryRow之后调用Scan方法，否则持有的数据库链接不会被释放 err := db.QueryRow(sqlStr, 1).Scan(&u.id, &u.name, &u.age) if err != nil { 	fmt.Printf("scan failed, err:%v\n", err) 	return } fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age) } ``` # 多行查询 多行查询db.Query()执行一次查询，返回多行结果（即Rows），一般用于执行select命令。参数args表示query中的占位参数。 ```go func (db *DB) Query(query string, args ...interface{}) (*Rows, error) ``` 具体示例代码： ```go // 查询多条数据示例 func queryMultiRowDemo() { sqlStr := "select id, name, age from user where id > ?" rows, err := db.Query(sqlStr, 0) if err != nil { 	fmt.Printf("query failed, err:%v\n", err) 	return } // 非常重要：关闭rows释放持有的数据库链接 defer rows.Close() 	// 循环读取结果集中的数据 for rows.Next() { 	var u user 	err := rows.Scan(&u.id, &u.name, &u.age) 	if err != nil { 		fmt.Printf("scan failed, err:%v\n", err) 		return 	} 	fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age) } } ``` # 插入数据 插入、更新和删除操作都使用``方法。 ```go func (db *DB) Exec(query string, args ...interface{}) (Result, error) ``` Exec执行一次命令（包括查询、删除、更新、插入等），返回的Result是对已执行的SQL命令的总结。参数args表示query中的占位参数。 具体插入数据示例代码如下： ```go // 插入数据 func insertRowDemo() { sqlStr := "insert into user(name, age) values (?,?)" ret, err := db.Exec(sqlStr, "王五", 38) if err != nil { 	fmt.Printf("insert failed, err:%v\n", err) 	return } theID, err := ret.LastInsertId() // 新插入数据的id if err != nil { 	fmt.Printf("get lastinsert ID failed, err:%v\n", err) 	return } fmt.Printf("insert success, the id is %d.\n", theID) } ``` # 更新数据 具体更新数据示例代码如下： ```go // 更新数据 func updateRowDemo() { sqlStr := "update user set age=? where id = ?" ret, err := db.Exec(sqlStr, 39, 3) if err != nil { 	fmt.Printf("update failed, err:%v\n", err) 	return } n, err := ret.RowsAffected() // 操作影响的行数 if err != nil { 	fmt.Printf("get RowsAffected failed, err:%v\n", err) 	return } fmt.Printf("update success, affected rows:%d\n", n) } ``` # 删除数据 具体删除数据的示例代码如下： ```go // 删除数据 func deleteRowDemo() { sqlStr := "delete from user where id = ?" ret, err := db.Exec(sqlStr, 3) if err != nil { 	fmt.Printf("delete failed, err:%v\n", err) 	return } n, err := ret.RowsAffected() // 操作影响的行数 if err != nil { 	fmt.Printf("get RowsAffected failed, err:%v\n", err) 	return } fmt.Printf("delete success, affected rows:%d\n", n) } ``` # MySQL预处理 # 什么是预处理？ 普通SQL语句执行过程：  客户端对SQL语句进行占位符替换得到完整的SQL语句。 客户端发送完整SQL语句到MySQL服务端 MySQL服务端执行完整的SQL语句并将结果返回给客户端。  预处理执行过程：  把SQL语句分成两部分，命令部分与数据部分。 先把命令部分发送给MySQL服务端，MySQL服务端进行SQL预处理。 然后把数据部分发送给MySQL服务端，MySQL服务端对SQL语句进行占位符替换。 MySQL服务端执行完整的SQL语句并将结果返回给客户端。  # 为什么要预处理？  优化MySQL服务器重复执行SQL的方法，可以提升服务器性能，提前让服务器编译，一次编译多次执行，节省后续编译的成本。 避免SQL注入问题。  # Go实现MySQL预处理 Go中的 ```go func (db *DB) Prepare(query string) (*Stmt, error) ``` Prepare方法会先将sql语句发送给MySQL服务端，返回一个准备好的状态用于之后的查询和命令。返回值可以同时执行多个查询和命令。 查询操作的预处理示例代码如下： ```go // 预处理查询示例 func prepareQueryDemo() { sqlStr := "select id, name, age from user where id > ?" stmt, err := db.Prepare(sqlStr) if err != nil { 	fmt.Printf("prepare failed, err:%v\n", err) 	return } defer stmt.Close() rows, err := stmt.Query(0) if err != nil { 	fmt.Printf("query failed, err:%v\n", err) 	return } defer rows.Close() // 循环读取结果集中的数据 for rows.Next() { 	var u user 	err := rows.Scan(&u.id, &u.name, &u.age) 	if err != nil { 		fmt.Printf("scan failed, err:%v\n", err) 		return 	} 	fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age) } } ``` 插入、更新和删除操作的预处理十分类似，这里以插入操作的预处理为例： ```go // 预处理插入示例 func prepareInsertDemo() { sqlStr := "insert into user(name, age) values (?,?)" stmt, err := db.Prepare(sqlStr) if err != nil { 	fmt.Printf("prepare failed, err:%v\n", err) 	return } defer stmt.Close() _, err = stmt.Exec("小王子", 18) if err != nil { 	fmt.Printf("insert failed, err:%v\n", err) 	return } _, err = stmt.Exec("沙河娜扎", 18) if err != nil { 	fmt.Printf("insert failed, err:%v\n", err) 	return } fmt.Println("insert success.") } ``` # Go实现MySQL事务 # 什么是事务？ 事务：一个最小的不可再分的工作单元；通常一个事务对应一个完整的业务(例如银行账户转账业务，该业务就是一个最小的工作单元)，同时这个完整的业务需要执行多次的DML(insert、update、delete)语句共同联合完成。A转账给B，这里面就需要执行两次update操作。 在MySQL中只有使用了Innodb数据库引擎的数据库或表才支持事务。事务处理可以用来维护数据库的完整性，保证成批的SQL语句要么全部执行，要么全部不执行。 # 事务的ACID 通常事务必须满足4个条件（ACID）：原子性（Atomicity，或称不可分割性）、一致性（Consistency）、隔离性（Isolation，又称独立性）、持久性（Durability）。                条件解释原子性一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。一致性在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。隔离性数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。持久性事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。 # 事务相关方法 Go语言中使用以下三个方法实现MySQL中的事务操作。 开始事务 ```go func (db *DB) Begin() (*Tx, error) ``` 提交事务 ```go func (tx *Tx) Commit() error ``` 回滚事务 ```go func (tx *Tx) Rollback() error ``` # 事务示例 下面的代码演示了一个简单的事务操作，该事物操作能够确保两次更新操作要么同时成功要么同时失败，不会存在中间状态。 ```go // 事务操作示例 func transactionDemo() { tx, err := db.Begin() // 开启事务 if err != nil { 	if tx != nil { 		tx.Rollback() // 回滚 	} 	fmt.Printf("begin trans failed, err:%v\n", err) 	return } sqlStr1 := "Update user set age=30 where id=?" _, err = tx.Exec(sqlStr1, 2) if err != nil { 	tx.Rollback() // 回滚 	fmt.Printf("exec sql1 failed, err:%v\n", err) 	return } sqlStr2 := "Update user set age=40 where id=?" _, err = tx.Exec(sqlStr2, 4) if err != nil { 	tx.Rollback() // 回滚 	fmt.Printf("exec sql2 failed, err:%v\n", err) 	return } err = tx.Commit() // 提交事务 if err != nil { 	tx.Rollback() // 回滚 	fmt.Printf("commit failed, err:%v\n", err) 	return } fmt.Println("exec trans success!") } ``` # sqlx使用 第三方库sqlx能够简化操作，提高开发效率。 # 安装 ```go go get github.com/jmoiron/sqlx ``` # 基本使用 # 连接数据库 ```go var db *sqlx.DB func initDB() (err error) { dsn := "user:password@tcp(127.0.0.1:3306)/test" // 也可以使用MustConnect连接不成功就panic db, err = sqlx.Connect("mysql", dsn) if err != nil { 	fmt.Printf("connect DB failed, err:%v\n", err) 	return } db.SetMaxOpenConns(20) db.SetMaxIdleConns(10) return } ``` # 查询 查询单行数据示例代码如下： ```go // 查询单条数据示例 func queryRowDemo() { sqlStr := "select id, name, age from user where id=?" var u user err := db.Get(&u, sqlStr, 1) if err != nil { 	fmt.Printf("get failed, err:%v\n", err) 	return } fmt.Printf("id:%d name:%s age:%d\n", u.ID, u.Name, u.Age) } ``` 查询多行数据示例代码如下： ```go // 查询多条数据示例 func queryMultiRowDemo() { sqlStr := "select id, name, age from user where id > ?" var users []user err := db.Select(&users, sqlStr, 0) if err != nil { 	fmt.Printf("query failed, err:%v\n", err) 	return } fmt.Printf("users:%#v\n", users) } ``` # 插入、更新和删除 sqlx中的exec方法与原生sql中的exec使用基本一致： ```go // 插入数据 func insertRowDemo() { sqlStr := "insert into user(name, age) values (?,?)" ret, err := db.Exec(sqlStr, "沙河小王子", 19) if err != nil { 	fmt.Printf("insert failed, err:%v\n", err) 	return } theID, err := ret.LastInsertId() // 新插入数据的id if err != nil { 	fmt.Printf("get lastinsert ID failed, err:%v\n", err) 	return } fmt.Printf("insert success, the id is %d.\n", theID) } // 更新数据 func updateRowDemo() { sqlStr := "update user set age=? where id = ?" ret, err := db.Exec(sqlStr, 39, 6) if err != nil { 	fmt.Printf("update failed, err:%v\n", err) 	return } n, err := ret.RowsAffected() // 操作影响的行数 if err != nil { 	fmt.Printf("get RowsAffected failed, err:%v\n", err) 	return } fmt.Printf("update success, affected rows:%d\n", n) } // 删除数据 func deleteRowDemo() { sqlStr := "delete from user where id = ?" ret, err := db.Exec(sqlStr, 6) if err != nil { 	fmt.Printf("delete failed, err:%v\n", err) 	return } n, err := ret.RowsAffected() // 操作影响的行数 if err != nil { 	fmt.Printf("get RowsAffected failed, err:%v\n", err) 	return } fmt.Printf("delete success, affected rows:%d\n", n) } ``` # 事务操作 对于事务操作，我们可以使用sqlx中提供的db.Beginx()和tx.MustExec()方法来简化错误处理过程。示例代码如下： ```go func transactionDemo() { tx, err := db.Beginx() // 开启事务 if err != nil { 	if tx != nil { 		tx.Rollback() 	} 	fmt.Printf("begin trans failed, err:%v\n", err) 	return } sqlStr1 := "Update user set age=40 where id=?" tx.MustExec(sqlStr1, 2) sqlStr2 := "Update user set age=50 where id=?" tx.MustExec(sqlStr2, 4) err = tx.Commit() // 提交事务 if err != nil { 	tx.Rollback() // 回滚 	fmt.Printf("commit failed, err:%v\n", err) 	return } fmt.Println("exec trans success!") } ``` # 注意事项 # SQL中的占位符 不同的数据库中，SQL语句使用的占位符语法不尽相同。               数据库占位符语法MySQL?PostgreSQL$1, $2等SQLite? 和$1Oracle:name # SQL注入 **我们任何时候都不应该自己拼接SQL语句！** 这里我们演示一个自行拼接SQL语句的示例，编写一个根据name字段查询user表的函数如下： ```go // sql注入示例 func sqlInjectDemo(name string) { sqlStr := fmt.Sprintf("select id, name, age from user where name='%s'", name) fmt.Printf("SQL:%s\n", sqlStr) 	var users []user err := db.Select(&users, sqlStr) if err != nil { 	fmt.Printf("exec failed, err:%v\n", err) 	return } for _, u := range users { 	fmt.Printf("user:%#v\n", u) } } ``` 此时以下输入字符串都可以引发SQL注入问题： ```go sqlInjectDemo("xxx' or 1=1#") sqlInjectDemo("xxx' union select * from user #") sqlInjectDemo("xxx' and (select count(*) from user) <10 #") ``` # 练习题  结合net/http和sqlx包实现一个注册及登陆的web程序。
```

# 43.Go操作Redis

在项目开发中redis的使用也比较频繁，本文介绍了Go语言如何操作Redis。

## Redis介绍

Redis是一个开源的内存数据库，Redis提供了多种不同类型的数据结构，很多业务场景下的问题都可以很自然地映射到这些数据结构上。除此之外，通过复制、持久化和客户端分片等特性，我们可以很方便地将Redis扩展成一个能够包含数百GB数据、每秒处理上百万次请求的系统。

## Redis支持的数据结构

Redis支持诸如字符串（strings）、哈希（hashes）、列表（lists）、集合（sets）、带范围查询的排序集合（sorted sets）、位图（bitmaps）、hyperloglogs、带半径查询和流的地理空间索引等数据结构（geospatial indexes）。

## Redis应用场景

- 缓存系统，减轻主数据库（MySQL）的压力。
- 计数场景，比如微博、抖音中的关注数和粉丝数。
- 热门排行榜，需要排序的场景特别适合使用ZSET。
- 利用LIST可以实现队列的功能。

## Redis与Memcached比较

Memcached中的值只支持简单的字符串，Reids支持更丰富的5中数据结构类型。 Redis的性能比Memcached好很多 Redis支持RDB持久化和AOF持久化。 Redis支持master/slave模式。

## Go操作Redis

## 安装

Go语言中使用第三方库https://github.com/go-redis/redis连接Redis数据库并进行操作。使用以下命令下载并安装:

```bash
Copygo get -u github.com/go-redis/redis
```

## 连接

```go
Copy// 声明一个全局的redisdb变量
var redisdb *redis.Client

// 初始化连接
func initClient() (err error) {
	redisdb = redis.NewClient(&amp;redis.Options{
		Addr:     &quot;localhost:6379&quot;,
		Password: &quot;&quot;, // no password set
		DB:       0,  // use default DB
	})

	_, err = redisdb.Ping().Result()
	if err != nil {
		return err
	}
	return nil
}
```

## 基本使用

## set/get示例

```go
Copyfunc redisExample() {
	err := redisdb.Set(&quot;score&quot;, 100, 0).Err()
	if err != nil {
		fmt.Printf(&quot;set score failed, err:%v\n&quot;, err)
		return
	}

	val, err := redisdb.Get(&quot;score&quot;).Result()
	if err != nil {
		fmt.Printf(&quot;get score failed, err:%v\n&quot;, err)
		return
	}
	fmt.Println(&quot;score&quot;, val)

	val2, err := redisdb.Get(&quot;name&quot;).Result()
	if err == redis.Nil {
		fmt.Println(&quot;name does not exist&quot;)
	} else if err != nil {
		fmt.Printf(&quot;get name failed, err:%v\n&quot;, err)
		return
	} else {
		fmt.Println(&quot;name&quot;, val2)
	}
}
```

## zset示例

```go
Copyfunc redisExample2() {
	zsetKey := &quot;language_rank&quot;
	languages := []*redis.Z{
		&amp;redis.Z{Score: 90.0, Member: &quot;Golang&quot;},
		&amp;redis.Z{Score: 98.0, Member: &quot;Java&quot;},
		&amp;redis.Z{Score: 95.0, Member: &quot;Python&quot;},
		&amp;redis.Z{Score: 97.0, Member: &quot;JavaScript&quot;},
		&amp;redis.Z{Score: 99.0, Member: &quot;C/C++&quot;},
	}
	// ZADD
	num, err := redisdb.ZAdd(zsetKey, languages...).Result()
	if err != nil {
		fmt.Printf(&quot;zadd failed, err:%v\n&quot;, err)
		return
	}
	fmt.Printf(&quot;zadd %d succ.\n&quot;, num)

	// 把Golang的分数加10
	newScore, err := redisdb.ZIncrBy(zsetKey, 10.0, &quot;Golang&quot;).Result()
	if err != nil {
		fmt.Printf(&quot;zincrby failed, err:%v\n&quot;, err)
		return
	}
	fmt.Printf(&quot;Golang's score is %f now.\n&quot;, newScore)

	// 取分数最高的3个
	ret, err := redisdb.ZRevRangeWithScores(zsetKey, 0, 2).Result()
	if err != nil {
		fmt.Printf(&quot;zrevrange failed, err:%v\n&quot;, err)
		return
	}
	for _, z := range ret {
		fmt.Println(z.Member, z.Score)
	}

	// 取95~100分的
	op := &amp;redis.ZRangeBy{
		Min: &quot;95&quot;,
		Max: &quot;100&quot;,
	}
	ret, err = redisdb.ZRangeByScoreWithScores(zsetKey, op).Result()
	if err != nil {
		fmt.Printf(&quot;zrangebyscore failed, err:%v\n&quot;, err)
		return
	}
	for _, z := range ret {
		fmt.Println(z.Member, z.Score)
	}
}
```

输出结果如下：

```bash
Copy$ ./06redis_demo 
zadd 0 succ.
Golang's score is 100.000000 now.
Golang 100
C/C++ 99
Java 98
JavaScript 97
Java 98
C/C++ 99
Golang 100
```

更多详情请查阅[文档](https://godoc.org/github.com/go-redis/redis)。

# 44.Go操作etcd

etcd是近几年比较火热的一个开源的、分布式的键值对数据存储系统，提供共享配置、服务的注册和发现，本文主要介绍etcd的安装和使用。

## etcd

## etcd介绍

[etcd](https://etcd.io/)是使用Go语言开发的一个开源的、高可用的分布式key-value存储系统，可以用于配置共享和服务的注册和发现。

类似项目有zookeeper和consul。

etcd具有以下特点：

- 完全复制：集群中的每个节点都可以使用完整的存档
- 高可用性：Etcd可用于避免硬件的单点故障或网络问题
- 一致性：每次读取都会返回跨多主机的最新写入
- 简单：包括一个定义良好、面向用户的API（gRPC）
- 安全：实现了带有可选的客户端证书身份验证的自动化TLS
- 快速：每秒10000次写入的基准速度
- 可靠：使用Raft算法实现了强一致、高可用的服务存储目录

## etcd应用场景

## 服务发现

服务发现要解决的也是分布式系统中最常见的问题之一，即在同一个分布式集群中的进程或服务，要如何才能找到对方并建立连接。本质上来说，服务发现就是想要了解集群中是否有进程在监听 udp 或 tcp 端口，并且通过名字就可以查找和连接。

[![etcd_01.png](http://www.chenyoude.com/go/etcd_01.png)](http://www.chenyoude.com/go/etcd_01.png)

## 配置中心

将一些配置信息放到 etcd 上进行集中管理。

这类场景的使用方式通常是这样：应用在启动的时候主动从 etcd 获取一次配置信息，同时，在 etcd 节点上注册一个 Watcher 并等待，以后每次配置有更新的时候，etcd 都会实时通知订阅者，以此达到获取最新配置信息的目的。

## 分布式锁

因为 etcd 使用 Raft 算法保持了数据的强一致性，某次操作存储到集群中的值必然是全局一致的，所以很容易实现分布式锁。锁服务有两种使用方式，一是保持独占，二是控制时序。

- **保持独占即所有获取锁的用户最终只有一个可以得到**。etcd 为此提供了一套实现分布式锁原子操作 CAS（`CompareAndSwap`）的 API。通过设置`prevExist`值，可以保证在多个节点同时去创建某个目录时，只有一个成功。而创建成功的用户就可以认为是获得了锁。
- 控制时序，即所有想要获得锁的用户都会被安排执行，但是**获得锁的顺序也是全局唯一的，同时决定了执行顺序**。etcd 为此也提供了一套 API（自动创建有序键），对一个目录建值时指定为`POST`动作，这样 etcd 会自动在目录下生成一个当前最大的值为键，存储这个新的值（客户端编号）。同时还可以使用 API 按顺序列出所有当前目录下的键值。此时这些键的值就是客户端的时序，而这些键中存储的值可以是代表客户端的编号。

[![etcd_02.png](http://www.chenyoude.com/go/etcd_02.png)](http://www.chenyoude.com/go/etcd_02.png)

## 为什么用 etcd 而不用ZooKeeper？

etcd 实现的这些功能，ZooKeeper都能实现。那么为什么要用 etcd 而非直接使用ZooKeeper呢？

## 为什么不选择ZooKeeper？

1. 部署维护复杂，其使用的`Paxos`强一致性算法复杂难懂。官方只提供了`Java`和`C`两种语言的接口。
2. 使用`Java`编写引入大量的依赖。运维人员维护起来比较麻烦。
3. 最近几年发展缓慢，不如`etcd`和`consul`等后起之秀。

## 为什么选择etcd？

1. 简单。使用 Go 语言编写部署简单；支持HTTP/JSON API,使用简单；使用 Raft 算法保证强一致性让用户易于理解。
2. etcd 默认数据一更新就进行持久化。
3. etcd 支持 SSL 客户端安全认证。

最后，etcd 作为一个年轻的项目，正在高速迭代和开发中，这既是一个优点，也是一个缺点。优点是它的未来具有无限的可能性，缺点是无法得到大项目长时间使用的检验。然而，目前 `CoreOS`、`Kubernetes`和`CloudFoundry`等知名项目均在生产环境中使用了`etcd`，所以总的来说，etcd值得你去尝试。

## etcd集群

etcd 作为一个高可用键值存储系统，天生就是为集群化而设计的。由于 Raft 算法在做决策时需要多数节点的投票，所以 etcd 一般部署集群推荐奇数个节点，推荐的数量为 3、5 或者 7 个节点构成一个集群。

## 搭建一个3节点集群示例：

在每个etcd节点指定集群成员，为了区分不同的集群最好同时配置一个独一无二的token。

下面是提前定义好的集群信息，其中`n1`、`n2`和`n3`表示3个不同的etcd节点。

```bash
CopyTOKEN=token-01
CLUSTER_STATE=new
CLUSTER=n1=http://10.240.0.17:2380,n2=http://10.240.0.18:2380,n3=http://10.240.0.19:2380
```

在`n1`这台机器上执行以下命令来启动etcd：

```bash
Copyetcd --data-dir=data.etcd --name n1 \
	--initial-advertise-peer-urls http://10.240.0.17:2380 --listen-peer-urls http://10.240.0.17:2380 \
	--advertise-client-urls http://10.240.0.17:2379 --listen-client-urls http://10.240.0.17:2379 \
	--initial-cluster ${CLUSTER} \
	--initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}
```

在`n2`这台机器上执行以下命令启动etcd：

```bash
Copyetcd --data-dir=data.etcd --name n2 \
	--initial-advertise-peer-urls http://10.240.0.18:2380 --listen-peer-urls http://10.240.0.18:2380 \
	--advertise-client-urls http://10.240.0.18:2379 --listen-client-urls http://10.240.0.18:2379 \
	--initial-cluster ${CLUSTER} \
	--initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}
```

在`n3`这台机器上执行以下命令启动etcd：

```bash
Copyetcd --data-dir=data.etcd --name n3 \
	--initial-advertise-peer-urls http://10.240.0.19:2380 --listen-peer-urls http://10.240.0.19:2380 \
	--advertise-client-urls http://10.240.0.19:2379 --listen-client-urls http://10.240.0.19:2379 \
	--initial-cluster ${CLUSTER} \
	--initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}
```

etcd 官网提供了一个可以公网访问的 etcd 存储地址。你可以通过如下命令得到 etcd 服务的目录，并把它作为`-discovery`参数使用。

```bash
Copycurl https://discovery.etcd.io/new?size=3
https://discovery.etcd.io/a81b5818e67a6ea83e9d4daea5ecbc92

# grab this token
TOKEN=token-01
CLUSTER_STATE=new
DISCOVERY=https://discovery.etcd.io/a81b5818e67a6ea83e9d4daea5ecbc92


etcd --data-dir=data.etcd --name n1 \
	--initial-advertise-peer-urls http://10.240.0.17:2380 --listen-peer-urls http://10.240.0.17:2380 \
	--advertise-client-urls http://10.240.0.17:2379 --listen-client-urls http://10.240.0.17:2379 \
	--discovery ${DISCOVERY} \
	--initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}


etcd --data-dir=data.etcd --name n2 \
	--initial-advertise-peer-urls http://10.240.0.18:2380 --listen-peer-urls http://10.240.0.18:2380 \
	--advertise-client-urls http://10.240.0.18:2379 --listen-client-urls http://10.240.0.18:2379 \
	--discovery ${DISCOVERY} \
	--initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}


etcd --data-dir=data.etcd --name n3 \
	--initial-advertise-peer-urls http://10.240.0.19:2380 --listen-peer-urls http://10.240.0.19:2380 \
	--advertise-client-urls http://10.240.0.19:2379 --listen-client-urls http:/10.240.0.19:2379 \
	--discovery ${DISCOVERY} \
	--initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}
```

到此etcd集群就搭建起来了，可以使用`etcdctl`来连接etcd。

```bash
Copyexport ETCDCTL_API=3
HOST_1=10.240.0.17
HOST_2=10.240.0.18
HOST_3=10.240.0.19
ENDPOINTS=$HOST_1:2379,$HOST_2:2379,$HOST_3:2379

etcdctl --endpoints=$ENDPOINTS member list
```

## Go语言操作etcd

这里使用官方的[etcd/clientv3](https://github.com/etcd-io/etcd/tree/master/clientv3)包来连接etcd并进行相关操作。

## 安装

```bash
Copygo get go.etcd.io/etcd/clientv3
```

## put和get操作

`put`命令用来设置键值对数据，`get`命令用来根据key获取值。

```go
Copypackage main

import (
	&quot;context&quot;
	&quot;fmt&quot;
	&quot;time&quot;

	&quot;go.etcd.io/etcd/clientv3&quot;
)

// etcd client put/get demo
// use etcd/clientv3

func main() {
	cli, err := clientv3.New(clientv3.Config{
		Endpoints:   []string{&quot;127.0.0.1:2379&quot;},
		DialTimeout: 5 * time.Second,
	})
	if err != nil {
		// handle error!
		fmt.Printf(&quot;connect to etcd failed, err:%v\n&quot;, err)
		return
	}
    fmt.Println(&quot;connect to etcd success&quot;)
	defer cli.Close()
	// put
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	_, err = cli.Put(ctx, &quot;q1mi&quot;, &quot;dsb&quot;)
	cancel()
	if err != nil {
		fmt.Printf(&quot;put to etcd failed, err:%v\n&quot;, err)
		return
	}
	// get
	ctx, cancel = context.WithTimeout(context.Background(), time.Second)
	resp, err := cli.Get(ctx, &quot;q1mi&quot;)
	cancel()
	if err != nil {
		fmt.Printf(&quot;get from etcd failed, err:%v\n&quot;, err)
		return
	}
	for _, ev := range resp.Kvs {
		fmt.Printf(&quot;%s:%s\n&quot;, ev.Key, ev.Value)
	}
}
```

## watch操作

`watch`用来获取未来更改的通知。

```go
Copypackage main

import (
	&quot;context&quot;
	&quot;fmt&quot;
	&quot;time&quot;

	&quot;go.etcd.io/etcd/clientv3&quot;
)

// watch demo

func main() {
	cli, err := clientv3.New(clientv3.Config{
		Endpoints:   []string{&quot;127.0.0.1:2379&quot;},
		DialTimeout: 5 * time.Second,
	})
	if err != nil {
		fmt.Printf(&quot;connect to etcd failed, err:%v\n&quot;, err)
		return
	}
	fmt.Println(&quot;connect to etcd success&quot;)
	defer cli.Close()
	// watch key:q1mi change
	rch := cli.Watch(context.Background(), &quot;q1mi&quot;) // &lt;-chan WatchResponse
	for wresp := range rch {
		for _, ev := range wresp.Events {
			fmt.Printf(&quot;Type: %s Key:%s Value:%s\n&quot;, ev.Type, ev.Kv.Key, ev.Kv.Value)
		}
	}
}
```

将上面的代码保存编译执行，此时程序就会等待etcd中`q1mi`这个key的变化。

例如：我们打开终端执行以下命令修改、删除、设置`q1mi`这个key。

```bash
Copyetcd&gt; etcdctl.exe --endpoints=http://127.0.0.1:2379 put q1mi &quot;dsb2&quot;
OK

etcd&gt; etcdctl.exe --endpoints=http://127.0.0.1:2379 del q1mi
1

etcd&gt; etcdctl.exe --endpoints=http://127.0.0.1:2379 put q1mi &quot;dsb3&quot;
OK
```

上面的程序都能收到如下通知。

```bash
Copywatch&gt;watch.exe
connect to etcd success
Type: PUT Key:q1mi Value:dsb2
Type: DELETE Key:q1mi Value:
Type: PUT Key:q1mi Value:dsb3
```

## lease租约

```go
Copypackage main

import (
	&quot;fmt&quot;
	&quot;time&quot;
)

// etcd lease

import (
	&quot;context&quot;
	&quot;log&quot;

	&quot;go.etcd.io/etcd/clientv3&quot;
)

func main() {
	cli, err := clientv3.New(clientv3.Config{
		Endpoints:   []string{&quot;127.0.0.1:2379&quot;},
		DialTimeout: time.Second * 5,
	})
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(&quot;connect to etcd success.&quot;)
	defer cli.Close()

	// 创建一个5秒的租约
	resp, err := cli.Grant(context.TODO(), 5)
	if err != nil {
		log.Fatal(err)
	}

	// 5秒钟之后, /nazha/ 这个key就会被移除
	_, err = cli.Put(context.TODO(), &quot;/nazha/&quot;, &quot;dsb&quot;, clientv3.WithLease(resp.ID))
	if err != nil {
		log.Fatal(err)
	}
}
```

## keepAlive

```go
Copypackage main

import (
	&quot;context&quot;
	&quot;fmt&quot;
	&quot;log&quot;
	&quot;time&quot;

	&quot;go.etcd.io/etcd/clientv3&quot;
)

// etcd keepAlive

func main() {
	cli, err := clientv3.New(clientv3.Config{
		Endpoints:   []string{&quot;127.0.0.1:2379&quot;},
		DialTimeout: time.Second * 5,
	})
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(&quot;connect to etcd success.&quot;)
	defer cli.Close()

	resp, err := cli.Grant(context.TODO(), 5)
	if err != nil {
		log.Fatal(err)
	}

	_, err = cli.Put(context.TODO(), &quot;/nazha/&quot;, &quot;dsb&quot;, clientv3.WithLease(resp.ID))
	if err != nil {
		log.Fatal(err)
	}

	// the key 'foo' will be kept forever
	ch, kaerr := cli.KeepAlive(context.TODO(), resp.ID)
	if kaerr != nil {
		log.Fatal(kaerr)
	}
	for {
		ka := &lt;-ch
		fmt.Println(&quot;ttl:&quot;, ka.TTL)
	}
}
```

## 基于etcd实现分布式锁

`go.etcd.io/etcd/clientv3/concurrency`在etcd之上实现并发操作，如分布式锁、屏障和选举。

导入该包：

```go
Copyimport &quot;go.etcd.io/etcd/clientv3/concurrency&quot;
```

基于etcd实现的分布式锁示例：

```go
Copycli, err := clientv3.New(clientv3.Config{Endpoints: endpoints})
if err != nil {
    log.Fatal(err)
}
defer cli.Close()

// 创建两个单独的会话用来演示锁竞争
s1, err := concurrency.NewSession(cli)
if err != nil {
    log.Fatal(err)
}
defer s1.Close()
m1 := concurrency.NewMutex(s1, &quot;/my-lock/&quot;)

s2, err := concurrency.NewSession(cli)
if err != nil {
    log.Fatal(err)
}
defer s2.Close()
m2 := concurrency.NewMutex(s2, &quot;/my-lock/&quot;)

// 会话s1获取锁
if err := m1.Lock(context.TODO()); err != nil {
    log.Fatal(err)
}
fmt.Println(&quot;acquired lock for s1&quot;)

m2Locked := make(chan struct{})
go func() {
    defer close(m2Locked)
    // 等待直到会话s1释放了/my-lock/的锁
    if err := m2.Lock(context.TODO()); err != nil {
        log.Fatal(err)
    }
}()

if err := m1.Unlock(context.TODO()); err != nil {
    log.Fatal(err)
}
fmt.Println(&quot;released lock for s1&quot;)

&lt;-m2Locked
fmt.Println(&quot;acquired lock for s2&quot;)
```

输出：

```bash
Copyacquired lock for s1
released lock for s1
acquired lock for s2
```

[查看文档了解更多](https://godoc.org/go.etcd.io/etcd/clientv3/concurrency)

## 其他操作

其他操作请查看[etcd/clientv3官方文档](https://godoc.org/go.etcd.io/etcd/clientv3)。

参考链接：

- https://etcd.io/docs/v3.3.12/demo/
- https://www.infoq.cn/article/etcd-interpretation-application-scenario-implement-principle/



# 45.Go操作kafka

Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者规模的网站中的所有动作流数据，具有高性能、持久化、多副本备份、横向扩展等特点。本文介绍了如何使用Go语言发送和接收kafka消息。

## sarama

Go语言中连接kafka使用第三方库:[github.com/Shopify/sarama](https://github.com/Shopify/sarama)。

## 下载及安装

```bash
Copygo get github.com/Shopify/sarama
```

## 注意事项

`sarama` v1.20之后的版本加入了`zstd`压缩算法，需要用到cgo，在Windows平台编译时会提示类似如下错误：

```bash
Copy# github.com/DataDog/zstd
exec: &quot;gcc&quot;:executable file not found in %PATH%
```

所以在Windows平台请使用v1.19版本的sarama。

## 连接kafka发送消息

```go
Copypackage main

import (
	&quot;fmt&quot;

	&quot;github.com/Shopify/sarama&quot;
)

// 基于sarama第三方库开发的kafka client

func main() {
	config := sarama.NewConfig()
	config.Producer.RequiredAcks = sarama.WaitForAll          // 发送完数据需要leader和follow都确认
	config.Producer.Partitioner = sarama.NewRandomPartitioner // 新选出一个partition
	config.Producer.Return.Successes = true                   // 成功交付的消息将在success channel返回

	// 构造一个消息
	msg := &amp;sarama.ProducerMessage{}
	msg.Topic = &quot;web_log&quot;
	msg.Value = sarama.StringEncoder(&quot;this is a test log&quot;)
	// 连接kafka
	client, err := sarama.NewSyncProducer([]string{&quot;192.168.1.7:9092&quot;}, config)
	if err != nil {
		fmt.Println(&quot;producer closed, err:&quot;, err)
		return
	}
	defer client.Close()
	// 发送消息
	pid, offset, err := client.SendMessage(msg)
	if err != nil {
		fmt.Println(&quot;send msg failed, err:&quot;, err)
		return
	}
	fmt.Printf(&quot;pid:%v offset:%v\n&quot;, pid, offset)
}
```

## 连接kafka消费消息

```go
Copypackage main

import (
	&quot;fmt&quot;

	&quot;github.com/Shopify/sarama&quot;
)

// kafka consumer

func main() {
	consumer, err := sarama.NewConsumer([]string{&quot;127.0.0.1:9092&quot;}, nil)
	if err != nil {
		fmt.Printf(&quot;fail to start consumer, err:%v\n&quot;, err)
		return
	}
	partitionList, err := consumer.Partitions(&quot;web_log&quot;) // 根据topic取到所有的分区
	if err != nil {
		fmt.Printf(&quot;fail to get list of partition:err%v\n&quot;, err)
		return
	}
	fmt.Println(partitionList)
	for partition := range partitionList { // 遍历所有的分区
		// 针对每个分区创建一个对应的分区消费者
		pc, err := consumer.ConsumePartition(&quot;web_log&quot;, int32(partition), sarama.OffsetNewest)
		if err != nil {
			fmt.Printf(&quot;failed to start consumer for partition %d,err:%v\n&quot;, partition, err)
			return
		}
		defer pc.AsyncClose()
		// 异步从每个分区消费信息
		go func(sarama.PartitionConsumer) {
			for msg := range pc.Messages() {
				fmt.Printf(&quot;Partition:%d Offset:%d Key:%v Value:%v&quot;, msg.Partition, msg.Offset, msg.Key, msg.Value)
			}
		}(pc)
	}
}
```

# 46.Go依赖管理及Go module使用

Go语言的依赖管理随着版本的更迭正逐渐完善起来。

## 依赖管理

## 为什么需要依赖管理

最早的时候，Go所依赖的所有的第三方库都放在GOPATH这个目录下面。这就导致了同一个库只能保存一个版本的代码。如果不同的项目依赖同一个第三方的库的不同版本，应该怎么解决？

## godep

Go语言从v1.5开始开始引入`vendor`模式，如果项目目录下有vendor目录，那么go工具链会优先使用`vendor`内的包进行编译、测试等。

`godep`是一个通过vender模式实现的Go语言的第三方依赖管理工具，类似的还有由社区维护准官方包管理工具`dep`。

## 安装

执行以下命令安装`godep`工具。

```go
Copygo get github.com/tools/godep
```

## 基本命令

安装好godep之后，在终端输入`godep`查看支持的所有命令。

```bash
Copygodep save     将依赖项输出并复制到Godeps.json文件中
godep go       使用保存的依赖项运行go工具
godep get      下载并安装具有指定依赖项的包
godep path     打印依赖的GOPATH路径
godep restore  在GOPATH中拉取依赖的版本
godep update   更新选定的包或go版本
godep diff     显示当前和以前保存的依赖项集之间的差异
godep version  查看版本信息
```

使用`godep help [command]`可以看看具体命令的帮助信息。

## 使用godep

在项目目录下执行`godep save`命令，会在当前项目中创建`Godeps`和`vender`两个文件夹。

其中`Godeps`文件夹下有一个`Godeps.json`的文件，里面记录了项目所依赖的包信息。 `vender`文件夹下是项目依赖的包的源代码文件。

## vender机制

Go1.5版本之后开始支持，能够控制Go语言程序编译时依赖包搜索路径的优先级。

例如查找项目的某个依赖包，首先会在项目根目录下的`vender`文件夹中查找，如果没有找到就会去`$GOAPTH/src`目录下查找。

## godep开发流程

1. 保证程序能够正常编译
2. 执行`godep save`保存当前项目的所有第三方依赖的版本信息和代码
3. 提交Godeps目录和vender目录到代码库。
4. 如果要更新依赖的版本，可以直接修改`Godeps.json`文件中的对应项

## go module

`go module`是Go1.11版本之后官方推出的版本管理工具，并且从Go1.13版本开始，`go module`将是Go语言默认的依赖管理工具。

## GO111MODULE

要启用`go module`支持首先要设置环境变量`GO111MODULE`，通过它可以开启或关闭模块支持，它有三个可选值：`off`、`on`、`auto`，默认值是`auto`。

1. `GO111MODULE=off`禁用模块支持，编译时会从`GOPATH`和`vendor`文件夹中查找包。
2. `GO111MODULE=on`启用模块支持，编译时会忽略`GOPATH`和`vendor`文件夹，只根据 `go.mod`下载依赖。
3. `GO111MODULE=auto`，当项目在`$GOPATH/src`外且项目根目录有`go.mod`文件时，开启模块支持。

简单来说，设置`GO111MODULE=on`之后就可以使用`go module`了，以后就没有必要在GOPATH中创建项目了，并且还能够很好的管理项目依赖的第三方包信息。

使用 go module 管理依赖后会在项目根目录下生成两个文件`go.mod`和`go.sum`。

## GOPROXY

Go1.11之后设置GOPROXY命令为：

```bash
Copyexport GOPROXY=https://goproxy.cn
```

Go1.13之后`GOPROXY`默认值为`https://proxy.golang.org`，在国内是无法访问的，所以十分建议大家设置GOPROXY，这里我推荐使用[goproxy.cn](https://studygolang.com/topics/10014)。

```bash
Copygo env -w GOPROXY=https://goproxy.cn,direct
```

## go mod命令

常用的`go mod`命令如下：

Copy

```
go mod download    下载依赖的module到本地cache（默认为$GOPATH/pkg/mod目录） go mod edit        编辑go.mod文件 go mod graph       打印模块依赖图 go mod init        初始化当前文件夹, 创建go.mod文件 go mod tidy        增加缺少的module，删除无用的module go mod vendor      将依赖复制到vendor下 go mod verify      校验依赖 go mod why         解释为什么需要依赖 ``` # go.mod go.mod文件记录了项目所有的依赖信息，其结构大致如下： Copymodule github.com/Q1mi/studygo/blogger go 1.12 require ( 	github.com/DeanThompson/ginpprof v0.0.0-20190408063150-3be636683586 	github.com/gin-gonic/gin v1.4.0 	github.com/go-sql-driver/mysql v1.4.1 	github.com/jmoiron/sqlx v1.2.0 	github.com/satori/go.uuid v1.2.0 	google.golang.org/appengine v1.6.1 // indirect ) ``` 其中，  module用来定义包名 require用来定义依赖包及版本 indirect表示间接引用  # 依赖的版本 go mod支持语义化版本号，比如go get foo@v1.2.3，也可以跟git的分支或tag，比如go get foo@master，当然也可以跟git提交哈希，比如go get foo@e3702bed2。关于依赖的版本支持以下几种格式： ```go gopkg.in/tomb.v1 v1.0.0-20141024135613-dd632973f1e7 gopkg.in/vmihailenco/msgpack.v2 v2.9.1 gopkg.in/yaml.v2 <=v2.2.1 github.com/tatsushid/go-fastping v0.0.0-20160109021039-d7bb493dee3e latest ``` # replace 在国内访问golang.org/x的各个包都需要FQ，你可以在go.mod中使用replace替换成github上对应的库。 ```go replace ( golang.org/x/crypto v0.0.0-20180820150726-614d502a4dac => github.com/golang/crypto v0.0.0-20180820150726-614d502a4dac golang.org/x/net v0.0.0-20180821023952-922f4815f713 => github.com/golang/net v0.0.0-20180826012351-8a410e7b638d golang.org/x/text v0.3.0 => github.com/golang/text v0.3.0 ) ``` # go get 在项目中执行go get命令可以下载依赖包，并且还可以指定下载的版本。  运行go get -u将会升级到最新的次要版本或者修订版本(x.y.z, z是修订版本号， y是次要版本号) 运行go get -u=patch将会升级到最新的修订版本 运行go get package@version将会升级到指定的版本号version  如果下载所有依赖可以使用go mod download命令。 # 整理依赖 我们在代码中删除依赖代码后，相关的依赖库并不会在go.mod文件中自动移除。这种情况下我们可以使用go mod tidy命令更新go.mod中的依赖关系。 # go mod edit # 格式化 因为我们可以手动修改go.mod文件，所以有些时候需要格式化该文件。Go提供了一下命令： ```bash go mod edit -fmt ``` # 添加依赖项 ```bash go mod edit -require=golang.org/x/text ``` # 移除依赖项 如果只是想修改go.mod文件中的内容，那么可以运行go mod edit -droprequire=package path，比如要在go.mod中移除golang.org/x/text包，可以使用如下命令： ```bash go mod edit -droprequire=golang.org/x/text ``` 关于go mod edit的更多用法可以通过go help mod edit查看。 # 在项目中使用go module # 既有项目 如果需要对一个已经存在的项目启用go module，可以按照以下步骤操作：  在项目目录下执行go mod init，生成一个go.mod文件。 执行go get，查找并记录当前项目的依赖，同时生成一个go.sum记录每个依赖库的版本和哈希值。  # 新项目 对于一个新创建的项目，我们可以在项目文件夹下按照以下步骤操作：  执行go mod init 项目名命令，在当前项目文件夹下创建一个go.mod文件。 手动编辑go.mod中的require依赖项或执行go get自动发现、维护依赖。 
```

# 47.Go pprof性能调优

在计算机性能调试领域里，profiling 是指对应用程序的画像，画像就是应用程序使用 CPU 和内存的情况。 Go语言是一个对性能特别看重的语言，因此语言中自带了 profiling 的库，这篇文章就要讲解怎么在 golang 中做 profiling。

## Go性能优化

Go语言项目中的性能优化主要有以下几个方面：

- CPU profile：报告程序的 CPU 使用情况，按照一定频率去采集应用程序在 CPU 和寄存器上面的数据
- Memory Profile（Heap Profile）：报告程序的内存使用情况
- Block Profiling：报告 goroutines 不在运行状态的情况，可以用来分析和查找死锁等性能瓶颈
- Goroutine Profiling：报告 goroutines 的使用情况，有哪些 goroutine，它们的调用关系是怎样的

## 采集性能数据

Go语言内置了获取程序的运行数据的工具，包括以下两个标准库：

- `runtime/pprof`：采集工具型应用运行数据进行分析
- `net/http/pprof`：采集服务型应用运行时数据进行分析

pprof开启后，每隔一段时间（10ms）就会收集下当前的堆栈信息，获取格格函数占用的CPU以及内存资源；最后通过对这些采样数据进行分析，形成一个性能分析报告。

注意，我们只应该在性能测试的时候才在代码中引入pprof。

## 工具型应用

如果你的应用程序是运行一段时间就结束退出类型。那么最好的办法是在应用退出的时候把 profiling 的报告保存到文件中，进行分析。对于这种情况，可以使用`runtime/pprof`库。 首先在代码中导入`runtime/pprof`工具：

```go
Copyimport &quot;runtime/pprof&quot;
```

## CPU性能分析

开启CPU性能分析：

```go
Copypprof.StartCPUProfile(w io.Writer)
```

停止CPU性能分析：

```go
Copypprof.StopCPUProfile()
```

应用执行结束后，就会生成一个文件，保存了我们的 CPU profiling 数据。得到采样数据之后，使用`go tool pprof`工具进行CPU性能分析。

## 内存性能优化

记录程序的堆栈信息

```go
Copypprof.WriteHeapProfile(w io.Writer)
```

得到采样数据之后，使用`go tool pprof`工具进行内存性能分析。

`go tool pprof`默认是使用`-inuse_space`进行统计，还可以使用`-inuse-objects`查看分配对象的数量。

## 服务型应用

如果你的应用程序是一直运行的，比如 web 应用，那么可以使用`net/http/pprof`库，它能够在提供 HTTP 服务进行分析。

如果使用了默认的`http.DefaultServeMux`（通常是代码直接使用 http.ListenAndServe(“0.0.0.0:8000”, nil)），只需要在你的web server端代码中按如下方式导入`net/http/pprof`

```go
Copyimport _ &quot;net/http/pprof&quot;
```

如果你使用自定义的 Mux，则需要手动注册一些路由规则：

```go
Copyr.HandleFunc(&quot;/debug/pprof/&quot;, pprof.Index)
r.HandleFunc(&quot;/debug/pprof/cmdline&quot;, pprof.Cmdline)
r.HandleFunc(&quot;/debug/pprof/profile&quot;, pprof.Profile)
r.HandleFunc(&quot;/debug/pprof/symbol&quot;, pprof.Symbol)
r.HandleFunc(&quot;/debug/pprof/trace&quot;, pprof.Trace)
```

如果你使用的是gin框架，那么推荐使用`"github.com/DeanThompson/ginpprof"`。

不管哪种方式，你的 HTTP 服务都会多出`/debug/pprof` endpoint，访问它会得到类似下面的内容：[![pprof2.png](http://www.chenyoude.com/go/pprof2.png)](http://www.chenyoude.com/go/pprof2.png)这个路径下还有几个子页面：

- /debug/pprof/profile：访问这个链接会自动进行 CPU profiling，持续 30s，并生成一个文件供下载
- /debug/pprof/heap： Memory Profiling 的路径，访问这个链接会得到一个内存 Profiling 结果的文件
- /debug/pprof/block：block Profiling 的路径
- /debug/pprof/goroutines：运行的 goroutines 列表，以及调用关系

## go tool pprof命令

不管是工具型应用还是服务型应用，我们使用相应的pprof库获取数据之后，下一步的都要对这些数据进行分析，我们可以使用`go tool pprof`命令行工具。

`go tool pprof`最简单的使用方式为:

```bash
Copygo tool pprof [binary] [source]
```

其中：

- binary 是应用的二进制文件，用来解析各种符号；
- source 表示 profile 数据的来源，可以是本地的文件，也可以是 http 地址。

**注意事项：** 获取的 Profiling 数据是动态的，要想获得有效的数据，请保证应用处于较大的负载（比如正在生成中运行的服务，或者通过其他工具模拟访问压力）。否则如果应用处于空闲状态，得到的结果可能没有任何意义。

## 具体示例

首先我们来写一段有问题的代码：

```go
Copy// runtime_pprof/main.go
package main

import (
	&quot;flag&quot;
	&quot;fmt&quot;
	&quot;os&quot;
	&quot;runtime/pprof&quot;
	&quot;time&quot;
)

// 一段有问题的代码
func logicCode() {
	var c chan int
	for {
		select {
		case v := &lt;-c:
			fmt.Printf(&quot;recv from chan, value:%v\n&quot;, v)
		default:

		}
	}
}

func main() {
	var isCPUPprof bool
	var isMemPprof bool

	flag.BoolVar(&amp;isCPUPprof, &quot;cpu&quot;, false, &quot;turn cpu pprof on&quot;)
	flag.BoolVar(&amp;isMemPprof, &quot;mem&quot;, false, &quot;turn mem pprof on&quot;)
	flag.Parse()

	if isCPUPprof {
		file, err := os.Create(&quot;./cpu.pprof&quot;)
		if err != nil {
			fmt.Printf(&quot;create cpu pprof failed, err:%v\n&quot;, err)
			return
		}
		pprof.StartCPUProfile(file)
		defer pprof.StopCPUProfile()
	}
	for i := 0; i &lt; 8; i++ {
		go logicCode()
	}
	time.Sleep(20 * time.Second)
	if isMemPprof {
		file, err := os.Create(&quot;./mem.pprof&quot;)
		if err != nil {
			fmt.Printf(&quot;create mem pprof failed, err:%v\n&quot;, err)
			return
		}
		pprof.WriteHeapProfile(file)
		file.Close()
	}
}
```

通过flag我们可以在命令行控制是否开启CPU和Mem的性能分析。 将上面的代码保存并编译成`runtime_pprof`可执行文件，执行时加上`-cpu`命令行参数如下：

```bash
Copy./runtime_pprof -cpu
```

等待30秒后会在当前目录下生成一个`cpu.pprof`文件。

## 命令行交互界面

我们使用go工具链里的`pprof`来分析一下。

```bash
Copygo tool pprof cpu.pprof
```

执行上面的代码会进入交互界面如下：

```bash
Copyruntime_pprof $ go tool pprof cpu.pprof
Type: cpu
Time: Jun 28, 2019 at 11:28am (CST)
Duration: 20.13s, Total samples = 1.91mins (568.60%)
Entering interactive mode (type &quot;help&quot; for commands, &quot;o&quot; for options)
(pprof)  
```

我们可以在交互界面输入`top3`来查看程序中占用CPU前3位的函数：

```bash
Copy(pprof) top3
Showing nodes accounting for 100.37s, 87.68% of 114.47s total
Dropped 17 nodes (cum &lt;= 0.57s)
Showing top 3 nodes out of 4
      flat  flat%   sum%        cum   cum%
    42.52s 37.15% 37.15%     91.73s 80.13%  runtime.selectnbrecv
    35.21s 30.76% 67.90%     39.49s 34.50%  runtime.chanrecv
    22.64s 19.78% 87.68%    114.37s 99.91%  main.logicCode
```

其中：

- flat：当前函数占用CPU的耗时
- flat：:当前函数占用CPU的耗时百分比
- sun%：函数占用CPU的耗时累计百分比
- cum：当前函数加上调用当前函数的函数占用CPU的总耗时
- cum%：当前函数加上调用当前函数的函数占用CPU的总耗时百分比
- 最后一列：函数名称

在大多数的情况下，我们可以通过分析这五列得出一个应用程序的运行情况，并对程序进行优化。

我们还可以使用`list 函数名`命令查看具体的函数分析，例如执行`list logicCode`查看我们编写的函数的详细分析。

```bash
Copy(pprof) list logicCode
Total: 1.91mins
ROUTINE ================ main.logicCode in .../runtime_pprof/main.go
    22.64s   1.91mins (flat, cum) 99.91% of Total
         .          .     12:func logicCode() {
         .          .     13:   var c chan int
         .          .     14:   for {
         .          .     15:           select {
         .          .     16:           case v := &lt;-c:
    22.64s   1.91mins     17:                   fmt.Printf(&quot;recv from chan, value:%v\n&quot;, v)
         .          .     18:           default:
         .          .     19:
         .          .     20:           }
         .          .     21:   }
         .          .     22:}
```

通过分析发现大部分CPU资源被17行占用，我们分析出select语句中的default没有内容会导致上面的`case v:=<-c:`一直执行。我们在default分支添加一行`time.Sleep(time.Second)`即可。

## 图形化

或者可以直接输入web，通过svg图的方式查看程序中详细的CPU占用情况。 想要查看图形化的界面首先需要安装[graphviz](https://graphviz.gitlab.io/)图形化工具。

Mac：

```bash
Copybrew install graphviz
```

Windows: 下载[graphviz](https://graphviz.gitlab.io/_pages/Download/Download_windows.html) 将`graphviz`安装目录下的bin文件夹添加到Path环境变量中。 在终端输入`dot -version`查看是否安装成功。

[![cpu_pprof.png](http://www.chenyoude.com/go/cpu_pprof.png)](http://www.chenyoude.com/go/cpu_pprof.png)关于图形的说明： 每个框代表一个函数，理论上框的越大表示占用的CPU资源越多。 方框之间的线条代表函数之间的调用关系。 线条上的数字表示函数调用的次数。 方框中的第一行数字表示当前函数占用CPU的百分比，第二行数字表示当前函数累计占用CPU的百分比。

## go-torch和火焰图

火焰图（Flame Graph）是 Bredan Gregg 创建的一种性能分析图表，因为它的样子近似 🔥而得名。上面的 profiling 结果也转换成火焰图，如果对火焰图比较了解可以手动来操作，不过这里我们要介绍一个工具：`go-torch`。这是 uber 开源的一个工具，可以直接读取 golang profiling 数据，并生成一个火焰图的 svg 文件。

## 安装go-touch

```bash
Copy   go get -v github.com/uber/go-torch
```

火焰图 svg 文件可以通过浏览器打开，它对于调用图的最优点是它是动态的：可以通过点击每个方块来 zoom in 分析它上面的内容。

火焰图的调用顺序从下到上，每个方块代表一个函数，它上面一层表示这个函数会调用哪些函数，方块的大小代表了占用 CPU 使用的长短。火焰图的配色并没有特殊的意义，默认的红、黄配色是为了更像火焰而已。

go-torch 工具的使用非常简单，没有任何参数的话，它会尝试从`http://localhost:8080/debug/pprof/profile`获取 profiling 数据。它有三个常用的参数可以调整：

- -u –url：要访问的 URL，这里只是主机和端口部分
- -s –suffix：pprof profile 的路径，默认为 /debug/pprof/profile
- –seconds：要执行 profiling 的时间长度，默认为 30s

## 安装 FlameGraph

要生成火焰图，需要事先安装 FlameGraph工具，这个工具的安装很简单（需要perl环境支持），只要把对应的可执行文件加入到环境变量中即可。

1. 下载安装perl：https://www.perl.org/get.html
2. 下载FlameGraph：`git clone https://github.com/brendangregg/FlameGraph.git`
3. 将`FlameGraph`目录加入到操作系统的环境变量中。
4. Windows平台的同学，需要把`go-torch/render/flamegraph.go`文件中的`GenerateFlameGraph`按如下方式修改，然后在`go-torch`目录下执行`go install`即可。

```go
Copy// GenerateFlameGraph runs the flamegraph script to generate a flame graph SVG. func GenerateFlameGraph(graphInput []byte, args ...string) ([]byte, error) {
flameGraph := findInPath(flameGraphScripts)
if flameGraph == &quot;&quot; {
	return nil, errNoPerlScript
}
if runtime.GOOS == &quot;windows&quot; {
	return runScript(&quot;perl&quot;, append([]string{flameGraph}, args...), graphInput)
}
  return runScript(flameGraph, args, graphInput)
}
```

## 压测工具wrk

推荐使用https://github.com/wg/wrk 或 https://github.com/adjust/go-wrk

## 使用go-torch

使用wrk进行压测:`go-wrk -n 50000 http://127.0.0.1:8080/book/list` 在上面压测进行的同时，打开另一个终端执行`go-torch -u http://127.0.0.1:8080 -t 30`，30秒之后终端会初夏如下提示：`Writing svg to torch.svg`

然后我们使用浏览器打开`torch.svg`就能看到如下火焰图了。[![pprof3.png](http://www.chenyoude.com/go/pprof3.png)](http://www.chenyoude.com/go/pprof3.png)火焰图的y轴表示cpu调用方法的先后，x轴表示在每个采样调用时间内，方法所占的时间百分比，越宽代表占据cpu时间越多。通过火焰图我们就可以更清楚的找出耗时长的函数调用，然后不断的修正代码，重新采样，不断优化。

## pprof与性能测试结合

`go test`命令有两个参数和 pprof 相关，它们分别指定生成的 CPU 和 Memory profiling 保存的文件：

- -cpuprofile：cpu profiling 数据要保存的文件地址
- -memprofile：memory profiling 数据要报文的文件地址

我们还可以选择将pprof与性能测试相结合，比如：

比如下面执行测试的同时，也会执行 CPU profiling，并把结果保存在 cpu.prof 文件中：

```bash
Copygo test -bench . -cpuprofile=cpu.prof
```

比如下面执行测试的同时，也会执行 Mem profiling，并把结果保存在 cpu.prof 文件中：

```bash
Copygo test -bench . -memprofile=./mem.prof
```

需要注意的是，Profiling 一般和性能测试一起使用，这个原因在前文也提到过，只有应用在负载高的情况下 Profiling 才有意义。

## 练习题

1. 使用gin框架编写一个接口，使用`go-wrk`进行压测，使用性能调优工具采集数据绘制出调用图和火焰图。

# 48.Go操作NSQ

NSQ是目前比较流行的一个分布式的消息队列，本文主要介绍了NSQ及Go语言如何操作NSQ。

## NSQ

## NSQ介绍

[NSQ](https://nsq.io/)是Go语言编写的一个开源的实时分布式内存消息队列，其性能十分优异。 NSQ的优势有以下优势：

1. NSQ提倡分布式和分散的拓扑，没有单点故障，支持容错和高可用性，并提供可靠的消息交付保证
2. NSQ支持横向扩展，没有任何集中式代理。
3. NSQ易于配置和部署，并且内置了管理界面。

## NSQ的应用场景

通常来说，消息队列都适用以下场景。

## 异步处理

参照下图利用消息队列把业务流程中的非关键流程异步化，从而显著降低业务请求的响应时间。[![nsq1.png](http://www.chenyoude.com/go/nsq1.png)](http://www.chenyoude.com/go/nsq1.png)

## 应用解耦

通过使用消息队列将不同的业务逻辑解耦，降低系统间的耦合，提高系统的健壮性。后续有其他业务要使用订单数据可直接订阅消息队列，提高系统的灵活性。[![nsq2.png](http://www.chenyoude.com/go/nsq2.png)](http://www.chenyoude.com/go/nsq2.png)

## 流量削峰

类似秒杀（大秒）等场景下，某一时间可能会产生大量的请求，使用消息队列能够为后端处理请求提供一定的缓冲区，保证后端服务的稳定性。[![nsq3.png](http://www.chenyoude.com/go/nsq3.png)](http://www.chenyoude.com/go/nsq3.png)

## 安装

[官方下载页面](https://nsq.io/deployment/installing.html)根据自己的平台下载并解压即可。

## NSQ组件

## nsqd

nsqd是一个守护进程，它接收、排队并向客户端发送消息。

启动`nsqd`，指定`-broadcast-address=127.0.0.1`来配置广播地址

```go
Copy./nsqd -broadcast-address=127.0.0.1
```

如果是在搭配`nsqlookupd`使用的模式下需要还指定`nsqlookupd`地址:

```bash
Copy./nsqd -broadcast-address=127.0.0.1 -lookupd-tcp-address=127.0.0.1:4160
```

如果是部署了多个`nsqlookupd`节点的集群，那还可以指定多个`-lookupd-tcp-address`。

`nsqdq`相关配置项如下：

```go
Copy-auth-http-address value
    &lt;addr&gt;:&lt;port&gt; to query auth server (may be given multiple times)
-broadcast-address string
    address that will be registered with lookupd (defaults to the OS hostname) (default &quot;PROSNAKES.local&quot;)
-config string
    path to config file
-data-path string
    path to store disk-backed messages
-deflate
    enable deflate feature negotiation (client compression) (default true)
-e2e-processing-latency-percentile value
    message processing time percentiles (as float (0, 1.0]) to track (can be specified multiple times or comma separated '1.0,0.99,0.95', default none)
-e2e-processing-latency-window-time duration
    calculate end to end latency quantiles for this duration of time (ie: 60s would only show quantile calculations from the past 60 seconds) (default 10m0s)
-http-address string
    &lt;addr&gt;:&lt;port&gt; to listen on for HTTP clients (default &quot;0.0.0.0:4151&quot;)
-http-client-connect-timeout duration
    timeout for HTTP connect (default 2s)
-http-client-request-timeout duration
    timeout for HTTP request (default 5s)
-https-address string
    &lt;addr&gt;:&lt;port&gt; to listen on for HTTPS clients (default &quot;0.0.0.0:4152&quot;)
-log-prefix string
    log message prefix (default &quot;[nsqd] &quot;)
-lookupd-tcp-address value
    lookupd TCP address (may be given multiple times)
-max-body-size int
    maximum size of a single command body (default 5242880)
-max-bytes-per-file int
    number of bytes per diskqueue file before rolling (default 104857600)
-max-deflate-level int
    max deflate compression level a client can negotiate (&gt; values == &gt; nsqd CPU usage) (default 6)
-max-heartbeat-interval duration
    maximum client configurable duration of time between client heartbeats (default 1m0s)
-max-msg-size int
    maximum size of a single message in bytes (default 1048576)
-max-msg-timeout duration
    maximum duration before a message will timeout (default 15m0s)
-max-output-buffer-size int
    maximum client configurable size (in bytes) for a client output buffer (default 65536)
-max-output-buffer-timeout duration
    maximum client configurable duration of time between flushing to a client (default 1s)
-max-rdy-count int
    maximum RDY count for a client (default 2500)
-max-req-timeout duration
    maximum requeuing timeout for a message (default 1h0m0s)
-mem-queue-size int
    number of messages to keep in memory (per topic/channel) (default 10000)
-msg-timeout string
    duration to wait before auto-requeing a message (default &quot;1m0s&quot;)
-node-id int
    unique part for message IDs, (int) in range [0,1024) (default is hash of hostname) (default 616)
-snappy
    enable snappy feature negotiation (client compression) (default true)
-statsd-address string
    UDP &lt;addr&gt;:&lt;port&gt; of a statsd daemon for pushing stats
-statsd-interval string
    duration between pushing to statsd (default &quot;1m0s&quot;)
-statsd-mem-stats
    toggle sending memory and GC stats to statsd (default true)
-statsd-prefix string
    prefix used for keys sent to statsd (%s for host replacement) (default &quot;nsq.%s&quot;)
-sync-every int
    number of messages per diskqueue fsync (default 2500)
-sync-timeout duration
    duration of time per diskqueue fsync (default 2s)
-tcp-address string
    &lt;addr&gt;:&lt;port&gt; to listen on for TCP clients (default &quot;0.0.0.0:4150&quot;)
-tls-cert string
    path to certificate file
-tls-client-auth-policy string
    client certificate auth policy ('require' or 'require-verify')
-tls-key string
    path to key file
-tls-min-version value
    minimum SSL/TLS version acceptable ('ssl3.0', 'tls1.0', 'tls1.1', or 'tls1.2') (default 769)
-tls-required
    require TLS for client connections (true, false, tcp-https)
-tls-root-ca-file string
    path to certificate authority file
-verbose
    enable verbose logging
-version
    print version string
-worker-id
    do NOT use this, use --node-id
```

## nsqlookupd

nsqlookupd是维护所有nsqd状态、提供服务发现的守护进程。它能为消费者查找特定`topic`下的nsqd提供了运行时的自动发现服务。 它不维持持久状态，也不需要与任何其他nsqlookupd实例协调以满足查询。因此根据你系统的冗余要求尽可能多地部署`nsqlookupd`节点。它们小豪的资源很少，可以与其他服务共存。我们的建议是为每个数据中心运行至少3个集群。

`nsqlookupd`相关配置项如下：

```bash
Copy-broadcast-address string
    address of this lookupd node, (default to the OS hostname) (default &quot;PROSNAKES.local&quot;)
-config string
    path to config file
-http-address string
    &lt;addr&gt;:&lt;port&gt; to listen on for HTTP clients (default &quot;0.0.0.0:4161&quot;)
-inactive-producer-timeout duration
    duration of time a producer will remain in the active list since its last ping (default 5m0s)
-log-prefix string
    log message prefix (default &quot;[nsqlookupd] &quot;)
-tcp-address string
    &lt;addr&gt;:&lt;port&gt; to listen on for TCP clients (default &quot;0.0.0.0:4160&quot;)
-tombstone-lifetime duration
    duration of time a producer will remain tombstoned if registration remains (default 45s)
-verbose
    enable verbose logging
-version
    print version string
```

## nsqadmin

一个实时监控集群状态、执行各种管理任务的Web管理平台。 启动`nsqadmin`，指定`nsqlookupd`地址:

```bash
Copy./nsqadmin -lookupd-http-address=127.0.0.1:4161
```

我们可以使用浏览器打开`http://127.0.0.1:4171/`访问如下管理界面。[![nsqadmin0.png](http://www.chenyoude.com/go/nsqadmin0.png)](http://www.chenyoude.com/go/nsqadmin0.png)

`nsqadmin`相关的配置项如下：

```go
Copy-allow-config-from-cidr string
    A CIDR from which to allow HTTP requests to the /config endpoint (default &quot;127.0.0.1/8&quot;)
-config string
    path to config file
-graphite-url string
    graphite HTTP address
-http-address string
    &lt;addr&gt;:&lt;port&gt; to listen on for HTTP clients (default &quot;0.0.0.0:4171&quot;)
-http-client-connect-timeout duration
    timeout for HTTP connect (default 2s)
-http-client-request-timeout duration
    timeout for HTTP request (default 5s)
-http-client-tls-cert string
    path to certificate file for the HTTP client
-http-client-tls-insecure-skip-verify
    configure the HTTP client to skip verification of TLS certificates
-http-client-tls-key string
    path to key file for the HTTP client
-http-client-tls-root-ca-file string
    path to CA file for the HTTP client
-log-prefix string
    log message prefix (default &quot;[nsqadmin] &quot;)
-lookupd-http-address value
    lookupd HTTP address (may be given multiple times)
-notification-http-endpoint string
    HTTP endpoint (fully qualified) to which POST notifications of admin actions will be sent
-nsqd-http-address value
    nsqd HTTP address (may be given multiple times)
-proxy-graphite
    proxy HTTP requests to graphite
-statsd-counter-format string
    The counter stats key formatting applied by the implementation of statsd. If no formatting is desired, set this to an empty string. (default &quot;stats.counters.%s.count&quot;)
-statsd-gauge-format string
    The gauge stats key formatting applied by the implementation of statsd. If no formatting is desired, set this to an empty string. (default &quot;stats.gauges.%s&quot;)
-statsd-interval duration
    time interval nsqd is configured to push to statsd (must match nsqd) (default 1m0s)
-statsd-prefix string
    prefix used for keys sent to statsd (%s for host replacement, must match nsqd) (default &quot;nsq.%s&quot;)
-version
    print version string
```

## NSQ架构

## NSQ工作模式

[![nsq4.png](http://www.chenyoude.com/go/nsq4.png)](http://www.chenyoude.com/go/nsq4.png)

## Topic和Channel

每个nsqd实例旨在一次处理多个数据流。这些数据流称为`“topics”`，一个`topic`具有1个或多个`“channels”`。每个`channel`都会收到`topic`所有消息的副本，实际上下游的服务是通过对应的`channel`来消费`topic`消息。

`topic`和`channel`不是预先配置的。`topic`在首次使用时创建，方法是将其发布到指定`topic`，或者订阅指定`topic`上的`channel`。`channel`是通过订阅指定的`channel`在第一次使用时创建的。

`topic`和`channel`都相互独立地缓冲数据，防止缓慢的消费者导致其他`chennel`的积压（同样适用于`topic`级别）。

`channel`可以并且通常会连接多个客户端。假设所有连接的客户端都处于准备接收消息的状态，则每条消息将被传递到随机客户端。例如：

[![nsq5.gif](http://www.chenyoude.com/go/nsq5.gif)](http://www.chenyoude.com/go/nsq5.gif)总而言之，消息是从`topic -> channel`（每个channel接收该topic的所有消息的副本）多播的，但是从`channel -> consumers`均匀分布（每个消费者接收该channel的一部分消息）。

## NSQ接收和发送消息流程

[![nsq6.png](http://www.chenyoude.com/go/nsq6.png)](http://www.chenyoude.com/go/nsq6.png)

## NSQ特性

- 消息默认不持久化，可以配置成持久化模式。nsq采用的方式时内存+硬盘的模式，当内存到达一定程度时就会将数据持久化到硬盘。
  - 如果将`--mem-queue-size`设置为0，所有的消息将会存储到磁盘。
  - 服务器重启时也会将当时在内存中的消息持久化。
- 每条消息至少传递一次。
- 消息不保证有序。

## Go操作NSQ

官方提供了Go语言版的客户端：[go-nsq](https://github.com/nsqio/go-nsq)，更多客户端支持请查看[CLIENT LIBRARIES](https://nsq.io/clients/client_libraries.html)。

## 安装

```bash
Copygo get -u github.com/nsqio/go-nsq
```

## 生产者

一个简单的生产者示例代码如下：

```go
Copy// nsq_producer/main.go
package main

import (
	&quot;bufio&quot;
	&quot;fmt&quot;
	&quot;os&quot;
	&quot;strings&quot;

	&quot;github.com/nsqio/go-nsq&quot;
)

// NSQ Producer Demo

var producer *nsq.Producer

// 初始化生产者
func initProducer(str string) (err error) {
	config := nsq.NewConfig()
	producer, err = nsq.NewProducer(str, config)
	if err != nil {
		fmt.Printf(&quot;create producer failed, err:%v\n&quot;, err)
		return err
	}
	return nil
}

func main() {
	nsqAddress := &quot;127.0.0.1:4150&quot;
	err := initProducer(nsqAddress)
	if err != nil {
		fmt.Printf(&quot;init producer failed, err:%v\n&quot;, err)
		return
	}

	reader := bufio.NewReader(os.Stdin) // 从标准输入读取
	for {
		data, err := reader.ReadString('\n')
		if err != nil {
			fmt.Printf(&quot;read string from stdin failed, err:%v\n&quot;, err)
			continue
		}
		data = strings.TrimSpace(data)
		if strings.ToUpper(data) == &quot;Q&quot; { // 输入Q退出
			break
		}
		// 向 'topic_demo' publish 数据
		err = producer.Publish(&quot;topic_demo&quot;, []byte(data))
		if err != nil {
			fmt.Printf(&quot;publish msg to nsq failed, err:%v\n&quot;, err)
			continue
		}
	}
}
```

将上面的代码编译执行，然后在终端输入两条数据`123`和`456`：

```bash
Copy$ ./nsq_producer 
123
2018/10/22 18:41:20 INF    1 (127.0.0.1:4150) connecting to nsqd
456
```

使用浏览器打开`http://127.0.0.1:4171/`可以查看到类似下面的页面： 在下面这个页面能看到当前的`topic`信息：[![nsqadmin1.png](http://www.chenyoude.com/go/nsqadmin1.png)](http://www.chenyoude.com/go/nsqadmin1.png)

点击页面上的`topic_demo`就能进入一个展示更多详细信息的页面，在这个页面上我们可以查看和管理`topic`，同时能够看到目前在`LWZMBP:4151 (127.0.01:4151)`这个`nsqd`上有2条message。又因为没有消费者接入所以暂时没有创建`channel`。[![nsqadmin2.png](http://www.chenyoude.com/go/nsqadmin2.png)](http://www.chenyoude.com/go/nsqadmin2.png)

在`/nodes`这个页面我们能够很方便的查看当前接入`lookupd`的`nsqd`节点。[![nsqadmin3.png](http://www.chenyoude.com/go/nsqadmin3.png)](http://www.chenyoude.com/go/nsqadmin3.png)

这个`/counter`页面显示了处理的消息数量，因为我们没有接入消费者，所以处理的消息数量为0。[![nsqadmin4.png](http://www.chenyoude.com/go/nsqadmin4.png)](http://www.chenyoude.com/go/nsqadmin4.png)

在`/lookup`界面支持创建`topic`和`channel`。[![nsqadmin5.png](http://www.chenyoude.com/go/nsqadmin5.png)](http://www.chenyoude.com/go/nsqadmin5.png)

## 消费者

一个简单的消费者示例代码如下：

```go
Copy// nsq_consumer/main.go
package main

import (
	&quot;fmt&quot;
	&quot;os&quot;
	&quot;os/signal&quot;
	&quot;syscall&quot;
	&quot;time&quot;

	&quot;github.com/nsqio/go-nsq&quot;
)

// NSQ Consumer Demo

// MyHandler 是一个消费者类型
type MyHandler struct {
	Title string
}

// HandleMessage 是需要实现的处理消息的方法
func (m *MyHandler) HandleMessage(msg *nsq.Message) (err error) {
	fmt.Printf(&quot;%s recv from %v, msg:%v\n&quot;, m.Title, msg.NSQDAddress, string(msg.Body))
	return
}

// 初始化消费者
func initConsumer(topic string, channel string, address string) (err error) {
	config := nsq.NewConfig()
	config.LookupdPollInterval = 15 * time.Second
	c, err := nsq.NewConsumer(topic, channel, config)
	if err != nil {
		fmt.Printf(&quot;create consumer failed, err:%v\n&quot;, err)
		return
	}
	consumer := &amp;MyHandler{
		Title: &quot;沙河1号&quot;,
	}
	c.AddHandler(consumer)

	// if err := c.ConnectToNSQD(address); err != nil { // 直接连NSQD
	if err := c.ConnectToNSQLookupd(address); err != nil { // 通过lookupd查询
		return err
	}
	return nil

}

func main() {
	err := initConsumer(&quot;topic_demo&quot;, &quot;first&quot;, &quot;127.0.0.1:4161&quot;)
	if err != nil {
		fmt.Printf(&quot;init consumer failed, err:%v\n&quot;, err)
		return
	}
	c := make(chan os.Signal)        // 定义一个信号的通道
	signal.Notify(c, syscall.SIGINT) // 转发键盘中断信号到c
	&lt;-c                              // 阻塞
}
```

将上面的代码保存之后编译执行，就能够获取之前我们publish的两条消息了：

```bash
Copy$ ./nsq_consumer 
2018/10/22 18:49:06 INF    1 [topic_demo/first] querying nsqlookupd http://127.0.0.1:4161/lookup?topic=topic_demo
2018/10/22 18:49:06 INF    1 [topic_demo/first] (127.0.0.1:4150) connecting to nsqd
沙河1号 recv from 127.0.0.1:4150, msg:123
沙河1号 recv from 127.0.0.1:4150, msg:456
```

同时在nsqadmin的`/counter`页面查看到处理的数据数量为2。[![nsqadmin6.png](http://www.chenyoude.com/go/nsqadmin6.png)](http://www.chenyoude.com/go/nsqadmin6.png)

Cookie和Session是Web开发绕不开的一个环节，本文介绍了Cookie和Session的原理及在Go语言中如何操作Cookie。

# 49.Cookie

## Cookie的由来

HTTP协议是无状态的，这就存在一个问题。

无状态的意思是每次请求都是独立的，它的执行情况和结果与前面的请求和之后的请求都无直接关系，它不会受前面的请求响应情况直接影响，也不会直接影响后面的请求响应情况。

一句有意思的话来描述就是人生只如初见，对服务器来说，每次的请求都是全新的。

状态可以理解为客户端和服务器在某次会话中产生的数据，那无状态的就以为这些数据不会被保留。会话中产生的数据又是我们需要保存的，也就是说要“保持状态”。因此Cookie就是在这样一个场景下诞生。

## Cookie是什么

在 Internet 中，Cookie 实际上是指小量信息，是由 Web 服务器创建的，将信息存储在用户计算机上（客户端）的数据文件。一般网络用户习惯用其复数形式 Cookies，指某些网站为了辨别用户身份、进行 Session 跟踪而存储在用户本地终端上的数据，而这些数据通常会经过加密处理。

## Cookie的机制

Cookie是由服务器端生成，发送给User-Agent（一般是浏览器），浏览器会将Cookie的key/value保存到某个目录下的文本文件内，下次请求同一网站时就发送该Cookie给服务器（前提是浏览器设置为启用cookie）。Cookie名称和值可以由服务器端开发自己定义，这样服务器可以知道该用户是否是合法用户以及是否需要重新登录等，服务器可以设置或读取Cookies中包含信息，借此维护用户跟服务器会话中的状态。

总结一下Cookie的特点：

1. 浏览器发送请求的时候，自动把携带该站点之前存储的Cookie信息。
2. 服务端可以设置Cookie数据。
3. Cookie是针对单个域名的，不同域名之间的Cookie是独立的。
4. Cookie数据可以配置过期时间，过期的Cookie数据会被系统清除。

## 查看Cookie

我们使用Chrome浏览器打开一个网站，打开开发者工具查看该网站保存在我们电脑上的Cookie数据。

## Go操作Cookie

## Cookie

标准库`net/http`中定义了Cookie，它代表一个出现在HTTP响应头中Set-Cookie的值里或者HTTP请求头中Cookie的值的`HTTP cookie`。

```go
Copytype Cookie struct {
    Name       string
    Value      string
    Path       string
    Domain     string
    Expires    time.Time
    RawExpires string
    // MaxAge=0表示未设置Max-Age属性
    // MaxAge&lt;0表示立刻删除该cookie，等价于&quot;Max-Age: 0&quot;
    // MaxAge&gt;0表示存在Max-Age属性，单位是秒
    MaxAge   int
    Secure   bool
    HttpOnly bool
    Raw      string
    Unparsed []string // 未解析的“属性-值”对的原始文本
}
```

## 设置Cookie

`net/http`中提供了如下`SetCookie`函数，它在w的头域中添加Set-Cookie头，该HTTP头的值为cookie。

```go
Copyfunc SetCookie(w ResponseWriter, cookie *Cookie)
```

## 获取Cookie

`Request`对象拥有两个获取Cookie的方法和一个添加Cookie的方法：

获取Cookie的两种方法：

```go
Copy// 解析并返回该请求的Cookie头设置的所有cookie
func (r *Request) Cookies() []*Cookie

// 返回请求中名为name的cookie，如果未找到该cookie会返回nil, ErrNoCookie。
func (r *Request) Cookie(name string) (*Cookie, error)
```

添加Cookie的方法：

```go
Copy// AddCookie向请求中添加一个cookie。
func (r *Request) AddCookie(c *Cookie)
```

## gin框架操作Cookie

```go
Copyimport (
    &quot;fmt&quot;

    &quot;github.com/gin-gonic/gin&quot;
)

func main() {
    router := gin.Default()
    router.GET(&quot;/cookie&quot;, func(c *gin.Context) {
        cookie, err := c.Cookie(&quot;gin_cookie&quot;) // 获取Cookie
        if err != nil {
            cookie = &quot;NotSet&quot;
            // 设置Cookie
            c.SetCookie(&quot;gin_cookie&quot;, &quot;test&quot;, 3600, &quot;/&quot;, &quot;localhost&quot;, false, true)
        }
        fmt.Printf(&quot;Cookie value: %s \n&quot;, cookie)
    })

    router.Run()
}
```

## Session

## Session的由来

Cookie虽然在一定程度上解决了“保持状态”的需求，但是由于Cookie本身最大支持4096字节，以及Cookie本身保存在客户端，可能被拦截或窃取，因此就需要有一种新的东西，它能支持更多的字节，并且他保存在服务器，有较高的安全性。这就是`Session`。

问题来了，基于HTTP协议的无状态特征，服务器根本就不知道访问者是“谁”。那么上述的Cookie就起到桥接的作用。

用户登陆成功之后，我们在服务端为每个用户创建一个特定的session和一个唯一的标识，它们一一对应。其中：

- Session是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中；
- 唯一标识通常称为`Session ID`会写入用户的Cookie中。

这样该用户后续再次访问时，请求会自动携带Cookie数据（其中包含了`Session ID`），服务器通过该`Session ID`就能找到与之对应的Session数据，也就知道来的人是“谁”。

总结而言：Cookie弥补了HTTP无状态的不足，让服务器知道来的人是“谁”；但是Cookie以文本的形式保存在本地，自身安全性较差；所以我们就通过Cookie识别不同的用户，对应的在服务端为每个用户保存一个Session数据，该Session数据中能够保存具体的用户数据信息。

另外，上述所说的Cookie和Session其实是共通性的东西，不限于语言和框架。

## 练习题

1. 编写代码实现一个gin框架版Session中间件。
