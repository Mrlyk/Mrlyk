# JS doc

js 注释的规范编写，常规的就不做说明了。这里主要是记录一些容易忘记的。

## 类注释

```js
/**
 * @class Person
 * @classdesc Creates a new Person
 */
class Person {
    constructor(props) {
        this.props = this.props
    }
}

```

## 对象注释

```js
export default class ValidatorFactor {
    /**
     * 创建校验工厂
     * @param {object} props
     * @param {function} props.request 校验请求
     * @param {string} props.prop 请求传的值的 key，会和参数合并
     * @param {string} [props.trigger = 'blur'] 触发方式，非必需的
     * @param {object} [props.params] 请求额外参数，非必需的
     * @param {string} [props.initValue = ''] 请求传的初始值，非必需的
     * @param {string} [props.errMsg = '校验失败'] 校验失败的提示信息，非必需的
     * @param {function} [cb = () => {}] 回调函数，非必需的
     */
    constructor(props = {}, cb = () => {}) {
        const {trigger = 'blur', request, params, prop, initValue = '', errMsg = '校验失败'} = props;
        this.trigger = trigger;
        this.request = request;
        this.params = params;
        this.prop = prop;
        this.initValue = initValue;
        this.errMsg = errMsg;
        this.cb = cb;
        this.validator = this.valid.bind(this);
    }
}
```

## 对象解构注释

使用缺省的对象 params 来代替被解构的对象

```js
/**
* 
* @param {object} params
* @param {number} params.type
*/         
function isTest({ type }) {
  return type === 1 || type === 4;
}
```



## 参考网站

- JSDoc 在线手册：http://www.dba.cn/book/jsdoc/