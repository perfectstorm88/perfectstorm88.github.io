---
layout: post
categories: js
---

js中的slice和splice功能比较强大的，但是平时review代码看到的却不是很多,深入研究了下，发现挺有意思：

# splice
通过删除或添加元素来修改数组，此方法会改变原数组。

> array.splice(start[, deleteCount[, item1[, item2[, ...]]]])

- start 开始位置
  - 如果大于数组长度，表示从从最后添加
  - 如果为负值，表示从末位开始的第几位
  - 如果负值绝对值大于数组长度，表示从0开始
- deleteCount 删除元素的个数
  - 如果大于start 之后的元素的总数，从 start 后面的元素都将被删除（含第 start 位）。
  - 如果被省略了，则start之后所有元素被删除
  - 如果是 0 或者负数，则不移除元素。这种情况下，至少应添加一个新元素


```js
a=[0,1,2,3]
a.splice(2,2,a[3],a[2]) // 把2、3进行对调
console.log(a) // output:[ 0, 1, 3, 2 ]
a.splice(2,1)  // 删除第二个元素
console.log(a) // [ 0, 1, 2 ]
a.splice(2,0,'x') // 从第二个位置增加一个新元素
console.log(a) // [ 0, 1, x, 2 ]
a.splice(2)  // 从第二个位置 之后删除数据
console.log(a)  // [ 0, 1 ]
```

# slice 
浅拷贝一个新数组[begin,end),原始数组不会被改变。(slice中文含义为切片)

> array.slice([start[, end]])

- start 开始位置
  - 如果大于数组长度，返回空数组
  - 如果为负值，表示从末位开始的第几位
  - 如果负值绝对值大于数组长度，表示从0开始
  - 如果省略，表示从0开始
- end 终止处的索引
  - 如果被省略或大于总长度，会一直提取到原数组末尾
  - 如果负数，表示倒数第几个
```js
a=[0,1,2,3]
a.slice() // 整个数组拷贝
a.slice(-2)  // 最后两个元素
a.slice(-20)  // 整个数组拷贝
a.slice(1,2)  // 一个元素
a.slice(1,-1)  // 去掉两头
a.slice(1,20)  // 去掉两头
```



# 总结,敲黑板敲黑板
- splice中文含义为拼接,如果splice函数不使用deleteCount就非常好理解，用了deleteCount打破了函数名的意思
  - 重新思考了下，在某个位置上删掉几个元素，把剩余部分前后拼接起来，也叫splice，还是名副其实的
- slice中文含义为切片，函数确实也是切片


# 参考：

- [splice](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/splice)