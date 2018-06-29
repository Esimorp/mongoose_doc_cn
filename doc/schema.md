# 纲要 Schemas

## 定义你的纲要

要开始使用Mongoose的话，首先需要一个纲要。 每一个纲要都会映射到一个MongoDB的集合(Collection) 并且定义这个集合里面的文档的形态(Shape)

```js
var mongoose = require('mongoose');
  var Schema = mongoose.Schema;

  var blogSchema = new Schema({
    title:  String,
    author: String,
    body:   String,
    comments: [{ body: String, date: Date }],
    date: { type: Date, default: Date.now },
    hidden: Boolean,
    meta: {
      votes: Number,
      favs:  Number
    }
  });
```
如果等下你想要增加额外的键的话，请使用 [Schema#add](http://mongoosejs.com/docs/api.html#schema_Schema-add) 方法

在我们代码中的 blogSchema 对象的每一个键都定义了在文档中会被转换成它所关联的[纲要类别(SchemaType)]()的属性。比方说，我们已经定义了一个会被转换成 字符串纲要类别的属性 `title`
Each key in our code blogSchema defines a property in our documents which will be cast to its associated SchemaType. For example, we've defined a property title which will be cast to the String SchemaType and property date which will be cast to a Date SchemaType. Keys may also be assigned nested objects containing further key/type definitions like the meta property above.



