# 选择排序

第一轮选择当前数组中最小的项放在第一位，第二轮选择数组中最小的项放在第二位，以此类推。

表现最稳定的排序算法之一，因为无论什么数据进去都是 O(n²) 的时间复杂度，所以用到它的时候，数据规模越小越好。唯一的好处可能就是不占用额外的内存空间了吧。理论上讲，选择排序可能也是平时排序一般人想到的最多的排序方法了吧。

* 时间复杂度：O(n²)

## 图示

![](./asserts/selection.gif)

## 原始代码

```js
function selection(arr) {
  var minIndex;
  for (var i = 0; i < arr.length; i += 1) {
    minIndex = i;
    for (var j = i; j < arr.length - i; j += 1) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }

    var temp = arr[i];
    arr[i] = arr[minIndex];
    arr[minIndex] = temp;
  }
}
```

## 第一轮优化

如果最小数小标相同则不执行交换操作

```js
function selection(arr) {
  var minIndex;
  for (var i = 0; i < arr.length; i += 1) {
    minIndex = i;
    for (var j = i; j < arr.length - i; j += 1) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }

    if (minIndex !== i) {
      var temp = arr[i];
      arr[i] = arr[minIndex];
      arr[minIndex] = temp;
    }
  }
}
```