#### 立即执行函数，避免全局作用域污染

``` javascript
(function () {
    // 内部执行逻辑
})()
```

#### 判断当前执行环境

``` javascript
// 这里不用 window 对象，主要是 self 更具有通用性
var browserEnv = typeof self === 'object' && self.self === self && self

// node 执行环境
var nodeEnv = typeof global === 'object' && global.global === global && global

var root = browserEnv || nodeEnv || this || {};
```

#### 在 node 环境中，_ 对象作为模块导出

``` javascript
if (typeof exports != 'undefined' && !exports.nodeType) {
    if (typeof module != 'undefined' && !module.nodeType && module.exports) {
        exports = module.exports = _;
    }
    exports._ = _;
} else {
    root._ = _;
}
```

#### 保存之前的 _ 对象，有可能其他框架或者插件也用了 _ 作为对象

``` javascript
var previousUnderscore = root._;
```
#### 如何自定义 _ 对象，避免跟已有的 _ 对象冲突

``` javascript
// 如下例，以后就直接用 util 调用 underscore 方法
var util = _.noConflict();

// 具体实现
_.noConflict = function () {
    root._ = previousUnderscore;
    return this;
};
```

#### 用变量接住一些常用方法，方便调用

``` javascript
var ArrayProto = Array.prototype, ObjProto = Object.prototype;
var SymbolProto = typeof Symbol !== 'undefined' ? Symbol.prototype : null;

var push = ArrayProto.push,
    slice = ArrayProto.slice,
    toString = ObjProto.toString,
    hasOwnProperty = ObjProto.hasOwnProperty;
```

#### 如何处理 undefined
``` javascript
// 以下局部变量 undefined 被改写了，所以直接 a === undefined 就是判断两个变量之间的值
// 跟我们判断一个变量是否定义的初衷不一样，故无法准确判断是否未定义
function test(a) {
    var undefined = 1;
    console.log(undefined); // => 1
    if(a === undefined) {
        // ...
    }
}

// 可以用以下方法来准确判断一个变量未定义
// 以下方法返回的肯定是准确的未定义
// void 运算符对指定的表达式求值，void 表达式肯定返回 undefined， 选 0 是因为计算开销最小
void 0;
```

#### 迭代
``` javascript
_.map = _.collect = function (obj, iteratee, context) {
    // 迭代函数或者迭代对象等迭代物
    iteratee = cb(iteratee, context);
    var keys = !isArrayLike(obj) && _.keys(obj),
        length = (keys || obj).length,
        results = Array(length);
    for (var index = 0; index < length; index++) {
        var currentKey = keys ? keys[index] : index;
        results[index] = iteratee(obj[currentKey], currentKey, obj);
    }
    return results;
};

var cb = function (value, context, argCount) {
    // 是否用自定义的 iteratee
    if (_.iteratee !== builtinIteratee) return _.iteratee(value, context);
    // 如果是数组，函数
    if (value == null) return _.identity;
    if (_.isFunction(value)) return optimizeCb(value, context, argCount);
    if (_.isObject(value) && !_.isArray(value)) return _.matcher(value);
    return _.property(value);
};
```

#### underscore 中的创建对象

``` javascript
// 先用变量接住 create 方法，方便调用
var nativeCreate = Object.create;
// 空的构造函数
var Ctor = function () {};
/**
 * 创建一个对象，该对象继承自prototype
 * 并且保证该对象在其原型上挂载属性不会影响所继承的prototype
 * @param {object} prototype
 */
var baseCreate = function (prototype) {
    if (!_.isObject(prototype)) return {};
    // 如果存在原生的创建方法（Object.create），则用原生的进行创建
    if (nativeCreate) return nativeCreate(prototype);
    // 利用Ctor这个空函数，临时设置对象原型
    Ctor.prototype = prototype;
    // 创建对象，result.__proto__ === prototype
    var result = new Ctor;
    // 还原Ctor原型
    Ctor.prototype = null;
    return result;
};
```

#### 数组和对象的迭代
``` javascript
_.each = _.forEach = function (obj, iteratee, context) {
    iteratee = optimizeCb(iteratee, context);
    var i, length;
    if (isArrayLike(obj)) {
        for (i = 0, length = obj.length; i < length; i++) {
            iteratee(obj[i], i, obj);
        }
    } else {
        var keys = _.keys(obj);
        for (i = 0, length = keys.length; i < length; i++) {
            iteratee(obj[keys[i]], keys[i], obj);
        }
    }
    return obj;
};
```

#### map 在 underscore 中的实现

``` javascript
 _.map = _.collect = function (obj, iteratee, context) {
    iteratee = cb(iteratee, context);
    var keys = !isArrayLike(obj) && _.keys(obj),
        length = (keys || obj).length,
        results = Array(length);
    for (var index = 0; index < length; index++) {
        var currentKey = keys ? keys[index] : index;
        results[index] = iteratee(obj[currentKey], currentKey, obj);
    }
    return results;
};
````