# js 基准性能测试 Benchmark

官方仓库：https://github.com/bestiejs/benchmark.js

有时候我们想知道 js 怎么写性能更好，比如想判断字符串中是否包含某个字母，includes 和 indexOf 谁的性能更优？这时候我们就需要使用一些基准性能测试工具来帮我们进行比较。

Benchmark 就是这样的一个工具。

## 使用示例

比较正则表达式匹配和 indexOf 判断的性能。

```js
const Benchmark = require('benchmark');
const suite = new Benchmark.Suite;

// 添加测试
suite.add('RegExp#test', function() {
    /o/.test('Hello World!');
})
    .add('String#indexOf', function() {
        'Hello World!'.indexOf('o') > -1;
    })
// add listeners
    .on('cycle', function(event) {
        console.log(String(event.target));
    })
    .on('complete', function() {
        console.log('Fastest is ' + this.filter('fastest').pluck('name'));
    })
// run async
    .run({ 'async': true });


// 输入如下
// RegExp#test x 9,847,928 ops/sec ±1.47% (83 runs sampled)  // 每秒执行了 9847928 次，误差 ±1.47%，进行了 83 次采样测试
// String#indexOf x 23,366,017 ops/sec ±0.91% (96 runs sampled) // 每秒执行了 23366017 次，误差 ±0.91%
// Fastest is String#indexOf // 结论 indexOf 更快
```





## jsPerf

站点：[http://jsperf.com/](http://jsperf.com/)

基于 Benchmark 的一个在线测试网站，用于测试代码片段的性能，也可以给出不同浏览器上的性能比较。