## build

简单-构建, 本来就是趋势

能把复杂问题简单化, 才是能人`build.js` - `release.js`

---

## build/build.js

<details>

``` js
/* eslint-env node */

var fs = require('fs');
var rollup = require('rollup');
var uglify = require('uglify-js');
var buble = require('rollup-plugin-buble');
var {version} = require('../package.json');
var banner =
    '/*!\n' +
    ' * vue-resource v' + version + '\n' +
    ' * https://github.com/pagekit/vue-resource\n' +
    ' * Released under the MIT License.\n' +
    ' */\n';

rollup.rollup({
    input: 'src/index.js',
    plugins: [buble()]
})
.then(bundle =>
  bundle.generate({
      format: 'umd',
      banner: banner,
      name: 'VueResource'
  }).then(({code}) => write('dist/vue-resource.js', code, bundle))
)
.then(bundle =>
  write('dist/vue-resource.min.js', banner + '\n' +
    uglify.minify(read('dist/vue-resource.js')).code,
  bundle)
)
.then(bundle =>
  bundle.generate({
      format: 'es',
      banner: banner,
      footer: 'export { Url, Http, Resource };'
  }).then(({code}) => write('dist/vue-resource.esm.js', code, bundle))
)
.then(bundle =>
  bundle.generate({
      format: 'cjs',
      banner: banner
  }).then(({code}) => write('dist/vue-resource.common.js', code, bundle))
)
.catch(logError);

function read(path) {
    return fs.readFileSync(path, 'utf8');
}

function write(dest, code, bundle) {
    return new Promise(function (resolve, reject) {
        fs.writeFile(dest, code, function (err) {
            if (err) return reject(err);
            console.log(blue(dest) + ' ' + getSize(code));
            resolve(bundle);
        });
    });
}

function getSize(code) {
    return (code.length / 1024).toFixed(2) + 'kb';
}

function logError(e) {
    console.log(e);
}

function blue(str) {
    return '\x1b[1m\x1b[34m' + str + '\x1b[39m\x1b[22m';
}

```
</details>

## release.js

<details>

``` js
/* eslint-env node */

var replace = require('replace-in-file');
var version = process.argv[2];

replace({
    files: 'bower.json',
    from: /("version"\s*:\s*")\d+\.\d+\.\d+("\s*,)/g,
    to: '$1' + version + '$2'
});

replace({
    files: 'package.json',
    from: /("version"\s*:\s*")\d+\.\d+\.\d+("\s*,)/g,
    to: '$1' + version + '$2'
});

replace({
    files: 'README.md',
    from: /(\/|@)\d+\.\d+\.\d+/g,
    to: '$1' + version
});

```
</details>

