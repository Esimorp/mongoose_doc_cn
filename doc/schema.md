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

在我们代码中的 blogSchema 对象的每一个键都定义了在文档中会被转换成它所关联的[纲要类别(SchemaType)]()的属性。比方说，我们已经定义了一个会被转换成 字符串的属性 `title`，和一个会被转换成日期的属性 `date`。键也可以通过使用一个含有键名称和和类别的对象嵌套来定义就像上面的 `meta`属性那样。

下面是允许被使用的纲要类别的列表
* String
* Number
* Date
* Buffer
* Boolean
* Mixed
* ObjectId
* Array
* Decimal128
* Map

关于纲要类别，你可以在[这里]()找到更详细的信息。

纲要不仅仅定义了你的文档的结构或自动转换属性，它还定义了文档实体方法，模型静态方法，复合索引，还有被叫做中间件的生命周期钩子。

## 创建一个模型

要使用我们的纲要定义的话，我们需要把我们的 `blogSchema` 对象传换成一个可以用来使用的模型对象。要实现这个操作的话，我们需要将 `blogSchema` 对象传递到 `mongoose.model(modelName,schema)` 方法中去。

```js
  var Blog = mongoose.model('Blog', blogSchema);
  // 我们的模型 Blog 已经准备好了
```

## 模型实例方法

模型的实例是文档，文档有许多他们自己已经被定义好了的实例方法。我们也可以定义我们自己的文档实例方法。

```js
  // 定义一个纲要
  var animalSchema = new Schema({ name: String, type: String });

  // 在我们的animalSchema对象中的methods属性中定义一个方法
  animalSchema.methods.findSimilarTypes = function(cb) {
    return this.model('Animal').find({ type: this.type }, cb);
  };
```

现在我们的所有animal模型的实例都拥有了一个 `findSimilarTypes` 方法，

```js
  var Animal = mongoose.model('Animal', animalSchema);
  var dog = new Animal({ type: 'dog' });

  dog.findSimilarTypes(function(err, dogs) {
    console.log(dogs); // woof
  });

  // 如果只有这段代码的话并不会输出期望的 woof
```
* 重写一个 `mongoose` 的文档的默认方法可能产生非期望的结果。[详情在这里]()。

* 上面的例子直接使用 `Schema.methods` 属性来保存一个实例方法。你也可以通过使用 `Schema.method()` 工具方法来实现相同的需求，[详情在这里]()。

* 不要使用ES6中的箭头函数来定义方法。箭头函数会阻止你的方法方法绑定 `this` 上下文，所以在你的方法不能通过访问 `this` 来访问到文档对象，上面的例子中如果用箭头函数来声明方法的话将不会工作，

## 模型静态方法
为模型增加静态方法和增加实例方法一样简单，继续来看我们上面的 `animalSchema`的例子，
```js
  // 在我们的animalSchema对象中的statics属性中定义一个方法
  animalSchema.statics.findByName = function(name, cb) {
    return this.find({ name: new RegExp(name, 'i') }, cb);
  };

  var Animal = mongoose.model('Animal', animalSchema);
  Animal.findByName('fido', function(err, animals) {
    console.log(animals);
  });
```
* 不要使用ES6中的箭头函数来定义方法。箭头函数会阻止你的方法方法绑定 `this` 上下文，所以在你的方法不能通过访问 `this` 来访问到文档对象，上面的例子中如果用箭头函数来声明方法的话将不会工作，

## 查询的工具方法
你也可以增加类似模型静态方法的但是是用于`mongoose`查询的查询的工具方法。查询工具方法可以让你拓展`mongoose`的链式查询构造API。

```js
  animalSchema.query.byName = function(name) {
    return this.where({ name: new RegExp(name, 'i') });
  };

  var Animal = mongoose.model('Animal', animalSchema);

  Animal.find().byName('fido').exec(function(err, animals) {
    console.log(animals);
  });

  Animal.findOne().byName('fido').exec(function(err, animal) {
    console.log(animal);
  });
```

## 索引

`MongoDB` 支持二级索引。在 `mongoose`中，我们使用我们的纲要在路径级别或者纲要级别定义这些索引。在纲要级别定义索引是在使用复合索引的时候非常有必要的 

```js
  var animalSchema = new Schema({
    name: String,
    type: String,
    tags: { type: [String], index: true } // 路径级别定义索引
  });

  animalSchema.index({ name: 1, type: -1 }); // 纲要级别定义索引
```

当你的应用被启动起来以后，`mongoose`会使用你在纲要中定义的每一个索引上自动调用`createIndex`方法。 `mongoose`会按照顺序调用 `createIndex`方法。当一个模型的索引创建成功或者失败以后，`mongoose`会发射一个`index`事件给这个模型。这对与开发阶段是非常棒的，但是对于生产环境可能会引起性能问题。你可以通过在你的纲要的选项中将`autoIndex`选项设置为假来禁用这个行为，或者在全局链接的选项中将`autoIndex`选项设置为假。

```js
  mongoose.connect('mongodb://user:pass@localhost:port/database', { autoIndex: false });
  // 或者
  mongoose.createConnection('mongodb://user:pass@localhost:port/database', { autoIndex: false });
  // 或者
  animalSchema.set('autoIndex', false);
  // 或者
  new Schema({..}, { autoIndex: false });
```

当一个模型的索引创建成功或者失败以后，`mongoose`会发射一个`index`事件给这个模型

```js
  // 下面的代码将会造成错误，因为MongoDB默认在_id键上拥有非稀疏索引
  animalSchema.index({ _id: 1 }, { sparse: true });
  var Animal = mongoose.model('Animal', animalSchema);

  Animal.on('index', function(error) {
    // "_id index cannot be sparse"
    console.log(error.message);
  });
```
这里有更多关于[Model#ensureIndexes]() 方法的信息

## 虚拟属性(Virtuals)
虚拟属性是文档中不会被持久化到`MongoDB`的文档属性，访问器(getter)对于格式化或者拼接成员属性非常有帮助，
赋值器(setter)对于将一个不会被持久化的属性的变化映射到两个会被持久化的属性非常有帮助。

``` js
  // 定义一个纲要
  var personSchema = new Schema({
    name: {
      first: String,
      last: String
    }
  });

  // 生成模型
  var Person = mongoose.model('Person', personSchema);

  // 创建一个文档
  var axl = new Person({
    name: { first: 'Axl', last: 'Rose' }
  });
```

当你希望看到一个人的全名被打印出来的时候，你可以这样做
```js
console.log(axl.name.first + ' ' + axl.name.last); // Axl Rose
```
但是每次都要连接名和姓是非常笨重的。并且当你希望对全名进行一些操作的时候，比方说移除所有带音标的字母？这时候虚拟属性访问器可以使你定义一个不会被持久化到`MongoDB`的`fullName`属性

```js
personSchema.virtual('fullName').get(function () {
  return this.name.first + ' ' + this.name.last;
});
```
现在，`mongoose`将会在每次访问`fullName`属性的时候都调用你的访问器方法

```js
console.log(axl.fullName); // Axl Rose
```

当你使用`toJSON()`方法或者`toObject()`方法(或者对一个`mongoose`文档使用`JSON.stringify()`方法)的时候，默认情况下`mongoose`将不会把虚拟属性包含在内，如果你想要将虚拟属性包含在内，那么就将`{virtuals:true}`参数一并传递给`toObject()`方法或者`toJSON()`方法


你也可以为你的虚拟属性增加一个自定义的赋值器，这可以使你通过修改`fullName`虚拟属性来同时修改`first`和`last`两个属性

```js
personSchema.virtual('fullName').
  get(function() { return this.name.first + ' ' + this.name.last; }).
  set(function(v) {
    this.name.first = v.substr(0, v.indexOf(' '));
    this.name.last = v.substr(v.indexOf(' ') + 1);
  });

axl.fullName = 'William Rose'; // 现在 `axl.name.first` 变成了 "William"
```

使用虚拟属性赋值器修改属性的话会在其它的验证之前生效，所以当上面例子中的`first`和`last`属性被标记成非空的时候，例子也会生效。

只有非虚拟属性的属性能够在查询或者选择属性中使用。只要虚拟属性没有在`MongoDB`中持久化，你就不能去查询它们。

## 别名

别名是一种特别的虚拟属性。它的作用是为了节省网络的带宽，你可以使用短名来持久化在`MongoDB`中，然后使用一个长名来增加代码的可读性。

```js
var personSchema = new Schema({
  n: {
    type: String,
    // 现在可以通过访问`name`属性来获取`n`属性的值，也可以通过设置`n`属性的值来设置`name`属性的值
    alias: 'name'
  }
});

// 对于`name`属性的修改将会被传到到`n`属性
var person = new Person({ name: 'Val' });
console.log(person); // { n: 'Val' }
console.log(person.toObject({ virtuals: true })); // { n: 'Val', name: 'Val' }
console.log(person.name); // "Val"

person.name = 'Not Val';
console.log(person); // { n: 'Not Val' }
```

## 选项
纲要拥有一些可配置的选项，你可以通过构造方法传递或者直接调用`set`方法
Schemas have a few configurable options which can be passed to the constructor or set directly:

``` js
new Schema({..}, options);

// or

var schema = new Schema({..});
schema.set(option, value);
```

可以使用的选项

* autoIndex
* bufferCommands
* capped
* collection
* id
* _id
* minimize
* read
* shardKey
* strict
* strictQuery
* toJSON
* toObject
* typeKey
* validateBeforeSave
* versionKey
* collation
* skipVersioning
* timestamps

### 选项: autoIndex

当应用启动的时候，`mongoose`会给你在你的纲要上声明的每一个索引发送一个`createIndex`命令。在`mongoose` 3.x版本中，索引会默认在后台创建。如果你希望禁用自动创建功能并且手动处理索引创建的时机的话，你需要将你的纲要的`autoIndex`选项设置为假，并且在你的模型上手动调用`ensureIndexes`

```js
var schema = new Schema({..}, { autoIndex: false });
var Clock = mongoose.model('Clock', schema);
Clock.ensureIndexes(callback);
```

### 选项：bufferCommands

默认情况下，当数据库连接断开，`mongoose`将会缓存命令直到驱动重新连接到数据库。想要禁止缓存命令的话，你需要将`bufferCommands`选项设置为假

```js
var schema = new Schema({..}, { bufferCommands: false });
```
纲要级别的`bufferCommands`选项将会覆盖全局的`bufferCommands`选项

```js
mongoose.set('bufferCommands', true);
// 上面的全局设置将被下面的纲要设置覆盖
var schema = new Schema({..}, { bufferCommands: false });
```

### 选项：capped

`mongoose` 支持 `MongoDB`的定长集合，如果你想将集合指定为 `MongoDB` 底层的定长集合，你需要为`capped`选项以`字节`为单位设置集合的最大大小
```js
new Schema({..}, { capped: 1024 });
```

### 选项 collection

`mongoose`默认通过将模型名称传递给`utils.toCollectionName`方法以用来提供一个集合名称。这个方法会把模型的名称变为复数形式。如果你需要一个不一样的集合名称的话，你需要在 `collection` 选项中设置它。

```js
var dataSchema = new Schema({..}, { collection: 'data' });
```

### 选项 id

`mongoose` 会默认在你的每一个纲要上声明一个虚拟访问器，这个访问器会将文档的`_id`属性转换成字符串返回，当文档的`_id`属性为`ObjectID`类型时，它会返回`ObjectId`的十六进制字符串。如果你不希望这个名为`id`的虚拟属性访问器被添加到你的纲要中，你可以在纲要被创建的时候设置`id`选项的值为假。

```js
// 默认情况下
var schema = new Schema({ name: String });
var Page = mongoose.model('Page', schema);
var p = new Page({ name: 'mongodb.org' });
console.log(p.id); // '50341373e894ad16347efe01'

// 禁用id虚拟属性访问器
var schema = new Schema({ name: String }, { id: false });
var Page = mongoose.model('Page', schema);
var p = new Page({ name: 'mongodb.org' });
console.log(p.id); // undefined
```

### 选项 _id

对于你所声明的每一个纲要，若果你没有在这个纲要的构造中声明一个`_id`属性的话，`mongoose`默认会在这个纲要上声明一个_id属性。和`MongoDB`的默认行为一样，这个`_id`属性的类别会被声明为`ObjectId`。如果你不希望这个`_id`属性被添加到你的纲要中的话，你可以使用 `_id`选项来禁用它。

你**只可以**在子文档(subdocuments)中使用这个选项。`mongoose`在不知道一个文档的id的情况下不能对这个文档进行储存，所以当你尝试着去保存一个没有`_id`属性的文档的时候，你将会得到一个错误。

```js
// 默认行为
var schema = new Schema({ name: String });
var Page = mongoose.model('Page', schema);
var p = new Page({ name: 'mongodb.org' });
console.log(p); // { _id: '50341373e894ad16347efe01', name: 'mongodb.org' }

// 禁用 _id
var childSchema = new Schema({ name: String }, { _id: false });
var parentSchema = new Schema({ children: [childSchema] });

var Model = mongoose.model('Model', parentSchema);

Model.create({ children: [{ name: 'Luke' }] }, function(error, doc) {
  // doc.children[0]._id will be undefined
});
```

### 选项 minimize

`mongoose`默认会通过移除空对象来最小化纲要

```js
var schema = new Schema({ name: String, inventory: {} });
var Character = mongoose.model('Character', schema);

// 当inventory属性不为空的情况下，它将会被持久化
var frodo = new Character({ name: 'Frodo', inventory: { ringOfPower: 1 }});
Character.findOne({ name: 'Frodo' }, function(err, character) {
  console.log(character); // { name: 'Frodo', inventory: { ringOfPower: 1 }}
});

// 当inventory属性为空的情况下，它将不会被持久化
var sam = new Character({ name: 'Sam', inventory: {}});
Character.findOne({ name: 'Sam' }, function(err, character) {
  console.log(character); // { name: 'Sam' }
});
```

通过将`minimize`选项设置为假来让空属性可以被持久化

```js
var schema = new Schema({ name: String, inventory: {} }, { minimize: false });
var Character = mongoose.model('Character', schema);

// 即使inventory属性为空，它也将被持久化
var sam = new Character({ name: 'Sam', inventory: {}});
Character.findOne({ name: 'Sam' }, function(err, character) {
  console.log(character); // { name: 'Sam', inventory: {}}
});
```
### 选项 read

通过在纲要级别设置[query#read]() 选项来让我们设置一个模型的全部查询的 [读取优先级]()

```js
var schema = new Schema({..}, { read: 'primary' });            // 可以简写为 'p'
var schema = new Schema({..}, { read: 'primaryPreferred' });   // 可以简写为 'pp'
var schema = new Schema({..}, { read: 'secondary' });          // 可以简写为 's'
var schema = new Schema({..}, { read: 'secondaryPreferred' }); // 可以简写为 'sp'
var schema = new Schema({..}, { read: 'nearest' });            // 可以简写为 'n'
```

为了防止拼写错误，每个选项都提供了一个别名，就像`secondaryPreferred`可以简写为`sp`

选项 `read` 也允许我们使用 `tag` 组。这些`tag`组会告知驱动它应该从哪里去尝试读取数据。关于`tag`组的信息可以在[这里]()和[这里]()找到更多。

注意：你也可以在驱动连接的时候指定一个读取策略

```js
// 每隔一段时间检测连接
var options = { replset: { strategy: 'ping' }};
mongoose.connect(uri, options);

var schema = new Schema({..}, { read: ['nearest', { disk: 'ssd' }] });
mongoose.model('JellyBean', schema);
```

### 选项 shardKey

当你为你的`MongoDB`配置了分片策略后，你可以使用`sharedKey`选项。 每一个分片集合在进行插入或更新操作时，都必须要有一个分片键(shard key)。 我们只需要设置这个纲要选项，`mongoose`就会完成其余工作

注意，`mongoose`并不会为你发送`shardcollection`命令，你必须自己配置分片。

### 选项 strict

严格模式在`mongoose`中是默认开启的，它会确保那些没有在我们的纲要中定义并且被赋予给纲要的模型的属性不会被持久化到`MongoDB`

```js
var thingSchema = new Schema({..})
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing({ iAmNotInTheSchema: true });
thing.save(); // iAmNotInTheSchema 属性不会被持久化到数据库

// 将严格模式关闭
var thingSchema = new Schema({..}, { strict: false });
var thing = new Thing({ iAmNotInTheSchema: true });
thing.save(); // 现在iAmNotInTheSchema属性将会被持久化到数据库
```

严格模式同样对 `doc.set()`方法生效

```js
var thingSchema = new Schema({..})
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing;
thing.set('iAmNotInTheSchema', true);
thing.save(); // iAmNotInTheSchema 属性不会被持久化到数据库
```

严格模式的选项值可以通过在模型的构造方法中传递第二个布尔类型参数从而在模型级别被覆盖

```js
var Thing = mongoose.model('Thing');
var thing = new Thing(doc, true);  // 启动严格模式
var thing = new Thing(doc, false); // 禁用严格模式
```
严格模式的选项也可以被赋值为`"throw"`，如果这样设置的话，当你在模型中储存一个没有在纲要中声明的属性时，`mongoose`会抛出异常而不是简单的无视这些属性。

注意，通过`.`方式直接赋值给模型实例的键和值都将会被忽视，即使你在纲要级别将`strict`设置为假

```js
var thingSchema = new Schema({..})
var Thing = mongoose.model('Thing', thingSchema);
var thing = new Thing;
thing.iAmNotInTheSchema = true;
thing.save(); // iAmNotInTheSchema 属性不会被持久化到数据库
```

### 选项 strictQuery

为了向后兼容，`strict`选项并不会对查询的`filter`参数产生影响

```js
const mySchema = new Schema({ field: Number }, { strict: true });
const MyModel = mongoose.model('Test', mySchema);

// mongoose将**不会** 丢弃 `notInSchema: 1`参数，尽管有了 `strict: true` 选项
MyModel.find({ notInSchema: 1 });
```

但是`strict`选项会影响文档更新

```js
// Mongoose将会在更新操作中舍弃 `notInSchema`参数如果`strict`选项的值非假的话
MyModel.updateMany({}, { $set: { notInSchema: 1 } });
```

`mongoose`有一个单独的 `strictQuery`选项来作为严格模式是否会影响查询的`filter`参数的开关

```js
const mySchema = new Schema({ field: Number }, {
  strict: true,
  strictQuery: true //为所有查询打开严格模式
});
const MyModel = mongoose.model('Test', mySchema);

// mongoose 将会舍弃 `notInSchema: 1` 参数因为`strictQuery`选项的值为真
MyModel.find({ notInSchema: 1 });
```

### 选项 toObject

文档拥有一个`toObject`方法，它可以将`mongoose`文档对象转化为一个标准的`javascript`对象。这个方法可以接受一些参数。预期在每一个文档对象中设置这个选项，我们更应该在纲要级别设置这个选项，这样这个纲要所有的文档都会被默认设置。


想要让所有的虚拟属性在`console.log`方法中输出的话，只需要将纲要的`toObject`选项赋值为`{ getters: true }`

```js
var schema = new Schema({ name: String });
schema.path('name').get(function (v) {
  return v + ' is my name';
});
schema.set('toObject', { getters: true });
var M = mongoose.model('Person', schema);
var m = new M({ name: 'Max Headroom' });
console.log(m); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom is my name' }
```

[这里]()有全部的`toObject`选项可选值。

### 选项 toJSON

就像`toObject`选项一样，但是仅仅会在文档的`toJSON`方法被调用时生效

```js
var schema = new Schema({ name: String });
schema.path('name').get(function (v) {
  return v + ' is my name';
});
schema.set('toJSON', { getters: true, virtuals: false });
var M = mongoose.model('Person', schema);
var m = new M({ name: 'Max Headroom' });
console.log(m.toObject()); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom' }
console.log(m.toJSON()); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom is my name' }
// since we know toJSON is called whenever a js object is stringified:
console.log(JSON.stringify(m)); // { "_id": "504e0cd7dd992d9be2f20b6f", "name": "Max Headroom is my name" }
```

想要查看所有的`toJSON/toObject`选项的可选值，请看[这里]()。

### 选项 typeKey

默认情况下，如果你的对象在纲要中有一个名为`type`的键，`mongoose`会认为`type`的键和值是纲要类别的声明。

```js
// mongoose会认为loc是字符串类别，而不会认为loc是一个对象类别，有两个属性type和coordinates
var schema = new Schema({ loc: { type: String, coordinates: [Number] } });
```

然而对于像`geoJSON`一样的应用，`type`属性是非常关键的。如果你想控制`mongoose`应该用哪个键来读取纲要类别声明的话，你只需要在纲要选项中设置`typeKey`选项

```js
var schema = new Schema({
  //mongoose 会认为loc是一个对象类别，有两个属性type和coordinates
  loc: { type: String, coordinates: [Number] },
  // mongoose 会认为name是字符串类型的纲要类别
  name: { $type: String }
}, { typeKey: '$type' }); // 设置'$type'键为纲要类别声明键
```

### 选项 versionKey

`versionKey`是一个当文档第一次被`mongoose`创建的时候会被设置的属性。这个属性的值记录了文档在`mongoose`内部被修改的次数。`versionKey`选项的值是一个字符串类型的值，用来声明哪个属性(path)应该被用来记录版本。这个值默认为`__v`。如果这个值和你的应用产生冲突，你可以像这样设置你的纲要来避免。

```js
var schema = new Schema({ name: 'string' });
var Thing = mongoose.model('Thing', schema);
var thing = new Thing({ name: 'mongoose v3' });
thing.save(); // { __v: 0, name: 'mongoose v3' }

// 设置自定义的 versionKey
new Schema({..}, { versionKey: '_somethingElse' })
var Thing = mongoose.model('Thing', schema);
var thing = new Thing({ name: 'mongoose v3' });
thing.save(); // { _somethingElse: 0, name: 'mongoose v3' }
```

文档的版本管理可以被禁用。 但首先你需要明确的知道[你正在干什么]();

```js
new Schema({..}, { versionKey: false });
var Thing = mongoose.model('Thing', schema);
var thing = new Thing({ name: 'no versioning please' });
thing.save(); // { name: 'no versioning please' }
```

### 选项 collation

为一个纲要的所有查询和聚合操作设置一个排列模式。[这里是一个对新手很有要的排列模式概览]()

```js
var schema = new Schema({
  name: String
}, { collation: { locale: 'en_US', strength: 1 } });

var MyModel = db.model('MyModel', schema);

MyModel.create([{ name: 'val' }, { name: 'Val' }]).
  then(function() {
    return MyModel.find({ name: 'val' });
  }).
  then(function(docs) {
    // docs对象将会同时包含两个文档，因为`strength: 1`参数意味着 MongoDB将会在匹配中忽略大小写
  });
```

















