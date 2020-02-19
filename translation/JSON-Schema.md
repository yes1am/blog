[英文原版](https://json-schema.org/learn/getting-started-step-by-step.html#references)

# 一步步开始

## 1. 介绍

以下的例子，绝不能代表所有 JSON Schema 所能提供的价值。因此，你可能需要深入了解规范本身，在 [https://json-schema.org/specification.html](https://json-schema.org/specification.html) 查看更多。  

让我们假设我们正在和一个 JSON 格式的产品目录打交道，这个目录有一个产品，这个产品有以下的属性:  

- 唯一标识符: `productId`
- 产品名称: `productName`
- 消费零售价: `price`
- 一系列可选的标签: `tags`

例如:  

```json
{
    "productId": 1,
    "productName": "A green door",
    "price": 12.50,
    "tags": ["home", "green"]
}
```

通常来说，上面的例子有一些问题，以下列举一些:  

- 什么是 `productId`
- `productName` 是否是必须的
- `price` 能否为 0
- `tags` 是否都是字符串

当你在讨论数据格式的时候，你需要 **metadata 元信息** 来表示这些 **keys 属性** 的含义，包括这些属性的有效值. **JSON Schema** 是一项被建议的 **IETF** 标准，用于回答这些数据的问题。  

## 2. 开始 Schema

为了开始类型定义，我们先从一个简单的 JSON schema 开始.  

我们从四个关键字属性开始，这些属性也用作 JSON 的属性.  

> 是的，这项标准使用 JSON 数据文档，来描述数据文档，这些数据文档通常也是 JSON 数据，但是也可以是其他任意的内容类型，比如 `text/xml`  

- `$schema` 关键字声明，这个 schema 是根据标准的一个特定的草稿书写的，主要用于很多原因和版本控制(???,翻译不出来)
- `$id` 关键字定义了这个 schema 的 URI，以及这个 schema 中引用的其它 URI 的 base URI  
- `title` 和 `description` 只是描述性的词. 他们不会给要检验的数据添加约束. 这两个关键词用于描述这项 schema 的目的
- `type` 校验关键字定义了这项 JSON 数据的第一个约束，在这个例子中，它表明这个 JSON 数据是一个 JSON 对象

```
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/product.schema.json",
  "title": "Product",
  "description": "A product in the catalog",
  "type": "object"   // 对象
}
```  

当我们开始 schema 之前，我们先介绍以下的术语:  

- [Schema Keyword (Schema 关键字)](https://json-schema.org/draft/2019-09/json-schema-core.html): `$schema` 和 `$id`
- [Schema Annotations (Schema 注释)](https://json-schema.org/draft/2019-09/json-schema-validation.html): `title` 和 `description`
- [Validation Keyword(校验关键字)](https://json-schema.org/draft/2019-09/json-schema-validation.html): `type`

## 3. 定义属性

`productId` 是一个数字，是一个产品的唯一标识。因为它是产品通用的标识，如果一个产品没有这一项，那么将是无意义的，因此，这个属性是**required**.  

在 JSON Schema 中，我们添加:  

- `properties` 校验关键字
- `productId` 属性
    - `description` 注释和 `type` 校验关键字尤其要注意，我们在前一章节中已经说过了它们的含义。
-  `required` 校验关键字列表有 `productId`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {                // 定义属性
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"          // 表明 productId 是个整数
    }
  },
  "required": [ "productId" ]   // 必填的属性
}
```
- `productName` 是一个字符串值，用于描述产品。由于每个产品都有产品名称，所以这也是**rquired**。
- 因为 `required` 校验关键字是一个字符串数组，我们可以添加多个属性。我们现在添加 `productName` 
- 实际上 `productId` 和 `productName` 并没有什么区别，我们为了完整性所以包含这两项，因为计算机关注的是标识符 `productId`，而人们更加关注名称 `productName`。


```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"         // 表明 productName 是个字符串
    }
  },
  // 同时 productName 也是必填的
  "required": [ "productId", "productName" ]
}
```

## 4. 深入属性

对于商家而言，没有免费的产品  

- `price` 属性被添加近来，拥有通用的 `description` 注释和 `type` 检验关键字，这两个在之前都被提到过了。并且，也被添加到了 `required` 数组里了
- 通过 `exclusiveMinimum` 校验关键字，我们指定 `price` 的值必须是大于 0 的值
    - 如果我们想要 `price` 大于等于 0, 那么我们可以使用 `minimum` 关键字


```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"
    },
    "price": {
      "description": "The price of the product",
      "type": "number",    // 表明 price 属性是个数字
      "exclusiveMinimum": 0         // 而且大于0 (不能等于)
    }
  },
  // price 也是必填项
  "required": [ "productId", "productName", "price" ]
}
```

接下来，我们来到了 `tags` 属性  

商家说了:  

- 如果有 tags，那么至少要有一个 tag
- 所有的 tag 必须的唯一的，同属于一个产品的 tag 不能重复
- 所有的 tag 都必须的文本类型
- tags 非常好，但是不一定是 **必填项**

因此:  

- 添加 `tags` 属性，它拥有通用的注释和关键词
- 这一次，`type` 校验关键字是 `array`
- 我们介绍下 `items` 这个校验关键字，这样我们定义数组中的元素类型。在这个例子中，数组元素的 `type` 是 `string`
- `minItems` 校验关键字用来确保数组中至少有一个元素
- `uniqueItem` 校验关键字表明数组中所有的元素都是唯一的
- 因为 `tags` 是可选的，因此我们不把它加到 `required` 中

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"
    },
    "price": {
      "description": "The price of the product",
      "type": "number",
      "exclusiveMinimum": 0
    },
    "tags": {
      "description": "Tags for the product",
      "type": "array",          // tags 是一个数组
      "items": {
        "type": "string"        // 数组元素是字符串
      },
      "minItems": 1,            // 至少有一个元素
      "uniqueItems": true       // 每一个元素都是唯一的
    }
  },
  "required": [ "productId", "productName", "price" ]
}
```

## 5. 数据结构嵌套

到目前为止，我们都在处理一个比较扁平的 JSON 结构，只有一层。这一节示范一下嵌套的数据结构。  

- 新增 `dimensions` 这个属性，因为它的 `type` 校验关键字是 `Object`, 我们可以使用 `properties` 校验关键字来定义一个嵌套的数据结构
    - 我们省略了 `description` 注释关键字来简化这个例子。当然通常建议不省略注释关键字。然而在这个例子中，这些属性名和数据结构大部分开发者都很熟悉。  
- 你会注意到 `required` 检验关键字的作用于适用于 `dimensions` 属性里，而不是在外面


```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"
    },
    "price": {
      "description": "The price of the product",
      "type": "number",
      "exclusiveMinimum": 0
    },
    "tags": {
      "description": "Tags for the product",
      "type": "array",
      "items": {
        "type": "string"
      },
      "minItems": 1,
      "uniqueItems": true
    },
    "dimensions": {           // 外层对象还拥有 dimensions 这个属性
      "type": "object",       // 这个属性的值是一个对象              
      "properties": {
        "length": {
          "type": "number"
        },
        "width": {
          "type": "number"
        },
        "height": {
          "type": "number"
        }
      },
      // dimensions 这个对象的，length, width, height 属性都是必须的
      "required": [ "length", "width", "height" ]
    }
  },
  "required": [ "productId", "productName", "price" ]
}
```

## 6. 引用外部的 Schema

到目前为止我们的 JSON Schema 都是自包含的. 在很多的数据结构中共享 JSON Schema 从而实现中可复用，可阅读和可维护是很常见的。  

在这个例子中，我们介绍一个新的 JSON Schema，里面包含两个属性:  

- 使用 minmimum 校验关键字
- 使用 maximum 校验关键字
- 组合这两个，产生一个 range (范围的校验)

```json
{
  // 前面提到了，id 可以作为引用的 URI
  "$id": "https://example.com/geographical-location.schema.json",
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Longitude and Latitude",
  "description": "A geographical coordinate on a planet (most commonly Earth).",
  "required": [ "latitude", "longitude" ],
  "type": "object",
  "properties": {
    "latitude": {           // 维度在 -90 到 90 之间
      "type": "number",
      "minimum": -90,
      "maximum": 90
    },
    "longitude": {
      "type": "number",     // 经度在 -180 到 180 之间
      "minimum": -180,
      "maximum": 180
    }
  }
}

// 即描述了一个对象，包含经度维度两个属性
```

接下来，我们引用这个 Schema ， 这样就能组合成新的 Schema:  

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"
    },
    "price": {
      "description": "The price of the product",
      "type": "number",
      "exclusiveMinimum": 0
    },
    "tags": {
      "description": "Tags for the product",
      "type": "array",
      "items": {
        "type": "string"
      },
      "minItems": 1,
      "uniqueItems": true
    },
    "dimensions": {
      "type": "object",
      "properties": {
        "length": {
          "type": "number"
        },
        "width": {
          "type": "number"
        },
        "height": {
          "type": "number"
        }
      },
      "required": [ "length", "width", "height" ]
    },
    "warehouseLocation": {
      // 我们新增这个属性
      "description": "Coordinates of the warehouse where the product is located.",
      // 引用上面定义的 Schema
      "$ref": "https://example.com/geographical-location.schema.json"
    }
  },
  "required": [ "productId", "productName", "price" ]
}
```

## 7. 看一下我们定义的 JSON Schema 最终对应的数据格式


```json
{
    "productId": 1,
    "productName": "An ice sculpture",
    "price": 12.50,
    "tags": [ "cold", "ice" ],
    "dimensions": {      // dimensions 属性
      "length": 7.0,
      "width": 12.0,
      "height": 9.5
    },
    "warehouseLocation": {  // warehouseLocation 拥有 latitude 和 longitude 两个属性
      "latitude": -78.75,
      "longitude": 20.4
    }
}
```