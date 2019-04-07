# 冒泡排序

比较两个相邻的项，如果第一个比第二个大，则交换他们。

* 时间复杂度：O(n²)

## 图示

![](./asserts/bubble.gif)

## 原始代码

```js
// 最原始的方法
function bubble(arr) {
  for (var i = 0; i < arr.length; i += 1) {
    for (var j = 0; j < arr.length; j += 1) {
      if (arr[j] > arr[j + 1]) {
        var temp = arr[j];
        arr[j] = arr[j + 1];
        arr[j + 1] = temp;
      }
    }
  }
}
```

## 第一次优化

当我们在比较相邻两项的时候，数组的最后一项其实是在和 `undefined` 进行比较，值永远是 `false`，所以我们可以直接从内循环减去这次操作。

```js
function bubble(arr) {
  for (var i = 0; i < arr.length; i += 1) {
    for (var j = 0; j < arr.length - 1; j += 1) {
      if (arr[j] > arr[j + 1]) {
        var temp = arr[j];
        arr[j] = arr[j + 1];
        arr[j + 1] = temp;
      }
    }
  }
}
```

## 第二次优化

在我们每一轮的比较结束后，这一轮的最后一项一定是本轮中的最大值，即为有序项，故可以在后序的内循环中减去本次操作。

```js
function bubble(arr) {
  for (var i = 0; i < arr.length; i += 1) {
    for (var j = 0; j < arr.length - 1 - i; j += 1) {
      if (arr[j] > arr[j + 1]) {
        var temp = arr[j];
        arr[j] = arr[j + 1];
        arr[j + 1] = temp;
      }
    }
  }
}
```


