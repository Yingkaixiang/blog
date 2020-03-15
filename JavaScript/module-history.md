# 【译】JavaScript 模块化的演变史

原文：https://github.com/myshov/history-of-javascript/tree/master/4_evolution_of_js_modularity

GitHub：https://github.com/Yingkaixiang/evolution-of-js-modularity

> 译者注：在翻译本篇文章的同时，笔者也会将自己的理解以及实践补充进去，试图让正在阅读本篇文章的同学能够更加容易的理解什么是 JavaScript 的模块。

当 Brendan Eich 设计第一个 JavaScript 版本时，他可能不知道在过去的20年里他的项目会如何发展。目前，该语言已经有6个主要的规范版本，改进工作仍在继续。

老实说，JavaScript 从来都不是完美的编程语言。模块化就是它的弱点之一，更确切的说这是它的缺失。确实，当脚本语言仅仅用于制作飘落的花的动画或者是用于表单验证时，当所有代码都可以在同一个全局作用域下存在和交互时，为什么你需要去关心代码和依赖的隔离？

随着 JavaScript 逐渐转变为一种通用语言，它开始被用于在各种环境（浏览器，移动设备，服务器，IoT）中构建复杂的应用程序。通过全局作用域进行程序组件交互的旧方法变得不可靠，因为代码量的增加往往会使您的应用程序过于脆弱。这就是为什么为了简化创建 JavaScript 应用程序而创建了各种模块化的实现。

本文是通过与 TC39 成员，不同框架开发人员进行交流，并阅读源代码，博客和书籍的结果，我们将研究以下方法/格式：命名空间，模块，独立依赖项定义，沙盒，依赖注入，CommonJS，AMD，UMD，标记模块，YModules，ES2015模块。同时，我们将恢复它们出现的历史背景。

## 我们需要解决的问题

在深入研究模块化世界之前，让我们仔细研究一下我们将尝试解决的问题。

### 命名冲突

从出现之时起，JavaScript 就使用全局对象 `window` 作为所有未使用 `var` 关键字定义的变量的存储。在1995-1999年间，这非常方便，因为 JavaScript 代码倾向于解决不需要很多代码量的小任务。但是当应用程序的代码量变大时，由于命名冲突，这个特性就会导致很多令人头疼的问题出现。让我们来看这个例子：

```js
// greeting.js 文件
var helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

function writeHello(lang) {
    document.write(helloInLang[lang]);
}

// hello.js 文件
function writeHello() {
    document.write('The script is broken');
}
```

当我们将脚本 `greeting.js` 放在页面上并在它后面加载 `hello.js` 时就会发生冲突。也就是说，在这种特殊情况下，我们将收到消息 `'The script is broken'`，而不是问候。

显然，在大型项目中，这可能会引起很多麻烦。此外，你不能确定页面上的第三方脚本不会破坏应用程序中的任何内容。

### 对于大型代码库的支持

> 笔者认为本段内容解释为依赖管理更合理。

JavaScript 在构建大型应用程序时的另一个不便之处是，需要在最常见的 ES5 浏览器环境中使用 `<script>` 标签明确指定插入的脚本。

如果你关心应用程序的代码是否可以被维护时，那么你需要将其分成独立的代码。因此，文件的数量可能非常大。对于大量文件，脚本的手动控制（即通过 `<script>` 标签将脚本放置在页面上）会变得非常让人厌烦，因为首先你必须记住在页面中放置哪些你需要的脚本，其次要保证脚本的加载顺序正确，从而使得你文件中所有的依赖关系都被解决。

## 直接定义依赖 (1999)

"直接定义依赖" 是第一个用于实现模块结构以及实现依赖项分离的模式。[Erik Arvidsson](https://github.com/arv)（目前为TC39成员）早在1999年便使用了该模式。

那时候，Erik 在一家初创公司工作，开发了一个可以在浏览器中运行 gui 应用程序的平台，它被称为WebOS（注意，这不是由 Palm 开发的 webOS）。WebOS 是一个拥有专利的平台，所以我没有办法去获得它的源代码。因此，我们看一下使用 [Dojo Toolkit](https://github.com/dojo/dojo) 实现此模式的方法，该工具包由 [Alex Russell](https://github.com/slightlyoff) 和 [Dylan Schiemann](https://github.com/dylans) 于2004年开发。

> 译者注：Dojo 本质上是各种库的集合，就像一个工具箱一样里面包含了，AJAX、DOM 操作、面向对象编程、事件、Promise 等等。而当前最新版本的 Dojo 已经遵循 AMD 规范实现了一个模块加载器。

"直接定义依赖" 的要点在于通过显式调用函数 `dojo.require`（也用于初始化加载的模块）来获取模块的代码（仅支持 Dojo 的 resource）。也就是说，在这种方法中，依赖关系是直接在应该使用它们的代码中被定义的。

> 译者注：
> 
> Dojo 会把每一个单独的 js 文件当做一个 resource，每当浏览器加载了一个 js 文件后，`dojo.provide()` 方法就会将其注册为一个模块。原理是每个 js 文件在顶部都必须有一个 `dojo.provide()` 方法的调用，并且传入的字符串参数是一个表示当前模块在 `window` 对象下的属性路径，举个例子：`dojo.provide('dojo.foo')` 表示程序会获取 `window.dojo.foo` 这个属性的值作为它的模块代码，而这个模块的名字为 `dojo.foo`。
>
> 我们会发现 Dojo 最初的设计思想和 CommonJS 还是有几分相似的 :)。
>
> 同时，我将 Dojo 1.6 版本源码中和模块相关的代码提取了出来，并做了相关的代码精简从而让同学们能够更加清晰的理解它在那个年代是如何实现模块化的。请查看本项目 `/directly-defined-dependencies` 文件夹下的源代码。
 
让我们使用 Dojo 1.6 来修改示例：

```js
// greeting.js 文件
dojo.provide("app.greeting");

app.greeting.helloInLang = {
  en: 'Hello world!',
  es: '¡Hola mundo!',
  ru: 'Привет мир!'
};

app.greeting.sayHello = function (lang) {
  return app.greeting.helloInLang[lang];
};

// hello.js 文件
dojo.provide("app.hello");

dojo.require('app.greeting');

app.hello = function(x) {
  document.write(app.greeting.sayHello('es'));
};
```

在这里，我们看到模块是使用函数 `dojo.provide` 来定义的，并且当你使用 `dojo.require` 时，便开始获取模块代码。这是 Dojo 1.7 版之前使用的一种相当简单的方法。到目前为止，[Google Closure Library](https://github.com/google/closure-library) 一直在使用它。

> 译者注：Google Closure Library 的基本功能和 Dojo 雷同，就不在这里说明了，大家有兴趣可以自行学习。但顺带一提 Goole Closure 是目前 tree shaking 效果最好的库。

## 命名空间模式 (2002)

为了解决命名冲突的问题，你可以使用特殊的代码约定。例如，你可以为所有变量和函数添加特定的前缀：`myApp_`: myApp_address, myApp_validateUser()。你还可以使用 JavaScript 中函数是一等公民的概念（一等公民：即你可以将它们分配给变量，对象的属性以及作为其他函数的返回值）。因此，你可以创建类似于 `document` 和 `window` 对象的方法属性（document.write()，window.alert()）。

ui 库 Bindows 是第一个使用这种模式的项目。Bindows 由我们已经很熟悉的 Erik Arvidsson 于2002年创建。他没有在函数和变量的名称中使用前缀，而是使用了一个全局对象，该对象的属性包含库的数据和逻辑。事实上它大大减少了全局代码的污染。该代码组织的模式现在被称为“命名空间”（命名空间模式）。

如果将这个想法应用到示例中，我们将得到如下结果：

```js
// app.js 文件
var app = {};

// greeting.js 文件
app.helloInLang = {
  en: 'Hello world!',
  es: '¡Hola mundo!',
  ru: 'Привет мир!'
};

// hello.js 文件
app.writeHello = function (lang) {
  document.write(app.helloInLang[lang]);
};
```

你可以看到，逻辑和数据现在位于对象 `app` 的属性中。因此，它们不会污染全局，而是继续从不同文件访问应用程序的各个部分。

命名空间模式可能是当今 JavaScript 中最著名的模式。Bindows 是第一个，但之后有很多其他框架和库以这种方式组织逻辑，例如 Dojo（2005），YUI（2005）。同时我们需要知道 Erik 并不认为自己是这种模式的创建者，但他不记得自己是受到什么特定项目的启发了。

## 模块模式 (2003)

命名空间为代码组织提供了某种方式。但是这显然还不够，因为还没有解决方案来隔离代码和数据。

解决此问题的先驱是模块模式。它的主要思想是用闭包封装数据和代码，并通过外部可访问的方法提供对它们的访问。这是这种模式的基本示例：

```js
var greeting = (function () {
  var module = {};

  var helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
  };

  module.getHello = function (lang) {
    return helloInLang[lang];
  };

  module.writeHello = function (lang) {
    document.write(module.getHello(lang))
  };
    
  return module;
}());
```

在这里，我们看到了立即执行函数（IIEF），该函数返回一个模块对象，该模块对象又具有方法 `getHello`，该方法通过闭包访问对象 `helloInLang`。 因此 `helloInLang` 变得无法从外部访问，并且我们获得了一段原子代码，可以将该代码粘贴到任何其他脚本中，而不会发生名称冲突。

这种方法首次被使用是在2003年，当时 Richard Cornford 在 comp.lang.javascript 组中提供了这种模式的[示例](https://groups.google.com/forum/#!msg/comp.lang.javascript/eTzWVa1W_pE/N9lnvRG9WJ8J)，以说明闭包的用法。在2005-2006年，Yahoo！开发了YUI框架。在 Douglas Crockford 的领导下，他们的项目采用了这种方法。但是，最大的推动力是 Douglas 在2008年提出的，当时他在他的书《JavaScript the Good Parts》中描述了“模块”。

同样，这里有一篇不错的文章 [JavaScript Module Pattern：In-Depth](http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html)。它描述了该模块的多种实现方式。建议大家阅读一下。

## 模板依赖定义 (2006)

“模板依赖定义” 是分离依赖定义家族中的下一个模式。这种模式最早在 [Prototype 1.4](https://github.com/myshov/history-of-javascript/blob/master/old_libs/prototype-1.4.0/src/prototype.js)（2006）被使用。但我怀疑该模式也被用于早期版本的库中。如果您可以访问早期版本的 Prototype，请告诉我）。

Prototype 的开发由 [Sam Stephenson](https://github.com/sstephenson) 于2005年开始。Prototype 是当时 Ruby on Rails 不可或缺的一部分。因为 Sam 在 ruby 上很有经验，所以在管理依赖项时选择了简单的 erb 模板就不足为奇了。

> 译者注：erb 模板就是我们熟悉的模板引擎，例如：基于 NodeJS 的 pug 以及 ejs 等。

如果我们尝试概括一下，可以说，该模式通过将特殊标签包含在目标文件中来定义依赖项。可以通过模板化（erb，jinja，smarty）和特殊的构建工具（例如borshik）来将标签解析为实际代码。与先前讨论的分离依赖定义模式相反，该模式仅适用于预构建步骤。

让我们使用这种依赖关系定义样式来转换示例。为此，我们将使用 borshik。

```js
// app.tmp.js 文件

/*borschik:include:../lib/main.js*/

/*borschik:include:../lib/helloInLang.js*/

/*borschik:include:../lib/writeHello.js*/

// main.js 文件
var app = {};

// helloInLang.js 文件
app.helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

// file writeHello.js
app.writeHello = function (lang) {
    document.write(app.helloInLang[lang]);
};
```

在示例文件app.tmp.js中定义了插入的脚本及其顺序。如果你考虑一下这个示例，那么很明显，这种方法不会从根本上改变开发人员的开发方式。你只不过通过使用其他标签来代替 `<script>` 标签。因此，我们仍然会忘记某些脚本或弄乱被插入脚本的顺序。所以，此方法的主要目的将从多个文件创建成一个单一文件。

## 注释依赖定义 (2006)

“注释依赖定义” 也是分离依赖关系定义的子类型。它与 “直接定义依赖” 非常相似，但是在这种模式下，我们不使用某种编程方法，而是使用注释，其中包括有关特定模块所有依赖项的信息。

使用此模式的应用程序必须是预先构建的（该方法在2006年用于 [Valerio Proietti](https://github.com/kamicane) 创建的 [MooTools](https://github.com/mootools/mootools-core/blob/41b0bdedce3adeb921c181145d7c79a8ecbf4763/Plugins/Fxpack.js#L12)），或者必须动态解析下载的代码并在运行时解析依赖项。最后一种方法是由 NicolásBevacqua 创建的 [LazyJS](https://github.com/bevacqua/lazyjs) 中使用的。

如果使用此库重写我们的示例将如下所示：

```js
// helloInLang.js 文件
var helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

// sayHello.js 文件

/*! lazy require scripts/app/helloInLang.js */

function sayHello(lang) {
  return helloInLang[lang];
}

// hello.js 文件

/*! lazy require scripts/app/sayHello.js */

document.write(sayHello('en'));
```

我们来看看这个库时如何工作的。库下载文件时，它会分析其内容，找到相关的加载模块注释，并下载它们，然后重复之前的操作。

> 译者注：如同递归一般的加载解析文件，直到全部文件被解析完毕。

使用此方法最著名的库是 MooTools。 LazyJS 是一个有趣的实验，但是由于它的出现是在 CommonJS 和 AMD 之后发生的，因此 LazyJS 并没有引起开发人员的广泛关注。

## 外部依赖定义 (2007)

让我们看一下 “分离依赖定义” 系列中的最后一个模式。在外部定义的依赖关系模式中，所有依赖关系都在主要上下文之外定义，例如在配置文件中或在代码中作为对象或具有依赖关系列表的数组。但是，有一个准备阶段。在此阶段中，应用程序将以正确的顺序加载所有依赖项进行初始化。

我设法找到的最早使用这种方法的日期是2007年在MooTools 1.1中。

在最简单的情况下，可以像这样完成使用这种模式的示例（对于本示例，我将使用自己的使用该模式的实验性加载程序）。

```js
// file deps.json
{
    "files": {
        "main.js": ["sayHello.js"],
        "sayHello.js": ["helloInLang.js"],
        "helloInLang.js": []
    }
}

// file helloInLang.js
var helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

// file sayHello.js
function sayHello(lang) {
    return helloInLang[lang];
}

// file main.js
console.log(sayHello('en'));
```

文件deps.json是定义所有依赖关系的外部上下文。当您运行应用程序时，加载程序将接收到此文件，读取定义为数组的所有依赖项的列表，然后以正确的顺序加载并将它们放入页面。

如今，这种方法已在库中用于创建自定义版本。 例如，lodash 正在使用这种方法。

## 沙盒模式 (2009)

Yahoo的开发人员致力于YUI3中的新模块系统，他们正在解决在一页上使用不同版本的库的问题。该框架的先前的YUI3模块系统已经使用模块模式和命名空间的组合来实现。显然，通过这种方法，包含库代码的顶级对象只能是一个，因此同时使用多个版本的库确实很困难。

Adam Moore（YUI3的开发人员之一）建议使用“沙盒”解决此问题。使用此模式的模块化的简单实现如下所示：

```js
// file sandbox.js
function Sandbox(callback) {
    var modules = [];

    for (var i in Sandbox.modules) {
        modules.push(i);
    }

    for (var i = 0; i < modules.length; i++) {
        this[modules[i]] = Sandbox.modules[modules[i]]();
    }
    
    callback(this);
}

// file greeting.js
Sandbox.modules = Sandbox.modules || {};

Sandbox.modules.greeting = function () {
    var helloInLang = {
        en: 'Hello world!',
        es: '¡Hola mundo!',
        ru: 'Привет мир!'
    };

    return {
        sayHello: function (lang) {
            return helloInLang[lang];
        }
    };
};

// file app.js
new Sandbox(function(box) {
    document.write(box.greeting.sayHello('es'));
});
```

这种方法的本质在于，您可以使用全局构造函数来代替全局对象。可以将模块定义为此构造函数的属性。

“沙盒”是解决模块化问题的有趣方法，但是除了YUI3之外，这种模式并未引起人们的广泛关注。如果您想了解更多有关Sandbox的信息，建议您阅读Javascript Sandbox Pattern文章，以及有关创建该库的新模块的YUI官方文档。

## 依赖注入 (2009)

2004年，Martin Fowler引入了“依赖注入”（DI）概念，用于描述Java中组件通信的新机制。 要点是所有依赖项都来自组件外部，因此组件不负责初始化其依赖项，它仅使用它们。

五年后，Sun和Adobe的前雇员MiškoHevery（他主要从事Java开发）开始为他的创业公司设计一个新的JavaScript框架，该框架使用依赖注入作为组件之间通信的关键机制。该商业想法尚未证明其有效性，因此该框架的源代码已在其初创公司getangular.com的领域中开放并引入了全世界。我们都知道接下来会发生什么。谷歌接管了Miško及其项目，现在Angular是最著名的JavaScript框架之一。

Angular中的模块是通过依赖注入机制实现的。 顺便说一下，模块化不是DI的主要目的，关于这一点，Miško在回答相应问题时也明确表示。

为了说明这种方法，让我们使用Angular的第一个版本重写示例（是的，请记住该示例是非常综合的）：

```js
// file greeting.js
angular.module('greeter', [])
    .value('greeting', {
        helloInLang: {
            en: 'Hello world!',
            es: '¡Hola mundo!',
            ru: 'Привет мир!'
        },

        sayHello: function(lang) {
            return this.helloInLang[lang];
        }
    });

// file app.js
angular.module('app', ['greeter'])
    .controller('GreetingController', ['$scope', 'greeting', function($scope, greeting) {
        $scope.phrase = greeting.sayHello('en');
    }]);
```

如果您将在浏览器中打开带有此示例的页面，那么代码将神奇地运行，并且您将在页面上看到结果。

目前，依赖注入是Angular 2和Slot这样的框架中的关键机制。 还有大量的库可简化这种方法在不依赖任何框架的应用程序中的使用。

## CommonJS 模块 (2009)

甚至在Node.JS之前，还有客户端JavaScript引擎（在浏览器中）以及用于JavaScript的主要语言的服务器端开发平台。 由于缺少适当的规范，服务器解决方案未提供用于与操作系统及其环境（文件系统，网络，环境变量等）进行通信的统一API，因此在代码分发方面产生了问题。 例如，为旧的Netscape Enterprise Server编写的脚本在Rhino中不起作用，反之亦然。

转折点发生在2009年，当时Mozilla的Kevin Kevin Dangoor的一名员工发表了一篇有关服务器端JavaScript问题的帖子，并呼吁所有有兴趣的人加入一个非正式委员会，讨论和开发称为ServerJS的服务器端JavaScript API。 半年后，由于新概念开始成为讨论的一部分，ServerJS重命名为CommonJS。

工作开始沸腾了。 开发人员和研究人员最为关注的是模块的规范-CommonJS模块（有时称为CJS或仅CommonJS），该规范最终在Node.JS中实现。

作为CommonJS模块的示例，让我们通过以下方式修改我们的模块：

```js
// file greeting.js
var helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

var sayHello = function (lang) {
    return helloInLang[lang];
}

module.exports.sayHello = sayHello;

// file hello.js
var sayHello = require('./lib/greeting').sayHello;
var phrase = sayHello('en');
console.log(phrase);
```

在这里，我们看到了实现模块化的两个新实体-require和module，它们提供了加载模块并将其接口导出到外部世界的能力。 值得注意的是，既不是require语言也不是module语言的某种关键字。 在Node.JS中，由于辅助功能，我们可以使用它们。 此函数先包装每个模块，然后再将其发送到JavaScript引擎：

```js
(function (exports, require, module, __filename, __dirname) {
    // ...
    // Your code injects here!
    // ...
});
```

CommonJS规范仅定义了不同环境中模块互操作性所需的最低要求。 这意味着CommonJS已为扩展打开。 例如，Node.JS通过将属性main添加到require函数来使用此功能，如果直接执行组成该模块的文件，则指向该模块。

Babel还通过默认导出扩展了ES2015模块的编译过程中的需求（我将在本文末尾讨论该模块系统）：

```js
export default something;
```

Babel将此类导出转换为CommonJS模块，在该模块中，默认值与相应的属性一起导出。 简而言之，您可以通过转译获得如下内容：

```js
export.default = something;
```

打包工具Webpack还使用了各种扩展，例如require.ensure，require.cache，require.context，但是它们的讨论不在本文的讨论范围之内。

CommonJS是目前最常用的模块格式。 您不仅可以在Node.JS的服务器端使用它，还可以使用Browserfiy或Webpack在客户端使用它，它们可以将一组CommonJS模块转换为一个捆绑包。

## AMD (2009)

CommonJS规范的工作已经全面展开，与此同时，邮件列表中也进行了有关将规范异步加载模块的可能性的讨论。 主要动机在于以下事实：这将有助于加快Web应用程序的加载速度，而无需进行任何预先捆绑。

Kevin的另一位Mozilla开发人员James Burke的同事是所有讨论中最活跃的异步模块化捍卫者之一。 那时的James可能是专家，因为他是Dojo Framework 1.7中异步模块化系统的作者，并且自2009年以来他一直在开发加载器require.js。

James试图阐明的基本思想是，模块的加载不应该是同步的（即，模块一个接一个地加载）； 我们必须使用浏览器功能来并行加载脚本。 为了实现所有要求，James提出了自己的格式，称为AMD（异步模块定义）。

如果我们将重写我们的例子使用AMD格式，我们会得到这样的东西

```js
// file lib/greeting.js
define(function() {
    var helloInLang = {
        en: 'Hello world!',
        es: '¡Hola mundo!',
        ru: 'Привет мир!'
    };

    return {
        sayHello: function (lang) {
            return helloInLang[lang];
        }
    };
});

// file hello.js
define(['./lib/greeting'], function(greeting) {
    var phrase = greeting.sayHello('en');
    document.write(phrase);
});
```

文件hello.js是程序的入口。 在此文件中，有一个定义模块的函数定义。 该函数的第一个参数是一组依赖项。 只有在加载了该模块的所有依赖项之后，才启动在define的第二个参数中定义为函数的模块代码的执行。 模块的这种延迟代码执行使并行加载其依赖项成为可能。

2011年，当詹姆斯宣布创建一个单独的邮件列表来协调AMD的所有工作时，所有讨论都出现了转折，因为在这段时间内一直未与CommonJS小组达成共识。

通过个人观察，我可以说AMD仍然与客户端应用程序的开发有关，但是通过npm分发客户端库的趋势越来越使开发人员远离AMD。

## UMD (2011)

模块格式的明显对抗甚至在AMD与CommonJS Modules分离之前就已经开始。 那时，AMD阵营中已有许多开发人员，他们喜欢使用模块化代码的最低入门门槛。 由于Node.JS的日益普及以及Browserify的出现，CommonJS Modules的支持者数量也迅速增长。

因此，存在两种格式，它们无法相互配合。 未经代码修改，不得在实现CommonJS模块规范的环境中使用AMD模块。 CommonJS模块也不能与以AMD为主要格式（require.js，curl.js）的加载器一起使用。 对于整个JavaScript生态系统来说，这是一个糟糕的情况。

已经开发了UMD格式来解决此问题。 UMD代表“通用模块定义”，因此该格式允许您在AMD工具以及CommonJS环境中使用同一模块。

很难找到这种格式的原始作者，因此我不得不进行调查。 首先，我转到GitHub Addy Osmani上的UMD存储库的作者，他又向我介绍了James Burke和Kris Kowal。 这些人向Q的存储库指出了JavaScript中诺言的第一个实现。

自创建以来，Q库在不同的环境中工作：在浏览器中（通过脚本标记将模块放在页面上时），在Node.JS和Narwhal的服务器端（CommonJS模块）中。 一段时间之后，James将AMD的支持添加到了Q中。然后Addy在一个名为UMD的单一存储库中收集了类似的模式。 这种针对不同模块系统的代码适配的结果现在称为UMD。

作为示例，让我们重构我们的模块greeting.js以同时支持不同环境CommonJS和AMD：

```js
(function(define) {
    define(function () {
        var helloInLang = {
            en: 'Hello world!',
            es: '¡Hola mundo!',
            ru: 'Привет мир!'
        };

        return {
            sayHello: function (lang) {
                return helloInLang[lang];
            }
        };
    });
}(
    typeof module === 'object' && module.exports && typeof define !== 'function' ?
    function (factory) { module.exports = factory(); } :
    define
));
```

此实现模式的核心是立即调用的函数表达式。 该函数根据环境采用不同的参数。 如果将代码用作CommonJS模块，则传递的参数是以下函数：

```js
function (factory) {
    module.exports = factory();
} 
```

如果将代码用作AMD模块，则function参数为define。 由于此替换，因此代码可以在不同的环境中使用。

现在，UMD是大多数开发人员在需要能够在浏览器或Node.JS中使用其模块时使用的一种格式。 许多流行的库都支持导出为UMD格式，例如moment.js和lodash。

## Labeled Modules (2012)

自2010年以来，TC39委员会开始致力于新的JavaScript本机模块系统的开发，该系统当时被称为ES6模块。 到2012年，很明显它将采用哪种最终外观。 塞巴斯蒂安·马克博格（SebastianMarkbåge）委员会的一名成员（目前也是React的首席开发人员）主动提出了可传递模块格式。 假定即使在ES3环境中也可以使用这种格式，然后可以轻松地将其用于新的模块系统。 此格式称为标签模块。

这种格式的主要思想在于使用标签。 关键字“导入”和“导出”是该语言的保留字，因此不能用于标签。 因此，为此目的采用了相应的同义词。 标签“ exports”用于导出，标签“ require”用于导入。

与往常一样，让我们重新制作示例以展示这种格式的实际效果。

```js
// file greeting.js
var helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

exports: var greeting = {
    sayHello: function (lang) {
        return helloInLang[lang];
    }
};

// file hello.js
require: './lib/greeting';
var phrase = greeting.sayHello('es');
document.write(phrase);
```

您可以在此处获取使用标签模块构建应用程序的配置示例。

如我们所见，这是一个非常优雅的解决方案。 但是由于事实是2012年CommonJS和AMD对于大多数开发人员来说是一个非常受欢迎的选择，因此新格式无法克服这场激烈的竞争。 尽管此格式的支持已出现在webpack的第一个版本中，但该格式在JavaScript社区中并未引起太多关注。

## YModules (2013)

YModules是在Yandex上创建的模块系统，用于解决CommonJS和AMD无法解决的任务。 此模块系统有主要要求。 首先是尽可能透明地使用具有异步特性的模块，其次是重新定义模块的可能性。 首先是实现异步API的重要要求，例如Yandex.Maps，其次是重要的，因为需要在BEM中的定义级别使用模块。

Yandex.Maps和BEM的团队在2013年制定了新模块系统的规范。然后由Dmitry Filatov实施。

这是使用YModules的示例的实现：

```js
// file greeting.js
modules.define('greeting', function(provide) {
    provide({
        helloInLang: {
            en: 'Hello world!',
            es: '¡Hola mundo!',
            ru: 'Привет мир!'
        },

        sayHello: function (lang) {
            return this.helloInLang[lang];
        }
    });
});

// file app.js
modules.require(['greeting'], function(greeting) {
    document.write(greeting.sayHello('ru'));
});
// Result: "Привет мир!"
```

YModules在结构上与AMD非常相似，但是YModules的主要区别是通过特殊功能提供模块向消费者公开模块接口，而不是像AMD那样具有回报。

此功能使您能够从异步代码的块中进行提供，也就是说，它允许对外界隐藏模块的异步性质。 例如，如果我们将一些异步逻辑（例如setTimeout）添加到greeting.js中，则使用此模块的整个代码将保持不变：

```js
// file greeting.js
modules.define('greeting', function(provide) {
    // postpone of code execution for 1 second
    setTimeout(function () {
        provide({
            helloInLang: {
                en: 'Hello world!',
                es: '¡Hola mundo!',
                ru: 'Привет мир!'
            },

            sayHello: function (lang) {
                return this.helloInLang[lang];
            }
        });
    }, 1000);
});

// file: app.js
modules.require(['greeting'], function(greeting) {
    document.write(greeting.sayHello('ru'));
});
// result: "Привет мир!"
```

如前所述，YModules的主要特点是它可以与BEM的各级定义一起使用。让我们看看如何使用这个特性。

```js
// file moduleOnLevel1.js
modules.define('greeting', function(provide) {
    provide({
        helloInLang: {
            en: 'Hello world!',
            es: '¡Hola mundo!',
            ru: 'Привет мир!'
        },

        sayHello: function (lang) {
            return this.helloInLang[lang];
        }
    });
});

// file moduleOnLevel2.js
modules.define('greeting', function(provide, module) {
    // redeclaring of sayHello method
    module.sayHello = function (lang) {
        return module.helloInLang[lang].toUpperCase();
    };
    provide(module);
});

// file app.js
modules.require(['greeting'], function(greeting) {
    document.write(greeting.sayHello('ru'));
});
// Result: "ПРИВЕТ МИР!"
```

如果运行此示例，则作为重新定义greeting模块的结果，“ sayHello”方法将更改为新方法，并且输出消息的文本将转换为大写。 这是可能的，因为在YModules中再次定义模块时，其最后一个参数将包含模块的先前版本。

目前，YModules已在Yandex的各种项目中使用。 它也是框架i-bem.js中的主要模块系统。

## ES2015 Modules (2015)

当然，TC39委员会正在观察JavaScript世界中正在发生的事情。 很明显，现在是时候对该语言进行重大更改了。

模块化系统的工作始于2010年。该系统的设计由Dave Herman和Sam Tobin-Hochstadt设计。 这项工作持续了五年。 模块系统的最终设计已随规范ES2015一起发布。

按照传统，让我们修改示例以展示实际的规范：

```js
// file lib/greeting.js
const helloInLang = {
    en: 'Hello world!',
    es: '¡Hola mundo!',
    ru: 'Привет мир!'
};

export const greeting = {
    sayHello: function (lang) {
        return helloInLang[lang];
    }
};

// file hello.js
import { greeting } from "./lib/greeting";
const phrase = greeting.sayHello("en");
document.write(phrase);
```

如我们所见，该标准引入了全新的关键字，这些关键字用于使用关键字import导入模块，以及使用关键字export导出代码。

由于我们正在使用该语言处理新的关键字，并且由于负责在各种环境中支持加载模块的Module Loader API规范尚未准备就绪，因此我们不能仅仅选择并开始使用 新的本机模块系统。

尽管有这些限制，但是许多项目已经开始使用模块的新格式。 要在ES5最普遍的世界中开始使用新标准，可以使用Babel转换，这是相当普遍的做法。

## 结论

JS中还有其他模块化方法。 它们中的一些可以相互交织在一起，形成奇异的形式，其他的则专门为在特定项目中使用而创建，而另一些则被创建为可传递的格式。 描述它们都是一项不平凡的任务，因此本文仅讨论了最流行的方法和格式。 尽管如此，我认为这篇文章帮助您学习了一些新知识，使有关JavaScript模块化的知识系统化，并进一步了解了支持所有这些技术的那些人。

> 转载请注明出处或者私信评论我哦~
