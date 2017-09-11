# Note


## 01 - Getting Setup

1. 安装 [node.js](https://nodejs.org/en/)，为了使用`async/await`，版本需要大于 7.6

   ```command
   node -v
   ```

2. 下载 [starter file](https://github.com/wesbos/Learn-Node)，进入目录，安装 packages

   ```command
   npm install
   ```

   *tips 如果安装出错，可能是某一个包更新了且旧版本无法使用，比如`node-sass`，在`package.json`中修改为最新的版本号，再安装*

3. 安装模板语言插件（使用的是pug，VS Code自带）


## 02 - Setting up Mongo DB

1. 创建 [mlab](https://mlab.com) 账号 => `MongoDB Deployment` => `create new` => 选一个免费的 => 创建数据库

2. 把项目目录下的`variable.env.sample`改为`variable.env`。其中有一项`DATABASE=mongodb://user:pass@host.com:port/database`，用新数据库的`URI`替换掉默认的值

3. 为数据库创建用户，将用户名和密码替换到以上的`URI`中

4. 用 [MongoDB Compass](https://www.mongodb.com/products/compass) 连接数据库。复制`URI`，`MongoDB Compass`会自动检测到配置，直接连接就可以了

5. 如果需要离线数据库，下载 [MongoDB](https://www.mongodb.com/)。
   + 在C盘根目录创建`data\db\`。
   + 命令行进入安装目录，如`C:\Program Files\MongoDB\Server\3.4\bin`，运行`mongod`。（如果无效就双击`mongod.exe`）
   + 打开`MongoDB Compass`，根据默认的值直接连接就可以了

6. 参考[Connection String URI Format](https://docs.mongodb.com/manual/reference/connection-string/)

```
DATABASE=mongodb://localhost:27017/test
```


## 03 - Starter Files and Environmental Variables

1. 查看`app.js`的结构，引入了哪些包，之后会不断地进入这里搞清楚它们到底做了什么

2. 查看`start.js`

```js
// start.js
// 1. 验证 node.js 版本
// 2. 导入环境变量
// 3. 连接数据库
// 4. 开始
```

3. 🏁 `npm start`，然后打开浏览器`localhost:7777`，页面上显示 Hey! It works! 代表成功

4. 有关部分包和命令的介绍看视频解释的比较清楚


## 04 - Core Concept - Routing

1. 进入`routes/index.js`，以下是`router`的使用方法

```js
// routes/index.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.send('Hey! It works!');
});

module.exports = router;

//app.js
const routes = require('./routes/index');
...
app.use('/', routes);
...
```

2. 类似的可以编写自己的路由，如下。然后浏览器打开`localhost:7777/test`查看效果

```js
//routes/index.js
...
router.get('/test', (req, res) => {
  res.send('Hey! It is just a test!');
});
...
```

3. `res.send()`和`res.json()`

```js
//routes/index.js
...
router.get('/test', (req, res) => {
  console.log('It is an apple.') // 会显示在 terminal 中
  const man = { name: 'wa', age: '23', cool: true }; // 定义一个对象
  // res.send('Hey! What is this?');
  res.json(man); // 以 JSON 的形式返回该对象，注意只能发送一次数据
});
...
```

4. 使用`req.query`获得 URL 参数，假设 URL 为`localhost:7777/test/?name=wa&age=23&cool=true`

```js
//routes/index.js
...
router.get('/test', (req, res) => {
  res.send(req.query);
  // res.send(req.query.name); // 或具体的某一项
});
...
```

5. `req.params`，假设 URL 为`localhost:7777/test/wispamulet`

```js
...
router.get('/test/:name', (req, res) => {
  res.send(req.params.name); // wispamulet
  // or
  const reverse = [...req.params.name].reverse().join('');
  res.send(reverse); // telumapsiw
});
...
```

*tips*

+ *Request has all the information, response has all the method for sending data back*
+ *Request.query to get the query parameters, request.body will be using for posted parameters, request.params in order to access the things that are in the URL*
+ *Check [Express API](https://expressjs.com/en/4x/api.html) for more*


## 05 - Core Concept - Templating

1. 设置模板语言

```js
// app.js
...
app.set('views', path.join(__dirname, 'views')); // 路径
app.set('view engine', 'pug'); // 使用 pug 作为模板
...
```

2. 新建`views/hello.pug`文件，一些基础的语法

```jade
//- hello.pug
.wrapper
  p.hello Hello!
  span#yo Yo!
  img.dog(src="dog.jpg" alt="Dog")

h2
  | Hello!
  em How are you?

//- hello.html
<div class="wrapper">
  <p class="hello">Hello!</p>
  <span id="yo">Yo!</span>
  <img class="dog" src="dog.jpg" alt="Dog">
</div>

<h2>Hello!<em>How are you?</em></h2>
```

3. 使用`res.render()`在`routes/index.js`中渲染该文件

```js
// routes/index.js
...
routes.get('/', (req, res) => {
  res.render('hello');
});
...
```

4. `res.render()`的第二个参数可以看作需要传入的变量

```js
// routes/index.js
...
routes.get('/', (req, res) => {
  res.render('hello', {
    name: 'wisp',
    age: 20
  });
});
...

// views/hello.pug
p Hello! I am #{name} // 直接使用传入的变量
img.dog(src="dog.jpg" alt=`Dog ${name}` ) // 在 attribute 中使用变量
```

5. 和上一节结合一下

```js
// routes/index.js
...
router.get('/', (req, res) => {
  res.render('hello', {
    name: req.query.name
  });
});
...

// views/hello.pug
p I am #{name}

// 打开浏览器 localhost:7777/?name=wispamulet
// 页面会显示 I am wispamulet
```

6. pug 中还可以写 JavaScript

```js
// routes/index.js
...同上

// views/hello.pug
- const NAME = name.toUpperCase();
p I am #{NAME}
// or
p I am #{name.toUpperCase()}
```

7. 可以在 pug 中引入另一个 pug 文件，使 html 中的某些部分可以重复使用。如果`block`中原本有内容，新的会把旧的覆盖

```js
// views/layout.pug
...
block header
  ...

.content
  block content
...

// views/hello.pug
extends layout

block content
  p Hey!
```

8. 🏁 现在运行`npm start`，页面上会显示出已经定义好样式的头部，`content`中的内容为普通的文本 Hey!


## 06 - Core Concept - Template Helpers

1. 上面的页面中，可以看到页面的名字显示为`undefined`，查看`layout.pug`，发现有一个变量没有值。因此在`router`中渲染页面时传入这个变量

```js
// views/layout.pug
...
title= `${title} | ${h.siteName}`
...

// routes/index.js
router.get('/', (req, res) => {
  res.render('hello', { title: 'I love food' });
});
```

2. 除此之外，标题中的`${h.siteName}`定义在`helper.js`中，查看`helper.js`还定义了什么

```js
// helper.js
...
exports.siteName = `Now That's Delicious!`;
...

// app.js
const helpers = require('./helpers');

...
app.use((req, res, next) => {
  res.locals.h = helpers; // on every single request, put the information in the locals
  ...
  next();
});
...
```

3. [moment.js](https://momentjs.com/)

```js
// helper.js
exports.moment = require('moment');

// app.js
如上

// hello.pug
extends layout

block content
  p it's #{h.moment().format()}
```

4. 与 environmental variables (环境变量) 不同的是，环境变量是敏感的，不应该出现在公布的代码中。这里的变量只是为了方便存储在这里。


## 07 - Core Concept - Controllers and the MVC Pattern

1. 理解MVC模式

*tips*

+ *All the stuff that interact with fetching data from database is the model layer*
+ *The templates would be the view layer*
+ *Controller gets the data and puts it into a template*

2. 创建`controller/storeController.js`，`routes/index.js`中的逻辑处理部分可以视为 controller

```js
// controller/storeController.js
exports.homePage = (req, res) => {
  res.render('index');
};

// routes/index.js
...
const storeController = require('../controllers/storeController');

router.get('/', storeController.homePage);
...
```


## 08 - Core Concept - Middleware and Error Handling

1. 理解 Middleware

   如图
   ![middleware](./middleware.png "Middleware")

   + 如果用户发出一个 post 请求，上传一张图片，可能需要 resize 图片，重命名图片等操作之后再存储图片。也就是说，发出 request 之后，返回 response 之前可能会有很多其它的事情要做
   + 可以把处理操作的代码写在具体的`res.render()`之前，但为了重复利用代码，可以写在一个 middleware 中，然后在 router 中重复使用

2. 举例

```js
// controllers/storeController.js
exports.middleware = (req, res, next) => {
  req.name = 'wisp';
  next();
};

exports.homePage = (req, res) => {
  res.render('hello', { title: req.name });
};

// routes/index.js
...
const storeController = require('../controllers/storeController');

router.get('/', storeController.middleware, storeController.homePage);
...
```

3. 以上的例子是一个 routes-specific middleware，`Express`中还有 global middleware，发出的每一个请求会经过这些 middleware 之后才到达 routes

```js
// app.js
...
app.use(...); // using some global middlewares

app.use(...);

app.use(...);

app.use('/', routes);
...
```

4. 在 routes 之后，还需要一些 error handlers，比如说 404 page

```js
// app.js
...
app.use(...);

app.use('/', routes);

app.use(errorHandlers.notFound);
...

// handlers/errorHandlers.js
...
exports.notFound = (req, res, next) => {
  ...
};
```


## 09 - Creating our Store Model

1. 使用 MongoDB 存储数据之前需要定义 schema，创建`models/Store.js`

```js
// model/Store.js
const mongoose = require('mongoose');
mongoose.Promise = global.Promise; // tell mongoDB to use Promise
const slug = require('slugs');

const storeSchema = new mongoose.Schema({
  ...
});

module.exports = mongoose.model('Store', storeSchema);
```

2. 在`start.js`中引入 models

```js
// start.js
// import all of our models
require('./models/Store');
```

3. 在存储之前还要给 slug 赋值

```js
// model/Store.js
...
storeSchema.pre('save', function(next) { // 为了使用 this，使用普通函数的写法
  if (!this.isModified('name')) {
    next(); // skip it
    return; // return
  }
  this.slug = slug(this.name);
  next();
});
...
```


## 10 - Saving Stores and using Mixins

1. 添加`/add`页面

```js
// routes/index.js 先添加 router
router.get('/add', storeController.addStore);

// controllers/storeController.js
exports.addStore = (req, res) => {
  res.send('It works!'); // 页面显示 It works! 表示成功

  res.render('editStore', { title: 'Add Store' }); // 使用模板
};

// views/editStore.pug 然后创建模板
extends layout

block content
  p It works!
```

2. 修改`views/editStore.pug`模板，重复的代码块（比如这里的表单）可以写成 mixin 直接引入，创建`views/mixins/_storeForm.pug`

```jade
//- views/mixins/_storeForm.pug
mixin storeForm(store = {})
  p It works! #{store.name}

//- views/editStore.pug
extends layout

include mixins/_storeForm

block content
  .inner
    h2= title
    +storeForm({ name: 'wisp' })
```

3. 修改`views/mixins/_storeForm.pug`

   1. `label`可以看作`input`的标识，它的`for`属性和对应的`input`属性的`id`属性应相同。比如这里`label`的`for`属性的值和`checkbox`的`id`属性的值相同，只要点击`label`区域就会选择相应的`checkbox`
   2. 如果这里也设置`id="name"`，当点击`label`区域时，会选中输入框
   3. 遍历`choices`数组

```jade
//- views/mixins/_storeForm.pug
mixin storeForm(store = {})
  form(action="/add" method="POST" class="card")
    label(for="name") Name
    //- 2.
    input(type="text" name="name")
    label(for="description") Description
    textarea(type="text" name="description")
    - const choices = ['Wifi', 'Open Late', 'Family Friendly', 'Vegatarian', 'Licensed']
    ul.tags
      //- 3.
      each choice in choices
        .tag.tag__choice
          input(type="checkbox" id=choice value=choice name="tags")
          //- 1.
          label(for=choice) #{choice}
    input(type="submit" value="Save 🚀" class="button")
```

> [HTML <label> 标签的 for 属性](http://www.w3school.com.cn/tags/att_label_for.asp)

4. POST 和 GET

   + 当点击`submit`时，`form`的两个属性: `action`决定了 data 的目的地，`method`告诉浏览器如何发送 data
   + POST 发送的 data 不会显示在 URL 中，而 GET 会。修改`views/mixins/_storeForm.pug`表单中的`method="GET"`查看效果，输入的值都以 query 的形式出现在了 URL 中
   + 由于需要 POST 这个表单，而 router 中没有对应的处理方式，只会得到一个 404 page

```js
// routes/index.js
router.post('/add', storeController.createStore);

// controllers/storeController.js
exports.createStore = (req, res) => {
  res.json(req.body);
}
```

修改之后可以看到页面以 json 的格式返回了表单中输入的值


## 11 - Using Async Await

1. 获得了表单的数据当然不只是想直接显示在页面上，首先需要存在数据库中

```js
// controllers/storeController.js
const mongoose = require('mongoose');
const Store = mongoose.model('Store'); // model 已经在 start.js 中引入了，可以直接使用

exports.createStore = (req, res) => {
  const store = new Store(req.body);
  store.save(); // Save to database
  res.redirect('/'); // 存储数据之后重定向至首页
};
...
```

2. 数据在存储之前可能会有其它的操作，而且数据存储需要一定的时间。由于 Javascript 是单线程的，它不会等待数据是否存储成功就会直接重定向，也不管是否产生了错误

```js
// 在以前，一般使用回调函数，但是代码会变得很复杂
// 可以利用 Promise 链式地写代码
// 然而，使用 Async/Await 可以做得更好
// controllers/storeController.js

...
exports.createStore = async (req, res) => {
  const store = new Store(req.body);
  await store.save(); // 只有真的保存到数据库之后才执行下一步
  res.redirect('/');
}
```

3. 处理错误

```js
// 普通的做法
// controllers/storeController.js
...
exports.createStore = async (req, res) => {
  try {
    const store = new Store(req.body);
    await store.save();
    res.redirect('/');
  } catch(err) {
    ...
  }
}

// composition
// createStore 不变，使用另一个函数处理整个 createStore 函数
// handlers/errorHandler.js
exports.catchErrors = (fn) => {
  return function(req, res, next) {
    return fn(req, res, next).catch(next);
  };
};

// routes/index.js
...
const { catchErrors } = require ('../handlers/errorHandlers');
...
router.post('/add', catchErrors(storeController.createStore));
```

4. 🏁 重新提交一个表单，重定向到了首页，连接 MongoDB Compass，数据已经存储到了数据库中。再提交一次，刷新 MongoDB Compass，看到了添加的数据


## 12 - Flash Messages

1. 用户提交了表单之后，先弹出一些信息再 redirect，指引用户下一步的操作，这种信息被称为 flash

```js
// controllers/storeController.js
exports.createStore = async (req, res) => {
  ...
  req.flash('success', `Successfully create ${store.name}`);
  res.redirect('/'); // 再提交一次表单，弹出了 flash 信息，然后 redirect 到首页
};

// app.js
const flash = require('connect-flash');
...
app.use(flash());
```

2. 修改 homePage，查看有哪些 flash

```js
// controllers/storeController.js
exports.homePage = (req, res) => {
  req.flash('success', `SUCCESS`);
  req.flash('info', `INFO`);
  req.flash('warning', `WARNING`);
  req.flash('error', `ERROR`);
  res.render('layout'); // 由于这里只有一次 request，再刷新一次页面才会看到效果
};
```

3. 页面上出现了 4 个不同的 flash 框，分别有不同的样式

```js
// app.js
app.use((req, res, next) => {
  ...
  res.locals.flashes = req.flash(); // pass variables
  ...
  next();
});
```

```jade
//- views/layout.pug
block messages
  if locals.flashes //- 如果有 flashes 这个变量，才执行以下代码
  pre= h.dump(locals) /* 查看由哪些 locals
                         也可以只查看 flashes，h.dump(locals.flashes)
                         dump() 定义在 helpers.js 中
                      */
    .inner
      .flash-messages
        - const categories = Object.keys(locals.flashes)
        each category in categories
          each message in flashes[category]
            //- 添加 class
            .flash(class=`flash--${category}`)
              //- != 为了在 message 中应用 html tags
              p.flash__text!= message
              button.flash__remove(onClick="this.parentElement.remove()") &times;
```

4. 总结一下

*tips*

+ *Do flashing in `controllers`*
+ *On the next request, it will put them in `locals` (in `app.js`)*
+ *`locals` are all the variables that are available in templates*

5. 删除不必要的代码，修改如下

```js
// controllers/storeController.js
...
exports.createStore = async (req, res) => {
  const store = new Store(req.body);
  await store.save();
  req.flash('success', `Successfully create ${store.name}, wanna leave a review?`);
  res.redirect(`/store/${store.slug}`); // slug 是在数据库保存之前生成的，并不在 req.body 中，因此不存在 store.slug 这个变量
};

// 修改如下
exports.createStore = async (req, res) => {
  const store = await (new Store(req.body)).save();
  ...
  res.redirect(`/store/${store.slug}`);
};
```

6. 🏁 现在打开`/add`页面，填好之后提交表单，页面上跳出了 flash message，并且 redirect 到了对应的商店页面（比如商店名为`Nice Bread`，URL跳至`/store/nice-bread`）。当然，由于现在还没有创建这个页面，只会显示 404 page


## 13 - Querying our Database for Stores

1. 点击 header 中的图标和 STORES 项时都需要跳转到 Store page

```js
// routes/index.js
router.get('/', storeController.getStores);
router.get('/stores', storeController.getStores);

// controllers/storeController.js
exports.getStores = (req, res) => {
  res.render('stores', { title: 'Stores' });
}
```

```jade
//- views/stores.pug
extends layout

block content
  .inner
    h2= title
```

2. 为了将 stores 的信息展示在页面上，首先要在数据库查找。这时可以在 terminal 中看到存储过的 stores 信息

```js
// controllers/storeController.js
exports.getStores = async (req, res) => {
  const stores = await Store.find();
  console.log(stores);
  res.render('stores', { title: 'Stores' });
}
```

3. 将 stores 作为变量传给`views/stores.pug`

```js
// controllers/storeController.js
exports.getStores = async (req, res) => {
  const stores = await Store.find();
  res.render('stores', { title: 'Stores', stores: stores });
  // or
  res.render('stores', { title: 'Stores', stores });
}
```

```jade
//- views/stores.pug
extends layout

include mixins/_storeCard

block content
  .inner
    h2= title
    //- 先预览一下 stores 包含哪些信息
    pre= h.dump(stores)
    .stores
      each store in stores
        h2= store.name
```

4. 修改`views/stores.pug`的样式

```jade
//- views/stores.pug
extends layout

include mixins/_storeCard

block content
  .inner
    h2= title
    .stores
      each store in stores
        +storeCard(store)

//- views/mixins/_storeCard.pug
mixin storeCard(store = {})
  .store
    .store__hero
      img(src=`/uploads/${store.photo || 'store.png'}`)
      h2.title
        a(href=`/store/${store.slug}`) #{store.name}
    .store__details
      //- 以免描述过长
      p= store.description.split(' ').slice(0, 25).join(' ')
```


## 14 - Creating an Editing Flow for Stores

1. 为 store card 添加编辑键

```jade
//- views/mixins/_storeCard.pug
mixin storeCard(store = {})
  .store
    .store__hero
      .store__actions
        .store__action.store__action--edit
          //- _id 是 MongoDB 自动生成的 ID
          a(href=`/stores/${store._id}/edit`)
            != h.icon('pencil')
      img(src=`/uploads/${store.photo || 'store.png'}`)
//- ...
```

```js
// 模板中使用的 h.icon();

// helper.js
exports.icon = (name) => fs.readFileSync(`./public/images/icons/${name}.svg`);

// app.js
...
const helpers = require('./helpers');
...
app.use((req, res, next) => {
  res.locals.h = helpers;
  ...
  next();
});
```

2. 有了编辑键，还需要在路由中添加对应的页面

```js
// routes/index.js
...
router.get('/stores/:id/edit', catchErrors(storeController.editStore));

// controllers/storeController.js
...
exports.editStore = async (req, res) => {
  // 1.
  res.json(req.params); // 先获得 URL 中的 params，也就是 routes 中的 :id 部分
  // 2.
  const store = await Store.findOne({ _id: req.params.id }); // 找到 id 对应的 store
  res.json(store);
  // 3.
  const store = await Store.findOne({ _id: req.params.id });
  res.render('editStore', { title: `Edit ${store.name}`, store: store }); // 渲染页面
}
```

3. `editStore.pug`是之前创建好的模板，创建 store 时表单中的值为空，编辑 store 时表单中的值应该是数据库中记录的值

```jade
//- views/editStore.pug
//- ...
block content
  .inner
    //- 预览 store 的信息
    pre= h.dump(store)
    h2= title
    //- 把 store 传给 _storeForm.pug
    +storeForm(store)

//- views/mixins/_storeForm.pug
//- ...
form(action=`/add/${store._id || ''}` method="POST" class="card")
  //- ...
  input(type="text" name="name" value=store.name)
  //- ...
  textarea(type="text" name="description")= store.description
  //- ...
  - const choices = ['Wifi', 'Open Late', 'Family Friendly', 'Vegatarian', 'Licensed']
  - const tags = store.tags || []
    //- ...
    input(type="checkbox" id=choice value=choice name="tags" checked=(tags.includes(choice)))
```

4. 由于上面修改了`form(action='/add')`，再次尝试添加 store，添加成功。然而编辑 store 时却没有保存。修改 routes

```js
// routes/index.js
router.post('/add/:id', catchErrors(storeController.updateStore));

// controllers/storeController.js
exports.updateStore = async (req, res) => {
  // 1. Find and update the store
  const store = await Store.findOneAndUpdate({ _id: req.params.id }, req.body, {
    new: true, // return the new store instead of the old one
    runValidators: true
  }).exec();
  req.flash('success', `Successfully updated <strong>${store.name}</strong>. <a href="/store/${store.slug}">View Store 👉</a>`);
  // 2. Redirect them the store and tell them it worked
  res.redirect(`/stores/${store._id}/edit`);
};
```


## 15 - Saving Lat and Lng for each store

1. 为了存入更多的数据，先修改`Store.js`结构

```js
// models/Store.js
...
const storeSchema = new mongoose.Schema({
  ...
  created: {
    type: Date,
    default: Date.now
  },
  location: {
    type: {
      type: String,
      default: 'Point'
    },
    coordinates: [{
      type: Number,
      required: 'You must supply coordinates!'
    }],
    address: {
      type: String,
      required: 'You must supply address!'
    }
  }
});
...
```

2. 然后修改`views/mixins/_storeForm.pug`

```jade
//- views/mixins/_storeForm.pug
//- ...
textarea(type="text" name="description")= store.description
//- address, lng and lat
label(for="address") Address
  //- location[address] 会被 app.js 中的 bodyParser.urlencoded() 解析（？）为 location.address
  //- If store.location exists, it will go and return the second thing. If not, return false
input(type="text" id="address" name="location[address]" value=(store.location && store.location.address))
label(for="lng") Address lng
//- 可以添加 required 属性做客户端方面的验证
input(type="text" id="lng" name="location[coordinates][0]" value=(store.location && store.location.coordinates[0]) required)
label(for="lat") Address lat
input(type="text" id="lat" name="location[coordinates][1]"  value=(store.location && store.location.coordinates[1]) required)
//- ...
```

3. 尽管有客户端方面的验证，数据库的 schema 中也添加了 required 属性

```js
// 比如说创建 store 时，表单中如果没有填入 address 的值
// 页面会会弹出 flash 信息，显示 required 的值，这里为 You must supply address!

// models/Store.js
...
const storeSchema = new mongoose.Schema({
  ...
  address: {
    type: String,
    required: 'You must supply address!'
  }
});
...
```

```js
// 以上情况是如何实现的，如下
// 可以看到，controller 中并没有额外的验证代码
// 由于使用了 async/await，只有验证过数据是否符合 schema 后才会保存

// controllers/storeController.js
exports.createStore = async (req, res) => {
  const store = await (new Store(req.body)).save();
  ...
};
```

```js
// 产生的 error 会交给 catchErrors()

// routes/index.js
router.post('/add', catchErrors(storeController.createStore));

// handlers/errorHandlers.js
exports.catchErrors = (fn) => {
  return function(req, res, next) {
    return fn(req, res, next).catch(next);
  };
};
```

```js
// next 会来到 app.use('/', routes); 之后的内容
// flashValidationErrors() 会处理之前的 error

// app.js
...
app.use('/', routes);
app.use(errorHandlers.flashValidationErrors);

// handlers/errorHandlers.js
exports.flashValidationErrors = (err, req, res, next) => {
  // if there are no errors to show for flashes, skip it
  if (!err.errors) return next(err);
  // validation errors look like
  const errorKeys = Object.keys(err.errors);
  errorKeys.forEach(key => req.flash('error', err.errors[key].message));
  res.redirect('back');
};
```


## 16 - Geocoding Data with Google Maps

1. 首先要创建自己的 google api key。

   进入[cloud resource manager](https://console.developers.google.com/cloud-resource-manager)创建项目，然后进入 API manager -> Credentials -> Create credentials -> API key，将申请的 key 替换到`variables.env`

2. 打开`public/javascripts/delicious-app.js`，这是 webpack 打包的 entry point。`public/javascripts/modules/`文件夹中存放的是客户端的 javascript 文件。创建`public/javascripts/modules/autocomplete.js`

```js
// public/javascripts/modules/autocomplete.js
function autocomplete(input, latInput, lngInput) {
  // console.log(input, latInput, lngInput);
  if (!input) return; // skip this fn from running if there is no input on the page
  const dropdown = new google.maps.places.Autocomplete(input); // 使用 google api
}

export default autocomplete;

// public/javascripts/delicious-app.js
...
import { $, $$ } from './modules/bling';
import autocomplete from './modules/autocomplete';

autocomplete( $('#address'), $('#lat'), $('#lng') ); // $('') 等同于 document.querySelector('')，来自 bling.js
                                                     // #address 等是 views/mixins/_storeForm.pug 中的 ID 名
                                                     // 这里相当于获得了 #address #lat #lng 三个 DOM 元素
```

3. 刷新页面，地址的输入框已经改变，输入内容会自动联想，但地址的经度和纬度并没有随地址自动填充，修改如下

```js
// public/javascripts/modules/autocomplete.js
function autocomplete(input, latInput, lngInput) {
  ...
  const dropdown = new google.maps.places.Autocomplete(input); // 使用 google api

  dropdown.addListener('place_changed', () => {
    const place = dropdown.getPlace();
    console.log(place); // 返回关于该地址的一个 object，在 chrome dev tools 中查看
    // 自动改变其余两项的值
    latInput.value = place.geometry.location.lat();
    lngInput.value = place.geometry.location.lng();
  });

  // if someone htis enter on the address field, don't submit the form
  input.on('keydown', (e) => {
    if (e.keyCode === 13) e.preventDefault();
  });
}
...
```


## 17 - Quick Data Visualization Tip

1. 由于之前修改过 schema，有些数据缺少某些属性，删除所有的数据，重新添加商店的信息

2. 在 MongoDB Compass 中会根据地址把商店显示在地图上，修改某一项的地址之后，虽然依然可以显示对应的地点，但类型没有存储为 Point，修改如下

```js
// controllers/storeController.js
exports.updateStore = async (req, res) => {
  // Set the location data to be a point
  req.body.location.type = 'Point';
  ...
}；
```


## 18 - Uploading and Resizing Images with Middleware

1. 为了向服务器发送文件，为 form 添加属性`enctype="multipart/form-data"`

```jade
//- views/mixins/_storeForm.pug
form(action=`/add/${store._id || ''}` method="POST" class="card" enctype="multipart/form-data")

//- Image Upload
label(for="photo") Photo
  input(type="file" name="photo" id="photo" accept="image/gif, image/png, image/jpeg")
```

2. 为了上传并调整图片的大小，需要引入一些外部的包。`Multer`用来上传文件。

```js
// controllers/storeController.js
const multer = require('multer');

const multerOptions = {
  storage: multer.memoryStorage(),
  fileFilter(req, file, next) {
    const isPhoto = file.mimetype.startsWith('image/');
    if (isPhoto) {
      next(null, true);
    } else {
      next({ message: 'The filetype isn\'t allowed!' }, false);
    }
  }
};
...

exports.upload = multer(multerOptions).single('photo');
```

3. `GIMP`用来 resize 图片的大小。'uuid'用来重命名图片。

```js
// controllers/storeController.js
const jimp = require('jimp');
const uuid = require('uuid');

exports.resize = async (req, res, next) => {
  // check if there is no new file to resize
  if (!req.file) {
    next(); // skip to the next middleware
    return;
  }
  // console.log(req.file);
  const extension = req.file.mimetype.split('/')[1];
  req.body.photo = `${uuid.v4()}.${extension}`;
  // now we resize
  const photo = await jimp.read(req.file.buffer);
  await photo.resize(800, jimp.AUTO);
  await photo.write(`./public/uploads/${req.body.photo}`);
  // once we have written the photo to the filesystem, keep going!
  next();
}
```

4. 为保存的图片在数据库中添加 Schema，这样才会被保存并显示。

```js
const storeSchema = new mongoose.Schema({
  ...
  photo: String
})
```

5. 修改 routes

```js
// routes/index.js
router.post('/add',
  storeController.upload,
  catchErrors(storeController.resize),
  catchErrors(storeController.createStore)
);

router.post('/add/:id',
  storeController.upload,
  catchErrors(storeController.resize),
  catchErrors(storeController.updateStore)
);
```


## 19 - Routing and Templating Single Stores

1. 为每个商店创建具体的页面

```js
// routes/index.js
router.get('/store/:slug', catchErrors(storeController.getStoreBySlug));

// controllers/storeController.js
exports.getStoreBySlug = async (req, res) => {
  // res.send('It works!');
  const store = await Store.findOne({ slug: req.params.slug });
  if (!store) return next();
  res.render('store', { store, title: store.name });
};
```

2. 修改页面样式

```jade
//- views/store.pug
extends layout

block content
  .single
    .single__hero
      img.single__image(src=`/uploads/${store.photo || 'store.png'}`)
      h2.title.title--single
        a(href=`/store/${store.slug}`) #{store.name}
  .single__details.inner
    img.single__map(src=h.staticMap(store.location.coordinates))
    p.single__location= store.location.address
    p= store.description

    if store.tags
      ul.tags
        each tag in store.tags
          li.tag
            a.tag__link(href=`/tags/${tag}`)
              span.tag__text #{tag}
```


## 20 - Using Pre-Save hooks to make Unique Slugs

1. 为名字相同的 Store 创建不同的 slug

```js
// models/Store.js
storeSchema.pre('save', async function(next) {
  ...
  this.slug = slug(this.name);
  // find other stores that have a slug of wes, wes-1, wes-2
  const slugRegEx = new RegExp(`^(${this.slug})((-[0-9]*$)?)$`, 'i');
  const storesWithSlug = await this.constructor.find({ slug: slugRegEx });
  if (storesWithSlug.length) {
    this.slug = `${this.slug}-${storesWithSlug.length + 1}`;
  }
  ...
})
```


## 21 - Custom MongoDB Aggregations

1. 创建`/tags`页面

```js
// routes/index.js
router.get('/tags', catchErrors(storeController.getStoresByTag));
router.get('/tags/:tag', catchErrors(storeController.getStoresByTag));

// controllers/storeController.js
exports.getStoresByTag = async (req, res) => {
  // res.send('It works!');
  const tags = await Store.getTagsList();
  res.render('tag', { tags, title: 'Tags' });
};

// models/Store.js
...
storeSchema.statics.getTagsList = function () {
  return this.aggregate([
    { $unwind: '$tags' },
    { $group: { _id: '$tags', count: { $sum: 1 } } },
    { $sort: { count: -1 } }
  ]);
}
```

2. 创建`tag.pug`

```jade
//- views/tag.pug
extends layout

block content
  .inner
    h2 #{tag || 'Tags'}
    ul.tags
      each t in tags
        li.tag
          a.tag__link(href=`/tags/${t._id}` class=(t._id === tag ? 'tag__link--active' : ''))
            span.tag__text= t._id
            span.tag__count= t.count
```


## 22 - Multiple Query Promises with Async-Await

1. 查询 tags 的同时，还需要查询 tags 对应的商店。为了让它们同时进行，修改`getStoresByTag()`

```js
// controllers/storeController.js
exports.getStoresByTag = async (req, res) => {
  const tag = req.params.tag;
  const tagsPromise = Store.getTagsList();
  const storesPromise = Store.find({ tags: tag });
  const result = await Promise.all([tagsPromise, storesPromise]);
  res.json(result);
  // res.render('tag', { tags, title: 'Tags', tag });
};

// 解构之后
exports.getStoresByTag = async (req, res) => {
  ...
  const [tags, stores] = await Promise.all([tagsPromise, storesPromise]);
  res.render('tag', { tag, title: 'Tags', tags, stores });
}
```

2. 修改`tag.pug`以显示 stores

```jade
//- views/tag.pug
extends layout

include mixins/_storeCard

//-...

.stores
  each store in stores
    +storeCard(store)
```

3. 直接进入`/tags`页面时，由于没有选择某一个 tag，页面上并不显示任何商店，修改如下

```js
// controllers/storeController.js
exports.getStoresByTag = async (req, res) => {
  ...
  const tagQuery = tag || { $exists: true };
  ...
  const storesPromise = Store.find({ tags: tagQuery });
  ...
};
```


## 23 - Creating User Accounts

1. 添加`/login`页面

```js
// routes/index.js
const userController = require('../controllers/userController');
...
router.get('/login'. userController.loginForm);

// controllers/userController.js
const mongoose = require('mongoose');

exports.loginForm = (req, res) => {
  res.render('login', { title: 'Login' });
};
```

```jade
//- views/login.pug
extends layout

include mixins/_loginForm

block content
  .inner
    +loginForm()

//- views/mixins/_loginForm.pug
mixin loginForm()
  form.form(action="/login" method="POST")
    h2 Login
    label(for="email") Email
    input(type="email" name="email")
    label(for="password") Password
    input(type="password" name="password")
    input.button(type="submit" value="Log In →")
```

2. 为了保存数据，创建`models/User.js`

```js
const mongoose = require('mongoose');
mongoose.Promise = global.Promise;
const md5 = require('md5');
const validator = require('validator');
const mongodbErrorHandler = require('mongoose-mongodb-errors');
const passportLocalMongoose = require('password-local-mongoose');

const userSchema = new mongoose.Schema({
  email: {
    type: String,
    unique: true,
    lowercase: true,
    trim: true,
    validate: [validator.isEmail, 'Invalid Email Address'],
    required: 'Please supply an email address!'
  },
  name: {
    type: String,
    required: 'Please supply a name!',
    trim: true
  }
});

userSchema.plugin(passportLocalMongoose, { usernameField: 'email' });
userSchema.plugin(mongodbErrorHandler);

module.exports = mongoose.model('User', userSchema);
```

3. 添加`/register`页面

```js
// routes/index.js
router.get('/register', userController.registerForm);

// controllers/userController.js
exports.registerForm = (req, res) => {
  res.render('register', { title: 'Register' });
}
```

```jade
//- views/register.pug
extends layout

block content
  .inner
    form.form(action="/register" method="POST")
      h2 Register
      label(for="name") Name
      input(type="text" name="name" required)
      label(for="email") Email
      input(type="text" name="email" required)
      label(for="password") Password
      input(type="text" name="password")
      label(for="password-confirm") Confirm Password
      input(type="text" name="password-confirm")
      input.button(type="submit" value="Register →")
```

4. 添加`/register`页面提交表单

```js
// routes/index.js
router.post('/register', userController.validateRegister);

// controllers/userController.js
exports.validateRegister = (req, res, next) => {
  req.sanitizeBody('name');
  req.checkBody('name', 'You must supply a name!').notEmpty();
  req.checkBody('email', 'That email is not valid!').isEmail();
  req.sanitizeBody('email').normalizeEmail({
    remove_dots: false,
    remove_extension: false,
    gmail_remove_subaddress: false
  });
  req.checkBody('password', 'Password cannot be blank!').notEmpty();
  req.checkBody('password-confirm', 'Confirm password cannot be blank').notEmpty();
  req.checkBody('password-confirm', 'Oops! Your password do not match.').equals('req.body.password');

  const errors = req.validationErrors();
  if (errors) {
    req.flash('error', errors.map(err => err.msg));
    res.render('register', { title: 'Register', body: req.body, flashes: req.flash() });
    return;  // stop the function from running
  }
  next();  // there were no errors
};
```


## 24 - Saving Registered Users to the Database

1. 验证注册信息之后要保存信息，修改`routes`

```js
// routes/index.js
router.post('/register',
  userController.validateRegister,
  catchErrors(userController.register)
);

// controllers/UserController.js
const User = mongoose.model('User');
const promisify = require('es6-promisify');
...
exports.register = async (req, res, next) => {
  const user = new User({ email: req.body.email, name: req.body.name });
  // User.register(user, req.body.password, function (err, user) {

  // });
  const register = promisify(User.register, User);
  await register(user, req.body.password);
  res.send('it works!');
  next(); // pass to authController.login
}

// start.js
require('./models/User');
```

2. 创建`controllers/authController.js`

```js
// controllers/authController.js
const passport = require('passport');

exports.login = passport.authenticate('local', {
  failureRedirect: '/login',
  failureFlash: 'Failed Login!',
  successRedirect: '/',
  successFlash: 'You are now logged in!'
});

// routes/index.js
const authController = require('../controllers//authController');
...
router.post('/register',
  userController.validateRegister,
  catchErrors(userController.register),
  authController.login
);
```

3. 创建`passport.js`

```js
// handler/passport.js
const passport = require('passport');
const mongoose = require('mongoose');
const User = mongoose.model('User');

passport.use(User.createStrategy());

passport.serializeUser(User.serializeUser());
passport.deserializeUser(USer.deserializeUser());

// app.js
...
require('./handlers/passport');
...
```


## 25 - Virtual Fields, Login-Logout middleware and Protecting Routes

1. 添加`/logout`

```js
// controllers/authController.js
exports.logout = (req, res) => {
  req.logout();
  req.flash('success', 'You are now logged out!');
  res.redirect('/');
}

// routes/index.js
router.get('/logout', authController.logout);
```

2. 添加 POST `/login`

```js
// routes/index.js
router.post('/login', authController.login);
```

3. 使用 Virtual Field 为用户添加头像

```jade
//- views/layout.pug
img.avatar(src=user.gravatar + 'd=retro')
```

```js
// models/User.js
userSchema.virtual('gravatar').get(function () {
  const hash = md5(this.email);
  return `https://gravatar.com/avatar/${hash}?s=200`;
});
```

4. 只有登陆的用户才能添加 Store

```js
// controllers/authController.js
exports.isLoggedIn = (req, res, next) => {
  // first check if the user is authenticated
  if (req.isAuthenticated()) {
    next(); // carry on! They are logged in!
    return;
  }
  req.flash('error', 'Oops! You must be logged in to do that!');
  res.redirect('/login');
};

// routes/index.js
router.get('/add',
  authController.isLoggedIn,
  storeController.addStore
);
```


## 26 - Creating a User Account Edit Screen

1. 创建`/account`页面

```js
// routes/index.js
router.get('/account', userController.account);

// controllers/userController.js
exports.account = (req, res) => {
  res.render('account', { title: 'Edit Your Account' });
};
```

```jade
//- views/account.pug
extends layout

block content
  .inner
    h2= title
    form(action="/account" method="POST")
      label(for="name") Name
      input(type="text" name="name" value=user.name)
      label(for="email") Email Address
      input(type="text" name="email" value=user.email)
      input.button(type="submit" value="Update My Account")
```

2. 添加 POST `/account`

```js
// routes/index.js
router.post('account', catchErrors(userController.updateAccount));

// controllers/userController.js
exports.updateAccount = async (req, res) => {
  const updates = {
    name: req.body.name,
    email: req.body.email
  };

  const user = await User.findOneAndUpdate(
    { _id: req.user._id },
    { $set: updates },
    { new: true, runValidators: true, context: 'query' }
  );

  req.flash('success', 'Updated the profile!');
  res.redirect('back');
};
```


## 27 - Password Reset Flow

1. 修改`login.pug`

```jade
//- views/login.pug
extends layout

include mixins/_loginForm
include mixins/_forgotForm

block content
  .inner
    +loginForm()
    +forgotForm()

//- views/mixins/_forgotForm.pug
mixin forgotForm()
  form.form(action="/account/forgot" method="POST")
    h2 I forgot my password!
    label(for="email") Email
    input(type="text" name="email")
    input.button(type="submit" value="send a RESET")
```

2. 添加 POST `/account/forgot`

```js
// models/User.js
const userSchema = new mongoose.Schema({
  ...
  resetPasswordToken: String,
  resetPasswordExpires: Date
})

// routes/index.js
router.post('/account/forgot', catchErrors(authController.forgot));

// controllers/authController.js
const crypto = require('crypto');
const mongoose = require('mongoose');
const User = mongoose.model('User');

exports.forgot = async (req, res) => {
  // 1. See if a user with that email exists
  const user = await User.findOne({ email: req.body.email });
  if (!user) {
    req.flash('error', 'No account with that email exists!');
    return res.redirect('/login');
  }
  // 2. Set reset tokens and expiry on their account
  user.resetPasswordToken = crypto.randomBytes(20).toString('hex');
  user.resetPasswordExpires = Date.now() + 3600000; // 1 hour from now
  await user.save();
  // 3. Send them an email with the token
  const resetURL = `http://${req.headers.host}/account/reset/${user.resetPasswordToken}`;
  req.flash('success', `You have been emailed a password reset link. ${resetURL}`);
  // 4. redirect to login page
  res.redirect('/login');
};
```

3. 添加 resetURL 页面

```js
// routes/index.js
router.get('/account/reset/:token', catchErrors(authController.reset));

// controllers/authController.js
exports.reset = async (req, res) => {
  // res.json(req.params);
  const user = await User.findOne({
    resetPasswordToken: req.params.token,
    resetPasswordExpires: { $gt: Date.now() }
  });
  if (!user) {
    req.flash('error', 'Password reset token is invalid or has expired!');
    return res.redirect('/login');
  }
  res.render('reset', { title: 'Reset your password' });
};
```

```jade
//- views/reset.pug
extends layout

block content
  .inner
    form.form(method="POST")
      h2= title
      label(for="password") Password
      input(type="password" name="password")
      label(for="password-confirm") Confirm Password
      input(type="password" name="password-confirm")
      input.button(type="submit" value="Reset your password→")
```

4. 保存更新的密码

```js
// routes/index.js
router.post('/account/reset/:token',
  authController.confirmPasswords,
  catchErrors(authController.update)
);

// controllers/authController.js
exports.confirmPasswords = (req, res, next) => {
  if (req.body.password === req.body['password-confirm']) {
    next(); // keep it going
    return;
  }
  req.flash('error', 'Passwords do not match!');
  res.redirect('back');
};

exports.udpate = async (req, res) => {
  const user = await User.findOne({
    resetPasswordToken: req.params.token,
    resetPasswordExpires: { $gt: Date.now() }
  });
  if (!user) {
    req.flash('error', 'Password reset token is invalid or has expired!');
    return res.redirect('/login');
  }

  const setPassword = promisify(user.setPassword, user);
  await setPassword(req.body.password);
  user.resetPasswordToken = undefined;
  user.resetPasswordExpires = undefined;
  const updateUser = await user.save();
  await req.login(updateUser);
  req.flash('success', '😛 Nice! Your password has been reset! You are now logged in!');
  res.redirect('/');
}
```


## 28 - Sending email with Nodejs

1. [mailtrap](https://mailtrap.io)，首先修改`variables.env`中的参数`MAIL_USER``MAIL_POST`

```js
// handlers/mail.js
const nodemailer = require('nodemailer');
const pug = require('pug');
const juice = require('juice');
const htmlToText = require('html-to-text');
const promisify = require('es6-promisify');

const transport = nodemailer.createTransport({
  host: process.env.MAIL_HOST,
  port: process.env.MAIL_PORT,
  auth: {
    user: process.env.MAIL_USER,
    pass: process.env.MAIL_PASS
  }
});

transport.sendMail({
  from: 'Wes Bos <wesbos@gmail.com>',
  to: 'someone@example.com',
  subject: 'Just trying things out!',
  html: 'Hey <strong>LOVE</strong> you!',
  text: 'Hey **LOVE** you!'
});

// start.js
// TEMP send email
require('./handlers/mail');
```

2. 以上只是测试发出一封邮件，真实的情况是当用户修改密码时，发出邮件

```js
// handlers/mail.js
...
exports.send = async (options) => {
  const mailOptions = {
    from: 'Wes Bos <noreply@wesbos.com>',
    to: options.user.email,
    subject: options.subject,
    html: 'This will be filled in later.',
    text: 'This will also be filled in later.'
  };
  const sendMail = promisify(transport.sendMail, transport);
  return sendMail(mailOptions);
};

// controllers/authController.js
const mail = require('../handlers/mail');
...
exports.forgot = async (req, res) => {
  ...
  // 3.
  const resetURL = `http://${req.headers.host}/account/reset/${user.resetPasswordToken}`;
  await mail.send({
    user,
    subject: 'Password Reset',
  });
  req.flash('success', `You have been emailed a password reset link.`);
  ...
}
```

3. 使用 pug 渲染邮件

```js
// controllers/authController.js
exports.forgot = async (req, res) => {
  ...
  await mail.send({
    user,
    subject: 'Password Reset',
    resetURL,
    filename: 'password-reset'
  })
}

// handlers/mail.js
const generateHTML = (filename, options = {}) => {
  const html = pug.renderFile(`${__dirname}/../views/email/${filename}.pug`, options);
  const inlined = juice(html);
  return inlined;
};

exports.send = async (options) => {
  const html = generateHTML(options.filename, options);
  const text = htmlToText.fromString(html);

  const mailOptions = {
    ...
    html,
    text
  };
  ...
};
```


## 29 - Locking down our application with User Permissions

1. 为商店添加作者属性，用户只能修改属于自己的商店

```js
// models/Store.js
const storeSchema = new mongoose.Schema({
  ...
  author: {
    type: mongoose.Schema.ObjectId,
    ref: 'User',
    required: 'You must supply an author!'
  }
});

// controllers/storeController.js
exports.createStore = async (req, res) => {
  req.body.author = req.user._id;
  ...
}
```

2. 进入编辑页面前检查是否是作者

```js
// controllers/storeController.js
const confirmOwner = (store, user) => {
  if (!store.author.equals(user._id)) {
    throw Error('You must own a store in order to edit it!');
  }
};

exports.editStore = async (req, res) => {
  ...
  const store = await Store.findOne({ _id: req.params.id });
  // 2.
  confirmOwner(store, req.user);
  ...
};
```

3. 创建第二个用户，并创建新的商店。

```jade
//- views/mixins/_storeCard.pug
.store__actions
  if user && store.author.equals(user._id)
    //-...
```


## 30 - Loading Sample Data

1. 在 Terminal 中运行`npm run sample`，由于暂时还没有 Review 模块，需要先注释掉`load-sample-data.js`中关于 Review 的部分。


## 31 - JSON endpoints and creating MongoDB Indexes

1. 为 Store 创建 index，可以在 MongoDB Compass 中查看

```js
// models/Store.js
// Define the indexes
storeSchema.index({
  name: 'text',
  description: 'text'
});
```

2. 创建 search API，进入如`/api/search/?q=beer`查看结果

```js
// routes/index.js
router.get('/api/search', catchErrors(storeController.searchStore));

// controllers/storeController.js
exports.searchStore = async (req, res) => {
  const stores = await Store
    // first find stores that match
    .find({
      $text: { $search: req.query.q }
    }, {
      score: { $meta: 'textScore' }
    })
    // then sort them
    .sort({
      score: { $meta: 'textScore' }
    })
    // limit to only 5 results
    .limit(5);
    res.json(stores);
};
```


## 32 - Creating an Ajax Search Interface

1. 利用之前设计的 API 完成搜索功能

```js
// public/javascripts/modules/typeAhead.js
import axios from 'axios';

function searchResultsHTML(stores) {
  return stores.map(store => {
    return `
      <a href="/store/${store.slug}" class="search__result">
        <strong>${store.name}</strong>
      </a>
    `;
  }).join('');
}

function typeAhead(search) {
  if (!search) return;

  const searchInput = search.querySelector('input[name=search]');
  const searchResults = search.querySelector('.search__results');

  searchInput.on('input', function () {
    // if there is no value, quit it!
    if (!this.value) {
      searchResults.style.display = 'none';
      return; // stop!
    }

    // show the search results!
    searchResults.style.display = 'block';
    // searchResults.innerHTML = '';

    axios
      .get(`/api/search?q=${this.value}`)
      .then(res => {
        if (res.data.length) {
          searchResults.innerHTML = searchResultsHTML(res.data)
          return;
        }
        // tell them nothing come back
        searchResults.innerHTML = `<div class="search__result">No results for ${this.value} found!</div>`;
      })
      .catch(err => {
        console.error(err);
      });
  });
}

export default typeAhead;


// public/javascripts/delicious-app.js
...
import typeAhead from './modules/typeAhead';

typeAhead( $('.search') );
```

2. 搜索时，还可以用方向键选择项

```js
// public/javascripts/modules/typeAhead.js
...
function typeAhead(search) {
  ...
  // handle keyboard inputs
  searchInput.on('keyup', (e) => {
    // if they aren't pressing up, down or enter, who cares!
    if (![38, 40, 13].includes(e.keyCode)) {
      return; // skip it!
    }
    const activeClass = 'search__result--active';
    const current = search.querySelector(`.${activeClass}`);
    const items = search.querySelectorAll('.search__result');
    let next;
    if (e.keyCode === 40 && current) {
      next = current.nextElementSibling || items[0];
    } else if (e.keyCode === 40) {
      next = items[0];
    } else if (e.keyCode === 38 && current) {
      next = current.previousElementSibling || items[items.length - 1];
    } else if (e.keyCode === 38) {
      next = items[items.length - 1];
    } else if (e.keyCode === 13 && current.href) {
      window.location = current.href;
      return;
    }

    if (current) {
      current.classList.remove(activeClass);
    }
    next.classList.add(activeClass);
  });
}
...
```

3. 为了避免 XSS 攻击，使用 `dompurify`。或者直接禁止在商店名或描述中使用 html 语句 (optional)

```js
// public/javascripts/modules/typeAhead.js
import dompurify from 'dompurify';
...
searchResults.innerHTML = dompurify.sanitize(searchResultsHTML(res.data));
...
searchResults.innerHTML = dompurify.sanitize(`<div class="search__result">No results for ${this.value} found!</div>`);
...
```


## 33 - Creating a Geospatial Ajax Endpoint

1. 创建 map API

```js
// models/Store.js
storeSchema.index({ location: '2dsphere' });

// routes/index.js
router.get('/api/map/near', catchErrors(storeController.mapStores));

// controllers/storeController.js
exports.mapStores = async (req, res) => {
  const coordinates = [req.query.lng, req.query.lat].map(parseFloat);
  const q = {
    location: {
      $near: {
        $geometry: {
          type: 'Point',
          coordinates
        },
        $maxDistance: 10000 // 10km
      }
    }
  };

  const stores = await Store.find(q).select('slug name description location').limit(10);
  res.json(stores);
};
```


## 34 - Plotting Stores on a Custom Google Map

1. 为`/map`创建 routes

```js
// routes/index.js
router.get('/map', storeController.mapPage);

// controllers/storeController.js
exports.mapPage = (req, res) => {
  res.render('map', { title: 'Map' });
};
```

```jade
//- views/map.pug
extends layout

block content
  .inner
    h2= title
    .map
      .autocomplete
        input.autocomplete__input(type="text" placeholder="Search for Anything" name="geolocate")
      #map
        p Loading Map...

```

2. 创建`map.js`

```js
// public/javascripts/delicious-app.js
import makeMap from './modules/map';

makeMap( $('#map') );

// public/javascripts/modules/map.js
import axios from 'axios';
import { $ } from './bling';

const mapOptions = {
  center: { lat: 43.2, lng: -79.8 },
  zoom: 10
}

function loadPlaces(map, lat = 43.2, lng = -79.8) {
  axios.get(`/api/stores/near?lat=${lat}&lng=${lng}`)
    .then(res => {
      const places = res.data;
      if (!places.length) {
        alert('no places found!');
        return;
      }
      // create a bounds
      const bounds = new google.maps.LatLngBounds();
      const infoWindow = new google.maps.InfoWindow();

      const markers = places.map(place => {
        const [placeLng, placeLat] = place.location.coordinates;
        const position = { lat: placeLat, lng: placeLng };
        bounds.extend(position);
        const marker = new google.maps.Marker({ map, position });
        marker.place = place;
        return marker;
      });

      // when someone clicks on a marker, show the details of that place
      markers.forEach(marker => marker.addListener('click', function () {
        const html = `
          <div class="popup">
            <a href="/store/${this.place.slug}">
              <img src="/uploads/${this.place.photo || 'store.png'}" alt="${this.place.name}" />
              <p>${this.place.name} - ${this.place.location.address}</p>
            </a>
          </div>
        `;
        infoWindow.setContent(html);
        infoWindow.open(map, this);
      }));

      // then zoom the map to fit all the markers perfectly
      map.setCenter(bounds.getCenter());
      map.fitBounds(bounds);
    });
}

function makeMap(mapDiv) {
  if (!mapDiv) return;
  // make our map
  const map = new google.maps.Map(mapDiv, mapOptions);
  loadPlaces(map);

  const input = $('[name="geolocate"]');
  const autocomplete = new google.maps.places.Autocomplete(input);
  autocomplete.addListener('place_changed', () => {
    const place = autocomplete.getPlace();
    loadPlaces(map, place.geometry.location.lat(), place.geometry.location.lng());
  });
}

export default makeMap;
```


## 35 - Pushing User Data to our API

1. 为`User.js`添加`hearts` field

```js
const userSchema = new mongoose.Schema({
  ...
  hearts: [
    { type: mongoose.Schema.ObjectId,  ref: 'Store' }
  ]
});
```

2. 修改`views/mixins/_storeCard.pug`

```jade
//-...
.store__actions
  if user
    .store__action.store__action--heart
      form.heart(action=`/api/stores/${store._id}/heart` method="POST")
        button(type="submit" name="heart")
          != h.icon('heart')
```

3. 为`/api/stores/${store._id}/heart`创建 routes

```js
// routes/index.js
router.post('/api/stores/:id/heart', catchErrors(storeController.heartStore));

// controllers/storeController.js
exports.heartStore = async (req, res) => {
  const hearts = req.user.hearts.map(obj => obj.toString());
  const operator = hearts.includes(req.params.id) ? '$pull' : '$addToSet';
  const user = await User.findByIdAndUpdate(req.user._id,
    { [operator]: { hearts: req.params.id } },
    { new: true }
  );
  res.json(user);
};
```

4. 继续修改`views/mixins/_storeCard.pug`，为 heart 添加 class

```jade
form.heart(action=`/api/stores/${store._id}/heart` method="POST")
  - const heartStrings = user.hearts.map(obj => obj.toString())
  - const heartClass = heartStrings.includes(store._id.toString()) ? 'heart__button--hearted' : ''
  button.heart__button(type="submit" name="heart" class=heartClass)
```

5. 创建`heart.js`

```js
// public/javascripts/modules/heart.js
import axios from 'axios';
import { $ } from './bling';

function ajaxHeart(e) {
  e.preventDefault();
  axios
    .post(this.action)
    .then(res => {
      const isHearted = this.heart.classList.toggle('heart__button--hearted');
      $('.heart-count').textContent = res.data.hearts.length;
      if (isHearted) {
        this.heart.classList.add('heart__button--float');
        setTimeout(() => this.heart.classList.remove('heart__button--float'), 2500);
      }
    })
    .catch(console.error);
}

export default ajaxHeart;

// public/javascripts/delicious-app.js
import '../sass/style.scss';
...
import ajaxHeart from './modules/heart';
...
const heartForms = $$('form.heart');
heartForms.on('submit', ajaxHeart);
```


## 36 - Displaying our Hearted Stores

1. 为`/hearts`创建 routes

```js
// routes/index.js
router.get('/hearts',
  authController.isLoggedIn,
  catchErrors(storeController.getHearts)
);

// controlelrs/storeController.js
exports.getHearts = async (req, res) => {
  const stores = await Store.find({
    _id: { $in: req.user.hearts }
  });
  res.render('stores', { title: 'Hearted Stores', stores } )
};
```


## 37 - Adding a Reviews Data Model

1. 创建`Review.js`

```js
// start.js
require('./models/Review');

// models/Review.js
const mongoose = require('mongoose');
mongoose.Promise = global.Promise;

const reviewSchema = new mongoose.Schema({
  created: {
    type: Date,
    default: Date.now
  },
  author: {
    type: mongoose.Schema.ObjectId,
    ref: 'User',
    required: 'You must supply an author!'
  },
  store: {
    type: mongoose.Schema.ObjectId,
    ref: 'Store',
    required: 'You must supply an store!'
  },
  text: {
    type:String,
    required: 'Your review must have text!'
  },
  rating: {
    type: Number,
    min: 1,
    max: 5
  }
});

module.exports = mongoose.model('Review', reviewSchema);
```

2. 修改`store.pug`

```jade
//- views/mixins/_reviewForm.pug
mixin reviewForm(store)
  form.reviewer(action=`/review/${store._id}` method="POST")
    textarea(name="text", placeholder="Did you try this place? Have something to say? Leave a review...")
    .reviewer__meta
      .reviewer__stars
        each num in [5, 4, 3, 2, 1]
          input(type="radio" required id=`star${num}` name="rating" value=num)
          label(for=`star${num}`) #{num} Stars
      input.button(type="submit" value="Submit Review 👍")

//- views/store.pug
include mixins/_reviewForm
  //-...
if user
  +reviewForm(store)
```

3. 为`review/${store._id}`创建 routes

```js
// routes/index.js
const reviewController = require('../controllers/reviewController');
...
router.post('/review/:id',
  authController.isLoggedIn,
  catchErrors(reviewController.addReview)
);

// controllers/reviewController.js
const mongoose = require('mongoose');
const Review = mongoose.model('Review');

exports.addReview = async (req, res) => {
  req.body.author = req.user._id;
  req.body.store = req.params.id;
  const newReview = new Review(req.body);
  await newReview.save();
  req.flash('success', 'Review Saved!');
  res.redirect('back');
};
```