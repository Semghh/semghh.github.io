# 1. async-validator

异步校验器。

在Element-UI中，表单校验使用的就是 async-validator



github地址: `https://github.com/yiminghe/async-validator`





安装

```
npm i async-validator
```









## 1.1 快速使用



基础的使用包含了  `定义一个描述器(descriptor)` 并给他分配一个 `schema` ，传入的Object对象将得到校验。 以及一个回调函数作为校验方法。



```js
import Schema from 'async-validator';

//描述器
const descriptor = {
  name: {
    type: 'string',   //类型
    required: true,   //是否必须
    validator: (rule, value) => value === 'muji',         //校验方法
  },
  age: {
    type: 'number',  //类型      
    asyncValidator: (rule, value) => {         //异步校验器
      return new Promise((resolve, reject) => {
        if (value < 18) {
          reject('too young');  // reject with error message
        } else {
          resolve();
        }
      });
    },
  },
};

//校验器,  new 一个Schema类,传入校验器。
const validator = new Schema(descriptor);

//使用 .validate方法，校验一个Obj。  传入校验对象,和一个回调函数
validator.validate({ name: 'muji' }, (errors, fields) => {
  if (errors) {
    // 校验失败, errors是一个包含了全部异常的数组
    // fields is an object keyed by field name with an array of
    // errors per field
    return handleErrors(errors, fields);
  }
  //校验通过
});

//使用Promise
validator.validate({ name: 'muji', age: 16 }).then(() => {
  // validation passed or without error message
}).catch(({ errors, fields }) => {
  return handleErrors(errors, fields);
});
```







# 2. API







## 2.1  Type 校验类型



可选的校验类型取下：



常规的校验类型:

- `string`: Must be of type `string`. `This is the default type.`
- `number`: Must be of type `number`.
- `boolean`: Must be of type `boolean`.
- `integer`: Must be of type `number` and an integer.
- `float`: Must be of type `number` and a floating point number.

- `array`: Must be an array as determined by `Array.isArray`.
- `object`: Must be of type `object` and not `Array.isArray`.



比较特殊的

- `method`: Must be of type `function`.
- `regexp`: Must be an instance of `RegExp` or a string that does not generate an exception when creating a new `RegExp`.
- `enum`: Value must exist in the `enum`.
- `date`: Value must be valid as determined by `Date`
- `url`: Must be of type `url`.
- `hex`: Must be of type `hex`.
- `email`: Must be of type `email`.
- `any`: Can be any type.







## 2.2 Required 是否必须

`boolean`  类型



指明这个属性是否必须在被校验对象中存在。





## 2.3 Pattern 正则

表示该属性必须满足一个正则。否则验证失败







## 2.4 Range



A range is defined using the `min` and `max` properties. For `string` and `array` types comparison is performed against the `length`, for `number` types the number must not be less than `min` nor greater than `max`.





## 2.5 Enumerable

在指定枚举类中的值



> Since version 3.0.0 if you want to validate the values `0` or `false` inside `enum` types, you have to include them explicitly.



To validate a value from a list of possible values use the `enum` type with a `enum` property listing the valid values for the field, for example:

```js
const descriptor = {
  role: { type: 'enum', enum: ['admin', 'user', 'guest'] },
};
```







## 2.6 Length

要验证字段的精确长度，请指定len属性。







对于字符串和数组类型的比较是在length属性上执行的。

对于数字类型，该属性指示与数字的精确匹配，即，它可能只严格等于len。







例如：

```js
const descriptor = {
  roles: {
    type: 'array',
    required: true,
    len: 3,   //数组length的值为len
    fields: {
      0: { type: 'string', required: true },
      1: { type: 'string', required: true },
      2: { type: 'string', required: true },
    },
  },
};
```















## 2.6 Deep Rules

深度校验规则。如果我们想校验一个具有多层属性的对象。可以像如下嵌套规则。



```js
const descriptor = {
    
  address: {
    type: 'object',
    required: true,
    fields: { //字段
      street: { type: 'string', required: true },  //嵌套属性
      city: { type: 'string', required: true },
      zip: { type: 'string', required: true, len: 8, message: 'invalid zip' },
    },
  },
  name: { type: 'string', required: true },
};

//校验器
const validator = new Schema(descriptor);
validator.validate({ address: {} }, (errors, fields) => {
  // errors for address.street, address.city, address.zip
});
```



### 2.6.1 注意点

如果 父rule 没有声明 `requeired:true` 的话，那么子属性全都不存在也是合法的，同时子属性的校验规则也完全不生效。

因为 Nothing to validate







### 2.6.2 校验集合



The parent rule is also validated so if you have a set of rules such as:

```js
const descriptor = {
  roles: {
    type: 'array',
    required: true,
    len: 3,
    fields: {
      0: { type: 'string', required: true },
      1: { type: 'string', required: true },
      2: { type: 'string', required: true },
    },
  },
};
```







And supply a source object of `{ roles: ['admin', 'user'] }` then two errors will be created. One for the array length mismatch and one for the missing required array entry at index 2.

同样支持对象源的校验规则。

这样的对象源将会同时创建2种规则： 数组长度不匹配。或者 缺失其中的某一个元素







## 2.7 message

当参数校验失败，自定义带有的失败消息.



根据您的应用程序需求，您可能需要i18n支持，或者你也可能喜欢不同的验证错误消息。





实现这一点最简单的方法是将一条消息分配给一个规则：

```js
{ 
    name: { 
        type: 'string', 
        required: true,
        message: 'Name is required' 
    } 
}
```





