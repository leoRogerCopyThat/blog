---
title: JavaScript常用高阶函数小记
toc: true
categories: Code
tags: [ 'JavaScript', 'Code', 'Notes' ]
excerpt: 高阶函数是对其他函数进行操作的函数，可以将它们作为参数或返回它们。简单来说，高阶函数是一个函数，它接收函数作为参数或将函数作为输出返回。
date: 2021-11-09 17:10:23
updated: 2021-11-09 17:10:23
---
# 什么是高阶函数?
高阶函数是对其他函数进行操作的函数，可以将它们作为参数或返回它们。

简单来说，高阶函数是一个函数，它接收函数作为参数或将函数作为输出返回。

## 1. 函数可以作为参数
```JavaScript
function bar(fn){
    if(typeof fn === "function"){
        fn()
    }
}
//调用
bar(function () {})
```
## 2. 函数可以作为返回值
```JavaScript
function bar(){
    return function (){}
}
//调用
const fn = bar ()
console.log(fn)
```

# JS中的高阶函数

## 1. map
***
* `map()` 返回一个新的数组，数组中的元素为原始数组调用函数处理后的值。
* `map()` 不会对空数组进行检测。
* `map()` 不会改变原始数组。

传递给 `map()` 方法的回调函数接受 **3** 个参数：`currentValue`，`index` 和 `array`。

1. `currentValue`：**必须**。当前元素的的值。
2. `index`：**可选**。当前元素的索引。
3. `arr`：**可选**。当前元素属于的数组对象。

有这样一个数组 `[10,20,45,50,65,150,70,40]` 现在有如下需求

把数组中所有的元素 * **2**。

```JavaScript
let arr = [10, 20, 45, 50, 65, 150, 70, 40];
let newArr = arr.map((item) => {
    return item * 2;
});
console.log(newArr)// [20, 40, 90, 100, 130, 300, 140, 80]
```

## 2. filter
***

* `filter()` 方法创建一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素。
* `filter()` 不会对空数组进行检测。
* `filter()` 不会改变原始数组。

传递给 `filter()` 方法的回调函数接受 **3** 个参数：`currentValue`，`index` 和 `array`。

1. `currentValue`：必须。当前元素的的值。
2. `index`：可选。当前元素的索引。
3. `arr`：可选。当前元素属于的数组对象。

有这样一个数组 `[20, 40, 90, 100, 130, 300, 140, 80]` 现在有如下需求

返回数组中所有小于 **100** 的元素。

```JavaScript
let arr = [20, 40, 90, 100, 130, 300, 140, 80];
let newArr = arr.filter((item) => {
    return item < 100;
});
console.log(newArr);//[20, 40, 90, 80]
```

## 3. forEach
***

* `forEach()` 方法类似于 `map()`，传入的函数不需要返回值,并将元素传递给回调函数。
* `forEach()` 对于空数组是不会执行回调函数的。
* `forEach()` 不会返回新的数组,总是返回 `undefined`。

传递给 `forEach()` 方法的回调函数接受 **3** 个参数：`currentValue`，`index` 和 `array`。

1. `currentValue`：必须。当前元素的的值。
2. `index`：可选。当前元素的索引。
3. `arr`：可选。当前元素属于的数组对象。

有这样一个数组 `[20, 40, 90, 100]` 现在有如下需求

分别打印数组中的元素及其索引。

```JavaScript
let arr = [20, 40, 90, 100];
let newArr = arr.forEach((item,index) => {
    console.log(item,index);
});
//20 0
//40 1
//90 2
//100 3
```

## 4. sort
***

* `sort()` 方法用于对数组的元素进行排序。
* `sort()` 会修改原数组。

`sort()` 方法接受一个可选参数,用来规定排序顺序,必须是函数。

如果没有传递参数, `sort()` 方法默认把所有元素先转换为 `String` 再排序，根据 `ASCII` 码进行排序。

如果想按照其他标准进行排序，就需要提供比较函数，该函数要比较两个值，然后返回一个用于说明这两个值的相对顺序的数字。比较函数应该具有两个参数 **a** 和 **b**，其返回值如下：

* 若 `a` 小于 `b`，在排序后的数组中 `a` 应该出现在 `b` 之前，则返回一个小于 `0` 的值。
* 若 `a` 等于 `b`，则返回 `0`。
* 若 `a` 大于 `b`，则返回一个大于 `0` 的值。

有这样一个数组 `[10, 20, 1, 2]` 现在有如下需求

按从小到大排序

```JavaScript
let arr = [10, 20, 1, 2];
arr.sort(function (x, y) {
    if (x < y) {
        return -1;
    }
    if (x > y) {
        return 1;
    }
    return 0;
});
console.log(arr); // [1, 2, 10, 20]
```

按从大到小排序

```JavaScript
let arr = [10, 20, 1, 2];
arr.sort(function (x, y) {
    if (x < y) {
        return 1;
    }
    if (x > y) {
        return -1;
    }
    return 0;
}); // [20, 10, 2, 1]
```

## 5. some
***

* `some()` 方法用于检测数组中的元素是否满足指定条件。
* `some()` 方法会依次执行数组的每个元素。
* 如果有一个元素满足条件，则表达式返回 `true`, 剩余的元素不会再执行检测。
* 如果没有满足条件的元素，则返回 `false` 。
* `some()` 不会对空数组进行检测。
* `some()` 不会改变原始数组。

传递给 `some()` 方法的回调函数接受 `3` 个参数：`currentValue`，`index` 和 `array`。

1. `currentValue`：必须。当前元素的的值。
2. `index`：可选。当前元素的索引。
3. `arr`：可选。当前元素属于的数组对象。

有这样一个数组 `[10, 20, 1, 2]` 现在有如下需求。
判断数组中是否含有大于 `10` 的数字。

```JavaScript
let arr = [10, 20, 1, 2];
let result = arr.some((item) => {
    return item > 10;
});
console.log(result); // true
```

## 6. every
***

* `every()` 方法用于检测数组所有元素是否都符合指定条件。
* `every()` 方法会依次执行数组的每个元素。
* 如果数组中检测到有一个元素不满足，则整个表达式返回 `false` ，且剩余的元素不会再进行检测。
* 如果所有元素都满足条件，则返回 `true`。
* `every()` 不会对空数组进行检测。
* `every()` 不会改变原始数组。

传递给 `every()` 方法的回调函数接受 `3` 个参数：`currentValue`，`index` 和 `array`。

1. `currentValue`：必须。当前元素的的值。
2. `index`：可选。当前元素的索引。
3. `arr`：可选。当前元素属于的数组对象。

有这样一个数组 `[11, 20, 51, 82]` 现在有如下需求。
判断数组中是否所有的数字都大于 `10`。

```JavaScript
let arr = [11, 20, 51, 82];
let result = arr.every((item) => {
    return item > 10;
});
console.log(result); // true
```

## 7. reduce
***

* `reduce()` 方法接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终计算为一个值。
* `reduce()` 对于空数组是不会执行回调函数的。

`reduce` 方法接收两个参数

* 回调函数
* 一个可选的 `initialValue` (初始值)。如果不传第二个参数 `initialValue`，则函数的第一次执行会将数组中的第一个元素作为 `prev` 参数返回。

传递给 `reduce()` 方法的回调函数接受 **4** 个参数：`prev`, `current`, `currentIndex`, `arr`。

`prev`：必须。函数传进来的初始值或上一次回调的返回值。
`current`：必须。数组中当前处理的元素值。
`currentIndex`：可选。当前元素索引。
`arr`：可选。当前元素所属的数组本身。

有这样一个数组 `[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]` 现在有如下需求

返回数组所有元素累加之和

```JavaScript
let arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let sum = arr.reduce((prev, current) => {
    return prev + current;
}, 0);
console.log(sum); // 55
```

## 8. reduceRight
***

`reduceRight()` 方法的功能和 `reduce()` 功能是一样的，不同的是 `reduceRight()` 从数组的末尾向前将数组中的数组项做累加。

## 9. find
***

* `find()` 方法用于查找符合条件的第一个元素，如果找到了，返回这个元素，否则，返回 `undefined`。
* `find()` 不会对空数组进行检测。
* `find()` 不会改变原始数组。

* 传递给 `find()` 方法的回调函数接受 `3` 个参数：`currentValue`，`index` 和 `array`。

`currentValue`：必须。当前元素的的值。
`index`：可选。当前元素的索引。
`arr`：可选。当前元素属于的数组对象。

有这样一个数组 `[11, 20, 51, 82]` 现在有如下需求
返回数组中第一个大于 `50` 的元素

```JavaScript
let arr = [11, 20, 51, 82];
let result = arr.find((item) => {
    return item > 50;
}, 0);
console.log(result); // 51
```

## 10. findIndex
***

`findindex()` 和 `find()` 类似，也是查找符合条件的第一个元素，不同之处在于 `findindex()` 会返回这个元素的索引，如果没有找到，返回 `-1` 。

有这样一个数组 `[11, 20, 51, 82]` 现在有如下需求。

返回数组中第一个大于 `50` 的元素索引。

```JavaScript
let arr = [11, 20, 51, 82];
let result = arr.findIndex((item) => {
    return item > 50;
}, 0);
console.log(result); // 2
```

# 链式调用高阶函数

假如有一个数组 `[10, 20, 45, 50, 65, 150, 70, 40]`

需求一：给数组中的每个元素 `* 2`

我们使用的是 `map()` ,得到了 `[20, 40, 90, 100, 130, 300, 140, 80]`

需求二：返回需求一中得到的新数组所有小于 `100` 的元素

我们使用的是 `filter()`, 得到了 `[20, 40, 90, 80]`

需求三：计算需求二中得到的新数组所有元素之和

我们使用的是 `reduce()`,得到了 `230`

我们可以链式调用上述三个函数，来得到最终的结果

```JavaScript
let arr = [10, 20, 45, 50, 65, 150, 70, 40];
let total = arr.map(n => n * 2).filter(n => n < 100).reduce((pre, n) => pre + n);
console.log(total); // 230
```

原文：https://juejin.cn/post/7028385753042255909
