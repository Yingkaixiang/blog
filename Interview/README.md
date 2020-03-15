# 面试问题整理

## 一、你对前端安全有了解吗？请简述几个安全问题以及相关的解决方案

### XSS 攻击

#### 介绍

##### 1. 反射型XSS

发请求时，XSS代码出现在URL中，提交给服务端。服务端返回的内容，也带上了这段XSS代码。最后浏览器执行XSS代码。

##### 2. 存储型XSS

存储型和反射型的区别就是，提交的XSS代码会存储在服务器端。如果这个过程中没有经过任何转义，那么这段html就直接执行了。这样，所有访问你个人主页的用户，就都中招了。

##### 3. DOM XSS

eval不安全的原因。

```js
// 这个时候，如果用户在网址后面加上恶意代码
// http://www.xss.com#alert(document.cookie)
eval(location.hash.substr(1));
```

#### 防御手段

##### 1. 过滤转义输入输出

* 后端对于数据类型进行检查，比如数字必须为 number。
* 对于特殊符号进行转义，无论是前端还是后端都需要进行这个操作

```
& --> &amp;
< --> &lt;
> --> &gt;
" --> &quot;
' --> &#x27;
/ --> &#x2F;
```

* 避免使用eval，new Function等执行字符串的方法，除非确定字符串和用户输入无关
* cookie 尽量开启 http only 属性

#### 自动化检测 XSS 漏洞

1. 字符串检测 https://github.com/0xsobky/HackVault/wiki/Unleashing-an-Ultimate-XSS-Polyglot

2. Arachni、Mozilla HTTP Observatory、w3af 自动检测工具

### CSRF

#### 介绍

跨站请求伪造。通过 web 端的一些可以进行跨域请求的方式引导用户去发起请求，请求的同时会带上当前用户的cookie数据来实现伪造登录

### iframe

第三方的 iframe 可以运行脚本，弹框等恶意行为

#### 解决方法

使用 iframe 的 sandbox 属性

#### 解决方案

1. 检测http referer是否是同域名，通常来讲，用户提交的请求，referer应该是来来自站内地址，所以如果发现referer中地址异常，那么很可能是遭到了CSRF攻击。

2. 避免登录的session长时间存储在客户端中。

3. 关键请求使用验证码或者token机制

***

## 二、你了解 JS 的自定义事件是什么吗？它有哪些使用场景？

自定义事件相当于是观察者模式，可以把复杂逻辑解耦，代码可以写的很清晰，而且很容易复用。

## 请简述下数据库索引的实现方式

将大量的数据变成一块的一块的数据然后快速排除无关的数据，从而提高查询的速度

***

## 三、请简述下 debounce throttle 的实现原理

### debounce

上一次被调用后，延迟 n 秒后调用 x 方法

```js
function debounce(fn, wait, options) {   
    wait = wait || 0;   
    var timerId;    

    function debounced() {   
        if (timerId) {   
            clearTimeout(timerId);

            timerId = null;   
        }   
        timerId = setTimeout(function() {   
            fn();     
      }, wait);
   }
    return debounced;
 }
```

### throttle

在 n 秒内最多执行 fn 一次的函数。

```js
function throttle(fn, wait, options) {   
    wait = wait || 0;   
    var timerId, lastTime = 0;    
    
    function throttled() {   
        var currentTime = new Date();   
        if (currentTime >= lastTime + wait) {   
            fn();   
            lastTime = currentTime;   
        } else {   
            if (timerId) {   
                clearTimeout(timerId);   
                timerId = null;   
            }   
            timerId = setTimeout(function() {   
                fn()   
            }, wait);   
       }   
    }    
    return throttled;  
}
```

***

## 四、同样作为继承的一种方式请 mixin extend 有什么区别？

### mixin

优点：可以传递参数更灵活。

缺点：编译后不能产生 dry css 式的代码，也就是有重复代码。

```css
@mixin button { background-color: green; }

.button-1 { @include button; }

.button-2 { @include button; }

/* 编译后 */
.button { background-color: green; }

.button-1 { background-color: green; }

.button-2 { background-color: green; }
```

### extend

优点：编译后能产生 dry css 式的代码。

缺点：不够灵活，还会增加选择器之间的关系可能有副作用。

```css
.button { background: green; }

.button-1 { @extend .button; }

.button-2 { @extend .button; }

/* 编译后 */
.button,

.button-1,

.button-2 { background: green; }
```

***

## 五、什么是 babel-helper ？

用于转移语法的函数集合，并提供相关操作方法。

***

## 六、请实现一个在线表格同步的功能

### 通讯

* 长连接
* websocket

### 编辑冲突

#### 编辑锁

同一块文档同时只能有一个人编辑，用户体验较差。

#### diff

加载页面以后获取一个文档副本，本地编辑时进行 diff 并将结果发送给服务器，服务器保存后再将结果返回给其他正在编辑的用户，在进行 diff 然后更新。

### OT

通过 `编辑距离` 算法获取改变文本的步骤，然后通过 OT 算法将不同用户的步骤合并。

***

## 七、前端监控

### 性能

自研：通过 performace api 进行性能数据上报，上报请求是 1px * 1px gif 图片（[原因](https://toutiao.io/posts/xpy6p8/preview)）。在 `onload` 事件中进行上报，为了不影响页面性能使用 `requestIdleCallback` 方法等待主线程空闲在运行。定义合适的指标 ![](https://static.geekbang.org/infoq/5c6baefb99cda.png)

工具：lighthouse

### 埋点

* 手动买点
* 无埋点

### 错误上报

* sentry

***

* canvas 确定文字宽高
* 弹幕实现
* spa 用户操作路径记录如何实现
* ts es6 区别
* 怎么从一个 js 项目迁移到 ts
* webpack loader plugin 区别
* react 组件性能优化
* 基本类型 引用类型的内存分配方式
* JSbridge 实现方式
* 秒杀系统实现方式