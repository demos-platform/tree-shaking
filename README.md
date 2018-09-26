Webpack 4 默认启用了 Tree Shaking，同时消除了副作用。

### 测试一：[官网 demo](https://webpack.js.org/guides/tree-shaking/#src/components/Sidebar/Sidebar.jsx)

根据官网的提示，使用了 `import`、`export`，以及将 `mode` 设置为 `production` 时，自动开启 `Tree Shaking`

```js
// webpack.config.js
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  },
  mode: 'production',
}
```

```js
// index.js
import { cube } from './math.js'

console.log(cube(5))

// main.js
export function square(x) {
  console.log('square')
  return x * x
}

export function cube(x) {
  console.log('cube')
  return x * x * x
}
```

实验结果：打包文件里有 `cube` 函数，无 `square` 函数。

### 测试二：测试副作用

关于副作用，可以参考此文 [你的Tree-Shaking并没什么卵用](https://segmentfault.com/a/1190000012794598#articleHeader6)，阅读完这篇文章，我们得知获取对象的属性这个操作是有副作用的(触发 getter 操作)，老版本的 webpack 配合的 uglify 插件不会删除有副作用的代码，这也是大部分 Tree Shaking 不生效的罪魁祸首。

下面这个案例可以证明 `webpack 4` 中确实默认消除了副作用。

```js
// webpack.config.js
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  },
  mode: 'production',
}
```

```js
// index.js
import { cube } from './math.js'

console.log(cube(5))

// main.js
export function square(x) {
  console.log('square') // 如果没有消除副作用，打包代码中将存在 'square'
  return x.a
}

export function cube(x) {
  console.log('cube')
  return x * x * x
}
```

实验结果：打包文件里有 `cube` 函数，无 `square` 函数。

### 测试三：让副作用生效(实验失败)

假如我们现在想要保留一段`具有副作用但是没引用`的代码，那该怎么做呢？官方提示在 `package.json` 上加上 `sideEffects` 属性。来试一下吧。

```json
// package.json
{
  ...
  "sideEffects": [
    "./src/math.js",
  ],
  ...
}
```

其它代码与测试二中的代码保持一致。

实验结果：打包文件里有 `cube` 函数，无 `square` 函数。(`sideEffects` 属性实验失败。。。)

### 测试四：引入 webpack 打包的文件

实验结果：无法 Tree Shaking

暂时解释：`umd` 形式的包不支持 Tree Shaking。

### 测试五：引入 rollup 打包的文件

待实验