---
layout: post
title:  "[译]golang使用JSON"
date:   2019-04-08 19:27:02 +0800
comments: true
tags:
- golang
- json
---

#### 原文
https://yourbasic.org/golang/json-example/

#### 默认类型
go类型对应的json类型如下：
- bool 对应json booleans
- float64 对应json numbers
- string 对应json strings
- nil 对应json null

另外， time.Time以及math/big包内的数字类型都能够自动编码成json的strings.

注意json并不支持整型， 经常用浮点型数字近似。

#### 编码（marshal）结构体对象为json
encoding/json包内的json.Marshal函数用于生成json数据。

```
type FruitBasket struct {
    Name    string
    Fruit   []string
    Id      int64  `json:"ref"`
    private string // An unexported field is not encoded.
    Created time.Time
}

basket := FruitBasket{
    Name:    "Standard",
    Fruit:   []string{"Apple", "Banana", "Orange"},
    Id:      999,
    private: "Second-rate",
    Created: time.Now(),
}

var jsonData []byte
jsonData, err := json.Marshal(basket)
if err != nil {
    log.Println(err)
}
fmt.Println(string(jsonData))

// output

{"Name":"Standard","Fruit":["Apple","Banana","Orange"],"ref":999,"Created":"2018-04-09T23:00:00Z"}
```

只有能被json表示的数据能被编码，参考json.Marshal规则（https://golang.org/pkg/encoding/json/#Marshal）

- 只有结构体对象的外部可见的fields能被json获取，其他将被忽略
- json 中的field: 结构体对象的标签名江北json存储而不是其变量名
- 指针会被编码成其指向的值， 或者null如果指针是nil

#### 更美观的打印
上例中，使用json.MarshalIndent替换json.Marshal:

```
jsonData, err := json.MarshalIndent(basket, "", "    ")

//output

{
    "Name": "Standard",
    "Fruit": [
        "Apple",
        "Banana",
        "Orange"
    ],
    "ref": 999,
    "Created": "2018-04-09T23:00:00Z"
}
```

#### 将json反序列化为结构体对象
json.Unmarshal函数将会解析JSON 数据：

```
type FruitBasket struct {
    Name    string
    Fruit   []string
    Id      int64 `json:"ref"`
    Created time.Time
}

jsonData := []byte(`
{
    "Name": "Standard",
    "Fruit": [
        "Apple",
        "Banana",
        "Orange"
    ],
    "ref": 999,
    "Created": "2018-04-09T23:00:00Z"
}`)

var basket FruitBasket
err := json.Unmarshal(jsonData, &basket)
if err != nil {
    log.Println(err)
}
fmt.Println(basket.Name, basket.Fruit, basket.Id)
fmt.Println(basket.Created)

//output

Standard [Apple Banana Orange] 999
2018-04-09 23:00:00 +0000 UTC
```

Unmarshal允许自动分配slice， 对于slice, maps, 指针同样适用。

对于给定的JSON key Foo, Unmarshal 将会按照下面的顺序尝试去匹配结构对象的变量：
- 外部可见的fields,并且结构标签为Foo
- 外部可见的field为Foo
- 外部可见的field为FOO或者FoO或者其他大小写不敏感匹配

只有找到目标类型才会被编码。
- 如果只希望部分fields被选中，这个特性会很有用
- 另外，strut对象外部不可见的fields则不会被影响到

#### Arbitrary objects and arrays
encoding/json 包使用：
- map[string]interface{}来存储复合的json对象
- []interface{} 用来存储json array

如下JSON 数据：

```
{
    "Name": "Eve",
    "Age": 6,
    "Parents": [
        "Alice",
        "Bob"
    ]
}
```

json.Unmarshal函数会解为map， key为string, 值为interface{}类型：

```
map[string]interface{}{
    "Name": "Eve",
    "Age":  6.0,
    "Parents": []interface{}{
        "Alice",
        "Bob",
    },
}
```

使用range表达式和switch来访问和遍历：

```
jsonData := []byte(`{"Name":"Eve","Age":6,"Parents":["Alice","Bob"]}`)

var v interface{}
json.Unmarshal(jsonData, &v)
data := v.(map[string]interface{})

for k, v := range data {
    switch v := v.(type) {
    case string:
        fmt.Println(k, v, "(string)")
    case float64:
        fmt.Println(k, v, "(float64)")
    case []interface{}:
        fmt.Println(k, "(array):")
        for i, u := range v {
            fmt.Println("    ", i, u)
        }
    default:
        fmt.Println(k, v, "(unknown)")
    }
}

//output

Name Eve (string)
Age 6 (float64)
Parents (array):
     0 Alice
     1 Bob
```

#### JSON 文件举例
json.Decoder和json.Encoder提供了json文件的文件流读和写。

- 从Reader(strings.Reader)读取JSON对象的文件流
- 从每个对象中删除Age field
- 将其写入到Writer（os.Stdout）

```
const jsonData = `
    {"Name": "Alice", "Age": 25}
    {"Name": "Bob", "Age": 22}
`
reader := strings.NewReader(jsonData)
writer := os.Stdout

dec := json.NewDecoder(reader)
enc := json.NewEncoder(writer)

for {
    // Read one JSON object and store it in a map.
    var m map[string]interface{}
    if err := dec.Decode(&m); err == io.EOF {
        break
    } else if err != nil {
        log.Fatal(err)
    }

    // Remove all key-value pairs with key == "Age" from the map.
    for k := range m {
        if k == "Age" {
            delete(m, k)
        }
    }

    // Write the map as a JSON object.
    if err := enc.Encode(&m); err != nil {
        log.Println(err)
    }
}

// OUTPUT

{"Name":"Alice"}
{"Name":"Bob"}
```
