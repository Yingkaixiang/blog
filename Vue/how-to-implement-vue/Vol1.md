# 【原创】手把手教你实现一个 Vue.js

**笔者注：本教程的目的是通过使用最简单的场景和逻辑来让大家理解什么是 `MVVM` 模式的，而不是去通过复制某个框架代码来解释它到底是如何运作。笔者一直以为提升自己编程能力最好的方法是授人以渔而非授人以鱼。**



大家都知道目前社区里流行的前端框架核心都是依赖于 `MVVM` 模式的，简单说就是数据驱动，一旦我们更改了源数据，相对应的视图就会自动更新，而不像从前那样需要开发者手动去操作 `DOM` 如此繁琐。

不论是 `React`，`Vue` 还是 `Angular` 本质上只是实现 `MVVM` 的方式不同，今天笔者就带着你一步一步实现一个超简单的 `MVVM` 模式。

## 实现目标

既然是数据驱动，我们核心要解决的问题当然是如何将视图以及数据绑定在一起，形成一个动态的变化。对于 web 技术来说视图就是 `HTML` 数据就是 `JavaScript`。为了贯彻**极简**这个思想，我们暂时就不实现双向绑定了，简单说我们今天要实现的是通过数据变化（如：进行 js 赋值操作）引起视图变化（如：html 更新）。

如下代码所示，我们的目标是实现和 `Vue` 相同的数据绑定和渲染逻辑。

1. 页面默认显示两行文案 “掘金” 和 “juejin.im”。
2. 当我们修改了数据源后页面会自动更新成 “谷歌” 和 “google.com”。

```html
<div id="app">
  <div>{{ name }}</div>
  <div>{{ host }}</div>
</div>

<script>
    // 默认展示数据
    const data = { name: "掘金", host: "juejin.im"  };
    // 修改数据，页面会更新成新的内容
    data.name = "谷歌";
    data.host = "google.com";
</script>
```

OK，让我们开始我们的代码编写~

## 1. 实现响应式数据

既然我们在前面给自己定的目标是通过数据变化引起视图变化，所以首先我们得把数据变成可以监听的，当它被取值或者赋值时我们就可以执行一些自定义的逻辑了。这时候我们想到了 `Object.defineProperty` 方法，它可以给指定对象的 `key` 设定 `getter`，`setter` 方法这样我们就可以达到监听的目的了。

```js
// 通过遍历数据源给每一个 key 加上监听事件
// 因为是超极简所以暂不考虑引用对象的场景并且对象只有一层
Object.keys(data).forEach((key) => {
    reactive(data, key);
});

// 使得我们的数据源成为响应式的方法
function reactive(obj, key) {
    var value = obj[key];
    Object.defineProperty(obj, key, {
        get() {
            console.log(`获取到的数据为：${value}`);
            return value;
        },
        set(newVal) {
            console.log(`将数据设置为：${newVal}`);
            value = newVal;
        }
    })
}

data.name; // console: 获取到的数据为：掘金
data.name = "谷歌"; // console: 将数据设置为：谷歌
```

到这里我们就可以实现了对于 `data` 对象的控制了。在真实场景中我们需要通过递归的方式将所有的 `key` 都变成响应式。同时这个对象必须是一个纯对象。

## 2. 实现模板引擎

我们知道原生 `HTML` 是不支持 `{{ name }}` 这样的语法的，如果我们不进行任何操作则页面上显示的就是 `{{ name }}`，所以我们现在要实现模板引擎的功能，让本来在 `HTML` 中作为字符串解析的文字变成一个可以让我们在 `JavaScript` 中进行控制的变量。

我们可以通过 `DOM` 操作拿到当前页面的节点信息。`<div id="app"></div>` 就是我们的入口节点。

```js
// 首先获取我们的入口节点
// 只有入口节点下的所有子节点才能使用 mvvm 模式进行视图更新
// 因为它们经过一些逻辑处理后可以受我们 js 代码控制的
var root = document.querySelector('#app');

// 接着我们创建一个文档片段
// 将 #app 节点下的所有子节点都放进去
// 以便我们可以对每个节点进行控制
var fragment = document.createDocumentFragment();
var node = root.firstChild;

while(node) {
    // 一个关于 appendChild 的小知识点
    // 当被插入的节点是文档中已存在的节点时
    // appendChild 会将这个节点从原来的位置删除
    // 然后再插入到新指定的位置
    // 这时 firstChild 属性会指向下一个节点
    // 这样就可以通过循环的方式拿到所有的节点了
    fragment.appendChild(node);
    node = root.firstChild;
}
// 当你执行完这段代码时你会发现页面上的内容都不见了
// 因为它们都在 fragment 里等待你的处理
// 直到你执行 root.appendChild(fragment) 后
// 页面上才会展示被你处理以后的节点信息
```

通过以上的代码现在所有 `<div id="app"></div>` 下的子节点都已经在我们的掌控之中了，我们现在要做的就是将模板引擎的语法给解析出来。因为模板引擎的占位符和我们数据中的 `key` 是一一对应的，这就是视图与数据之间的联系。我们要想办法实现这种联系。

```js
// 用于存放全部的 updater 方法，先无视
const updaters = {};
// 遍历文档片段中的所有子节点
for (let i = 0; i < fragment.childNodes.length; i += 1) {
    const node = fragment.childNodes[i];
    // 为了极简我们只处理元素节点，如 <div>
    if (node.nodeType === 1) {
        // 获取节点的文字内容
        // 此处对应的就是 {{ name }}
        const text = node.textContent;
        // 通过正则获取占位符
        // 此处对应的就是 name
        const key = text.match(/\{\{(.*)\}\}/)[1].trim();
        
        // 重点来了！！！
        // 此时我们需要为这个节点创建一个 updater 方法
        // 它被用来描述当前节点是如何更新的
        // 只要这个节点对应的数据发生了变化
        // 我们就可以通过调用 updater 方法去更新对应的节点数据
        const updater = function() {
            // 通过占位符的 key 来获取 data 里的 value
            // 这就是我们之前说的联系
            node.textContent = data[key];
        }
        
        // 既然有了这个节点的更新方法
        // 那我们就需要把这些方法暴露出来
        // 这样才能使得对应数据在被赋值时可以调用它
        // 还记得我们一开始设置的 updaters 全局变量吗
        // 它被用来存放所有的更新函数
        updaters[key] = updater;
        // 最后我们执行下 updater 方法
        // 将指定节点的内容替换成 data 的值
        // 否则页面上还是只会显示 {{ name }}
        // 这一步其实是对于当前节点的初始化
        updater();
    }
}

// 最后将我们设置完的文档片段重新插入到根节点
// 记得将前面的 data.name data.name = "谷歌" 两行代码删除
// 页面上就会显示出 掘金 和 juejin.im 的文字了
root.appendChild(fragment);
```

## 3. 将模板与数据进行绑定

终于到了最后一步了，现在我们已经有了响应式的数据和对应节点的更新方法了，我们只要在源数据每个 `key` 的 `setter` 方法中调用对应的更新函数就可以实现我们的 MVVM 模式了。

```js
function reactive(obj, key) {
    var value = obj[key];
    Object.defineProperty(obj, key, {
        get() {
            console.log(`获取到的数据为：${value}`);
            return value;
        },
        set(newVal) {
            console.log(`将数据设置为：${newVal}`);
            value = newVal;
            // 将数据对应的更新函数放在这里
            // 就可以实现当数据变化时去更新视图
            updaters[key]();
        }
    })
}
```

至此我们这个极简的 `MVVM` 模式就实现了，让我们在代码底部插入这两行代码来测试是不是已经成功了。

```js
data.name = "谷歌";
data.host = "google.com";
```

果然页面上显示了 谷歌 和 google.com 大功告成！

## 总结

同学们在看完本教程后千万不要把它当做是 Vue 的源代码，因为它们毫无关系，笔者只是为了让大家能够在最简单的代码逻辑里尽可能地了解它的实现原理，所以并没有添加很多兼容性或者出于性能考虑的代码。目的就是为了拨开现今最流行框架的神秘面纱。同时笔者也想将他继续深入的迭代下去，并且添加更多的功能特性，毕竟罗马不是一天造就，框架也不是一个 commit 就能完成的，它也是一步步的不断优化和迭代才变的如此优秀。