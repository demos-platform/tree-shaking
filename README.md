Webpack 4 默认启用了 Tree Shaking，以下为当前测试配置：

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
  return x * x
}

export function cube(x) {
  return x * x * x
}
```

实验结果：打包文件里无 `square` 函数。

### 测试二：

```js
// index.js
import { cube } from './math.js'

console.log(cube(5))

// main.js
export function square(x) {
  return x
}

console.log(square('hi,test'))

export function cube(x) {
  return x * x * x
}
```

实验结果：打包文件里有 `square` 函数以及 `cube` 函数

### 测试三：

```js
// index.js
import { cube } from './math.js'

console.log(cube(5))

// main.js
export function square(x) {
  return x.a
}

console.log(square({ a: 'hi,test' }))

export function cube(x) {
  return x * x * x
}
```

实验结果：打包文件里有 `square` 函数以及 `cube` 函数

### 相关文章

[你的Tree-Shaking并没什么卵用](https://segmentfault.com/a/1190000012794598#articleHeader6)