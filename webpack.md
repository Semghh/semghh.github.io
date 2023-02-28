# 1.webpack











# 2. 零碎知识点





## 2.1 resolve

提供的 resolve选项能够设置模块到底如何被解析。通常 webpack会提供一些合理的默认值。但开发者也可以按照自己需求修改细节。



### 2.2.1 @ alias

别名。  老熟人了,在mybatis也有类似的功能。

`object` 



定义的 alias可以在 `import` 和 `require` 中使用

#### 2.2.1.1 语法



```js
alias: {
  Utilities: path.resolve(__dirname, 'src/utilities/'),
  Templates: path.resolve(__dirname, 'src/templates/')
}
```



#### 2.2.1.2 实例

```js
//以前
import Utility from '../../utilities/utility';

//现在
import Utility from 'Utilities/utility';
```







