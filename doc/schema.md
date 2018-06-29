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


