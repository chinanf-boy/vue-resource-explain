# Vue-resource

「 The plugin for [Vue.js](http://vuejs.org) provides services for making web requests and handle responses using a [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) or JSONP. 」

[![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)
    
Explanation

> "version": "1.5.0"

[github source](https://github.com/pagekit/vue-resource)

~~[english](./README.en.md)~~

---



---

本目录

---

## package.json

``` json
  "scripts": {
    "test": "jest --env=node", // 测试
    "karma": "karma start test/karma.conf.js --single-run", // 浏览器测试
    "build": "node build/build.js", // 构建
    "release": "node build/release.js", // 版本更新
    "webpack": "webpack --config test/webpack.config.js" // 构建-浏览器测试-文件
  },
```

- `test + karma + webpack`

> [test-explain](./test.explain.md)

- `build + release`

> [build-explain](./build.explain.md)

构建和测试方面的解释, 在这里

---

那，接下来，`build/build.js`

``` js
rollup.rollup({
    input: 'src/index.js',
```

---

我们找到了，入口，还有带着-例子🌰看`code`

```js
{
  // GET /someUrl
  this.$http.get('/someUrl').then(response => {

    // get body data
    this.someData = response.body;

  }, response => {
    // error callback
  });
}
```

先单看一种方法`http`

---

## 1. src-index

> 汇总-和-载入Vue

``` js
/**
 * Install plugin.
 */

import Url from './url/index';
import Http from './http/index'; // <====
import Promise from './promise';
import Resource from './resource';
import Util, {options} from './util';

function plugin(Vue) {

    if (plugin.installed) { // vue-插件机制=安装✅标志
        return;
    }

    Util(Vue); // 获取Vue-部分变量 Vue.config.debug .etc

    Vue.url = Url;
    Vue.http = Http;// <==== 全局定义
    Vue.resource = Resource;
    Vue.Promise = Promise;

    Object.defineProperties(Vue.prototype, { // 原型定义

        // $url: { //。。}

        // 例子🌰的示例 this.$http.get()
        $http: {

// ⏰     
// 这是 defineProperties 特性
// 定义当请求 $http 变量时, 
//     调用  get()
            get() {         // ⚠️注意 this.$http.get() 并不是这个 get(){ return ...}
                return options(Vue.http, this, this.$options.http); 
                            // ⚠️而是 this.$http == options(Vue.http, this, this.$options.http)
                            // ⚠️this.$http.get() == options(Vue.http, this, this.$options.http).get()
            }
// ⏰
// 有get,当然也有 set()
// 当 this.$http = "hello"
//  调用 set()
            // set(){
            //     // ...
            // }
        },

        // $resource: { //。。}

        // $promise: { //。。}

    });
}

if (typeof window !== 'undefined' && window.Vue) {
    window.Vue.use(plugin); // Vue-使用，起点来了🚩
}

export default plugin;

```

1.1 `options(Vue.http, this, this.$options.http)`

其实 `vue-resource`本身有两种用法

- `Vue.http.get('/someurl').then` - 全局使用

- `this.$http.get('/someurl').then` - 单个 vue实例-`new Vue({ el: #app //...})` 使用

由此可见, `Vue.http` 与 `this.$http` 的核心是相同, 不同的只是一些 `vue实例-config` 的问题

1.2 [`有关vue-插件官方`](??)

---

### 1.3 options

`src/utils.js`

``` js
// ⬆️文
    // 1:http请求函数 2: Vue实例 3: vue-options.http
options(Vue.http, this, this.$options.http)

//

export function options(fn, obj, opts) {

    opts = opts || {};

    if (isFunction(opts)) {
        opts = opts.call(obj);
    }

    return merge(fn.bind({$vm: obj, $options: opts}), fn, {$options: opts});
}
```

- `this.$options.http`

> 当 `new Vue(options)` 实例新建 -> 在 `new Vue` 自动构建 `options -> this.$options`

1.4 `merge(fn.bind({$vm: obj, $options: opts}), fn, {$options: opts})`

> 把第一变量后-的-变量, 塞进第一变量

``` js
export function merge(target) {

    var args = slice.call(arguments, 1);

    args.forEach(source => {
        _merge(target, source, true); // 每个都要塞
    });

    return target; // <==== 返回本身
}
```

1.5 _merge(target, source, true)

> 把第一变量后-的-变量, 塞进第一变量, 深度塞

``` js
function _merge(target, source, deep) {
    for (var key in source) {
        if (deep && (isPlainObject(source[key]) || isArray(source[key]))) {
            if (isPlainObject(source[key]) && !isPlainObject(target[key])) {
                target[key] = {};
            }
            if (isArray(source[key]) && !isArray(target[key])) {
                target[key] = [];
            }
            _merge(target[key], source[key], deep); // <==== 递归塞进去
        } else if (source[key] !== undefined) {
            target[key] = source[key];
        }
    }
}

```

经过一系列的 `merge` 塞进 -> `target == Vue.http` , 把`new Vue`-`options` 获得

> 返回 `target == Vue.http` 本身

到了这里, 我们可以说**之后**, `this.$http` 的代码逻辑就与`Vue.http` 相同了😊

---

## 2. Http

`src/index.js`

``` js
import Http from './http/index'; // L6
    Vue.http = Http; // L20
```

`src/http/index.js`

``` js
export default function Http(options) { //<===== 3

    var self = this || {}, client = Client(self.$vm); // 客户端

    defaults(options || {}, self.$options, Http.options);

    Http.interceptors.forEach(handler => {

        if (isString(handler)) {
            handler = Http.interceptor[handler]; // {before, method, jsonp, json, form, header, cors}
        }

        if (isFunction(handler)) {
            client.use(handler); // 添加给客户端 client
        }

    });

    // new Request(options) 构建一个http 请求函数对象-
    // client 请求开始
    return client(new Request(options)).then(response => {
         // 获得响应
        return response.ok ? response : Promise.reject(response);

    }, response => {

        if (response instanceof Error) {
            error(response);
        }

        return Promise.reject(response);
    });
}

Http.options = {};

Http.headers = {
    put: JSON_CONTENT_TYPE,
    post: JSON_CONTENT_TYPE,
    patch: JSON_CONTENT_TYPE,
    delete: JSON_CONTENT_TYPE,
    common: COMMON_HEADERS,
    custom: {}
};

Http.interceptor = {before, method, jsonp, json, form, header, cors};
Http.interceptors = ['before', 'method', 'jsonp', 'json', 'form', 'header', 'cors'];

['get', 'delete', 'head', 'jsonp'].forEach(method => {

// 然后到这里了 Http.get     <===== 1
    Http[method] = function (url, options) { 
        
//🧠 原来 只是确定 options.url 和 options.method
// 然后 用来调用 Http 本身
        return this(assign(options || {}, {url, method})); // Http.get() => this == Http <===== 2
    };

});

['post', 'put', 'patch'].forEach(method => {

    Http[method] = function (url, body, options) {
        return this(assign(options || {}, {url, method, body}));
    };

});
```

- `client = Client(self.$vm);`

- `client(new Request(options)).then`

---

#### Tips⏰

我们来捋一捋吧, vue-resource 主要作用

1. 收集用户-输入

`Vue.http.get(url)` 或者 `Vue.http.post()` 之类的函数, 都只是收集用户的输入

相当于`options.get = get; options.url = url; Vue.http(options)` == Vue.http.get(url)

2. 保存用户-输入, 调用-请求

client 作为-请求的入口, 隐藏了许多 , 带着 收集-`options`

`client = Client(self.$vm)` -> $vm

`client(new Request(options)).then` -> new Request(options)

> `return client(new Request(options)).then(response => {` 可以看到 then 之后就已经获得数据了

下面👇我们来[解释这部分](#client)

---

2.1 🌿🀄️ 有关`promise`问题 , 作为 es6 的重头戏, 

`vue-resource` -在请求阶段- 基本上建立在`promise`上的

<details>

<summary> 所以, 如果你还不理解promise , 点击</summary>

[es6.ruanyifeng-promise 宝藏在这里, 自己寻找吧](http://es6.ruanyifeng.com/#docs/promise)

Promise 新建后就会立即执行。

``` js
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');

// Promise
// Hi!
// resolved
```


</details>

---

## client

``` js
/**
 * Base client.
 */

import Promise from '../../promise';
import xhrClient from './xhr';
import nodeClient from './node';
import {warn, when, isObject, isFunction, inBrowser} from '../../util';

export default function (context) {

    const reqHandlers = [sendRequest], resHandlers = [];

    if (!isObject(context)) {
        context = null;
    }

    function Client(request) { // 从上面⬆️来看我们已经来到这里了 <==== 
        while (reqHandlers.length) { 
        // reqHandlers 来自
        // client.use(handler); // 添加给客户端 client

            const handler = reqHandlers.pop(); // 从后往前 
            // handler == cors
            // reqHandlers == [sendRequest, before, method, jsonp, json, form, header]

            if (isFunction(handler)) {

                let response, next;
                // 默认情况 所有 Http.interceptor = {before, method, jsonp, json, form, header, cors}
                // 都会运行一边， 设置 request
                
                // 其中最重要的 json
                response = handler.call(context, request, val => next = val) || next;

                if (isObject(response)) {
                    // 能进来这里, 是 sendRequest 的作用, 也是真正的网络请求代码
                    // response 属于 Promise 
                    return new Promise((resolve, reject) => {
//⏰ 可以看到这段简直就是 Promise 大集合 
// 所以, 先理解Promise     
                        // Array.forEach 并没有 return 的能力
                        // 所以这段循环♻️, 是对获得的数据进行处理
                        resHandlers.forEach(handler => { // handler- json 处理 返回数据
                            response = when(response, response => {
                                return handler.call(context, response) || response;
                            }, reject);
                        });

                        when(response, resolve, reject);

                    }, context); // 添加了 Vue 的实例

                    // 这里的 Promise 并不是单纯的, 它添加了 Vue 的实例, 作为this

                    // ❓, return/resolve 哪去了, 怎么返回 都在 when 里面
                }

                if (isFunction(response)) { // 其中最重要的 json
                    resHandlers.unshift(response); // 会添加进来
                }

            } else {
                warn(`Invalid interceptor of type ${typeof handler}, must be a function`);
            }
        }
    }

    Client.use = handler => {
        reqHandlers.push(handler);
    };

    return Client;
}

function sendRequest(request) {

    const client = request.client || (inBrowser ? xhrClient : nodeClient); 
    // 选择 请求代码 如果
    // 在 node 用 http框架 got
    // 在 browser 用 xhr

    return client(request); 
}
```

- `utils.js when`

``` js
export function when(value, fulfilled, rejected) {

    var promise = Promise.resolve(value); // resolve 在这里
    // 即使没有显著的 return 或者 resolve , 也能返回给⬆️集

    if (arguments.length < 2) {
        return promise;
    }

    // return promise 等待⌛️ 网络请求那层Promise数据下来
    return promise.then(fulfilled, rejected);
}
```

---

又捋一捋, 配合[ JSbin.com - example ](http://jsbin.com/dogarat/7/edit?js,console)

多层 `Promise` 时刻注意⚠️第一层 

1. `response = handler.call(context, request, val => next = val) || next;`

> 网络请求层 xhrClient | nodeClient, 只有这一层有数据下来, 后面的 Promise 才有作用

<details>

``` js
function sendRequest(request) {

    const client = request.client || (inBrowser ? xhrClient : nodeClient); 
    // 选择 请求代码 如果
    // 在 node 用 http框架 got
    // 在 browser 用 xhr

    return client(request); 
}
```

</details>

2. ` response = when(response, response => {`

> 数据处理-层

3. `when(response, resolve, reject);`

> 返回-层

4.                 

``` js
// src/client/index
if (isObject(response)) {
                    return new Promise((resolve, reject){ // <== 4.1
                        // .. 
                    }
// src/index
    return client(new Request(options)).then(response => { // <== 4.2 客户端简单处理

        return response.ok ? response : Promise.reject(response);
```

5. 用户层

``` js
{
  // GET /someUrl
  this.$http.get('/someUrl').then(response => { // <===

    // get body data
    this.someData = response.body;

  }, response => {
    // error callback
  });
}
```

> 头真晕

---