## test

本项目主要集中在, karma 浏览器测试中, jest 更像是随带的

---

## jest

`test/http.test.js`

<details>

``` js
var Vue = require('vue');
var VueResource = require('../dist/vue-resource.common.js');

Vue.use(VueResource);

describe('Vue.http', function () {

    it('post("jsfiddle.net/html")', () => {

        return Vue.http.post('http://jsfiddle.net/echo/html/', {html: 'text'}, {emulateJSON: true}).then(res => {

            expect(res.ok).toBe(true);
            expect(res.status).toBe(200);
            expect(typeof res.body).toBe('string');
            expect(res.body).toBe('text');

        });

    });

    it('post("jsfiddle.net/json")', () => {

        return Vue.http.post('http://jsfiddle.net/echo/json/', {json: JSON.stringify({foo: 'bar'})}, {emulateJSON: true}).then(res => {

            expect(res.ok).toBe(true);
            expect(res.status).toBe(200);
            expect(typeof res.body).toBe('object');
            expect(res.body.foo).toBe('bar');

        });

    });

});

```

</details>


## karma

`test/karma.config.js`

<details>

``` js
module.exports = config => {

  config.set({

    basePath: __dirname,

    frameworks: ['jasmine'],

    browsers: ['Chrome', 'Safari', 'Firefox'],

    files: [
      'index.js',
      {pattern: 'data/*', included: false},
    ],

    preprocessors: {
      'index.js': ['webpack'] // <=== 测试总文件, webpack 构建
    },

    proxies: {
      '/data/': '/base/data/'
    },

  });

};

```


</details>

## webpack

<details>

`test/webpack.config.js` - 测试总文件构建

``` js
var webpack = require('webpack');

module.exports = {
    entry: __dirname + '/index.js',
    output: {
        path: __dirname + '/',
        filename: 'specs.js' // <----
    },
    module: {
        loaders: [
            {test: /\.js$/, loader: 'buble-loader', exclude: /node_modules/}
        ]
    },
    plugins: [
        new webpack.optimize.ModuleConcatenationPlugin()
    ]
};

```

</details>