---
title: Web Components
category: htmlapi
layout: page
date: 2014-07-09
modifiedOn: 2014-07-09
---

## 概述

各种网站往往需要一些相同的模块，比如日历、调色板等等，这种模块就被称为“组件”（component）。Web Component就是网页组件式开发的技术规范。

采用组件进行网站开发，有很多优点。

（1）管理和使用非常容易。加载或卸载组件，只要添加或删除一行代码就可以了。

```html

<link rel="import" href="my-dialog.htm">

<my-dialog heading="A Dialog">Lorem ipsum</my-dialog>

```

上面代码加载了一个对话框组件。

（2）定制非常容易。组件往往留出接口，供使用者设置常见属性，比如上面代码的heading属性，就是用来设置对话框的标题。

（3）组件是模块化编程思想的体现，非常有利于代码的重用。标准格式的模块，可以跨平台、跨框架使用，构建、部署和与其他UI元素互动都有统一做法。

（4）组件提供了HTML、CSS、JavaScript封装的方法，实现了与同一页面上其他代码的隔离。

未来的网站开发，可以像搭积木一样，把组件合在一起，就组成了一个网站。这是非常诱人的。

Web Components不是单一的规范，而是一系列的技术组成，包括Template、Custom Element、Shadow DOM、HTML Import四种技术规范。使用时，并不一定这四者都要用到。其中，Custom Element和Shadow DOM最重要，Template和HTML Import只起到辅助作用。

## template标签

template标签表示网页中某些重复出现的部分的代码模板。它存在于DOM之中，但是在页面中不可见。

下面的代码用来检查，浏览器是否支持template标签。

```javascript

function supportsTemplate() {
  return 'content' in document.createElement('template');
}

if (supportsTemplate()) {
  // 支持
} else {
  // 不支持
}

```

下面是一个模板的例子。

```html

<template id="profileTemplate">
  <div class="profile">
    <img src="" class="profile__img">
    <div class="profile__name"></div>
    <div class="profile__social"></div>
  </div>
</template>

```

使用的时候，需要用JavaScript在模板中插入内容，然后将其插入DOM。

```javascript

var template = document.querySelector('#profileTemplate');
template.querySelector('.profile__img').src = 'profile.jpg';
template.querySelector('.profile__name').textContent = 'Barack Obama';
template.querySelector('.profile__social').textContent = 'Follow me on Twitter';
document.body.appendChild(template.content);

```

上面的代码是将模板直接插入DOM，更好的做法是克隆template节点，然后将克隆的节点插入DOM。这样做可以多次使用模板。

```javascript

var clone = document.importNode(template.content, true);
document.body.appendChild(clone);

```

接受template插入的元素，叫做宿主元素（host）。在template之中，可以对宿主元素设置样式。

```html

<template>
<style>
  :host {
    background: #f8f8f8;
  } 
  :host(:hover) {
    background: #ccc;
  }
</style>
</template>

```

## Custom Element

HTML预定义的网页元素，有时并不符合我们的需要，这时可以自定义网页元素，这就叫做Custom Element。它是Web component技术的核心。

下面的代码用于测试浏览器是否支持自定义元素。

```javascript

function supportsCustomElements() {
  return 'registerElement' in document;
}

if (supportsCustomElements()) {
  // 支持
} else {
  // 不支持
}

```

### document.registerElement()

document.registerElement方法用来注册新的网页元素，它返回一个构造函数。

```javascript

var XFoo = document.registerElement('x-foo');

```

上面代码注册了网页元素x-foo。可以看到，document.registerElement方法的第一个参数是一个字符串，表示自定义的网页元素的名称。需要注意的是，自定义元素的名称中，必须包含连字号（-），用来与预定义的网页元素相区分。连字号可以是一个也可以是多个。

document.registerElement方法还可以接受第二个参数，表示自定义网页元素的原型对象。

```javascript

var MyElement = document.registerElement('user-profile', {
  prototype: Object.create(HTMLElement.prototype)
});

```

上面代码注册了user-profile。document.registerElement方法的第二个参数，指定它的原型对象是HTMLElement.prototype（它是浏览器内部所有网页元素的原型）。如果想让自定义元素继承某种特定的网页元素，就要指定extends属性。比如，想让自定义元素继承h1元素，需要写成下面这样。

```javascript

var MyElement = document.registerElement('another-heading', {
  prototype: Object.create(HTMLElement.prototype),
  extends: 'h1' 
});

```

另一个是自定义按钮（button）元素的例子。

```javascript

var MyButton = document.registerElement('my-button', {
  prototype: Object.create(HTMLButtonElement.prototype),
  extends: 'button'
});

```

如果要继承一个自定义元素，比如创造`x-foo-extended`继承`x-foo`，也是一样写法。

```javascript

var XFooExtended = document.registerElement('x-foo-extended', {
  prototype: Object.create(HTMLElement.prototype),
  extends: 'x-foo'
});

```

总之，如果要自定义一个元素A继承元素B，那么元素A的原型必须是元素B的原型。使用的时候，需要把它写在所继承的网页元素的is属性之中，读起来就是B元素“is”A元素。

```html

<h1 is="another-heading">
  Page title
</h1>

<button is="my-button">

```

自定义元素注册成功以后，document.registerElement方法返回一个构造函数。下一步就可以用这个构造函数，生成自定义元素的实例。

```javascript

document.body.appendChild(new XFoo());

```

上面代码中，XFoo是自定义元素的构造函数，appendChild将它的实例插入DOM。

### 添加属性和方法

自定义元素的强大之处，就是可以在它上面定义新的属性和方法。

```javascript

var XFooProto = Object.create(HTMLElement.prototype);

var XFoo = document.registerElement('x-foo', {prototype: XFooProto});

```

上面代码注册了一个x-foo标签，并且指明原型继承HTMLElement.prototype。现在，我们就可以在原型上面，添加新的属性和方法。

```javascript

// 添加属性
Object.defineProperty(XFooProto, "bar", {value: 5});

// 添加方法
XFooProto.foo = function() {
  console.log('foo() called');
};

// 另一种写法
var XFoo = document.registerElement('x-foo', {
  prototype: Object.create(HTMLElement.prototype, {
    bar: {
      get: function() { return 5; }
    },
    foo: {
      value: function() {
        console.log('foo() called');
      }
    }
  })
});

```

### 回调函数

自定义元素的原型有一些属性，用来指定回调函数，在特定事件发生时触发。

- **createdCallback**：实例生成时
- **attachedCallback**：实例插入HTML文档时
- **detachedCallback**：实例从HTML文档移除时
- **attributeChangedCallback(attrName, oldVal, newVal)**：添加、移除、更新属性时

下面是一个例子。

```javascript

var proto = Object.create(HTMLElement.prototype);

proto.createdCallback = function() {
  console.log('created');
  this.innerHTML = 'This is a my-demo element!';
};

proto.attachedCallback = function() {
  console.log('attached');
};

var XFoo = document.registerElement('x-foo', {prototype: proto});

```

利用回调函数，可以方便地在自定义元素中插入HTML语句。

```javascript

var XFooProto = Object.create(HTMLElement.prototype);

XFooProto.createdCallback = function() {
  this.innerHTML = "<b>I'm an x-foo-with-markup!</b>";
};

var XFoo = document.registerElement('x-foo-with-markup', 
			{prototype: XFooProto});

```

上面代码定义了createdCallback回调函数，生成实例时，该函数运行，插入如下的HTML语句。

```html

<x-foo-with-markup>
   <b>I'm an x-foo-with-markup!</b>
</x-foo-with-markup>

```

## Shadow DOM

所谓Shadow DOM指的是，浏览器将模板、样式表、属性、JavaScript代码等，封装成一个独立的DOM元素。外部的设置无法影响到其内部，而内部的设置也不会影响到外部，与浏览器处理原生网页元素（比如&lt;video&gt;元素）的方式很像。Shadow DOM最大的好处有两个，一是可以向用户隐藏细节，直接提供组件，二是可以封装内部样式表，不会影响到外部。Chrome 35+支持Shadow DOM。

Shadow DOM元素需要通过createShadowRoot方法创造，然后将其插入HTML文档。

```javascript

var shadowRoot = element.createShadowRoot();
shadowRoot.appendChild(body);

```

上面代码创造了一个shadowRoot元素，然后将其插入HTML文档。

我们也可以指定，网页中某个现存的元素为Shadom DOM的根元素。

```html

<button>Hello, world!</button>

<script>
	var host = document.querySelector('button');
	var root = host.createShadowRoot();
	root.textContent = '你好，世界！';
</script>

```

上面代码指定现存的button元素，为Shadow DOM的根元素，并将button的文字从英文改为中文。

Shadow DOM更强大的作用是，可以为DOM元素指定独立的模板和样式表。

```html

<div id="nameTag">张三</div>

<template id="nameTagTemplate">
	<style>
		.outer {
			border: 2px solid brown;
		}
	</style>

	<div class="outer">
		<div class="boilerplate">
			Hi! My name is
		</div>
		<div class="name">
			Bob
		</div>
	</div>
</template>

```

上面代码是一个div元素和模板。接下来，就是要把模板应用到div元素上。

```javascript

var shadow = document.querySelector('#nameTag').createShadowRoot();
var template = document.querySelector('#nameTagTemplate');
shadow.appendChild(template.content.cloneNode());

```

上面代码先用createShadowRoot方法，对div创造一个根元素，用来指定Shadow DOM，然后把模板元素添加为Shadow的子元素。

另一种对Shadow DOM添加内容的方法，是直接对其指定innerHTML属性。

```javascript

var el = document.getElementById('demo1');
        
var shadow = el.createShadowRoot();
        
var shadowHTML = '<style>p {text-decoration: underline;}</style>';
shadowHTML += '<p>These paragraphs are in a shadow root.</p>';
shadow.innerHTML = shadowHTML;

```

上面代码在Shadow DOM之中，插入了样式和一个p元素。

## HTML Import

### 基本操作

长久以来，网页可以加载外部的样式表、脚本、图片、多媒体，却无法方便地加载其他网页，iframe和ajax都只能提供部分的解决方案，且有自己很大的局限。HTML Import就是为了解决这个问题，而提出来的。

下面代码用于测试当前浏览器是否支持HTML Import。

```javascript

function supportsImports() {
  return 'import' in document.createElement('link');
}

if (supportsImports()) {
  // 支持
} else {
  // 不支持
}

```

HTML Import用于将外部的HTML文档加载进当前文档。我们可以将组件的HTML、CSS、JavaScript封装在一个文件里，然后使用下面的代码插入需要使用该组件的网页。

```html

<link rel="import" href="dialog.html">

```

上面代码在网页中插入一个对话框组件，该组建封装在`dialog.html`文件。注意，dialog.html文件中的样式和JavaScript脚本，都对所插入的整个网页有效。

由于加载的网页的样式表和脚本，对当前网页也有效（准确得说，只有style标签中的样式对当前网页有效，link标签加载的样式表对当前网页无效）。所以可以把多个样式表和脚本，都放在所要加载的网页中，都从那里加载。这对大型的框架，是很方便的加载方法。

如果所要加载的网页，与当前网页不在同一个域时，对方的域名必须打开CORS。

```html

<!-- example.com必须打开CORS -->
<link rel="import" href="http://example.com/elements.html">

```

除了用link标签，也可以用JavaScript调用link元素。

```javascript

var link = document.createElement('link');
link.rel = 'import';
link.href = 'file.html'
link.onload = function(e) {...};
link.onerror = function(e) {...};
document.head.appendChild(link);

```

HTML Import加载成功时，会在link元素上触发load事件，加载失败时（比如404错误）会触发error，可以对这两个事件指定回调函数。

```html

<script async>
  function handleLoad(e) {
    console.log('Loaded import: ' + e.target.href);
  }
  function handleError(e) {
    console.log('Error loading import: ' + e.target.href);
  }
</script>

<link rel="import" href="file.html"
      onload="handleLoad(event)" onerror="handleError(event)">

```

上面代码中，handleLoad和handleError函数的定义，必须在link元素的前面。因为浏览器元素遇到link元素时，立刻解析并加载外部网页（同步操作），如果这时没有对这两个函数定义，就会报错。

HTML Import是同步加载，会阻塞当前网页的渲染，这主要是为了样式表的考虑。如果想避免这一点，可以为link元素加上async属性。当然，这也意味着，如果被加载的网页定义了Custome Element，就不能立即使用了，必须等HTML Import完成，才能使用。

```html

<link rel="import" href="/path/to/import_that_takes_5secs.html" async>

```

但是，HTML Import不会阻塞当前网页的解析和脚本执行。这意味着在加载的同时，主页面的脚本会继续执行。

HTML Import支持多重加载，即被加载的网页同时又加载其他网页。如果这些网页都重复加载同一个外部脚本，浏览器只会抓取并执行一次该脚本。比如，A网页加载了B网页，它们各自都需要加载jQuery，浏览器只会加载一次jQuery。

### 脚本的执行

被加载的网页中的内容，并不会显示在当前网页中，它只是用来供选用的，必须用DOM操作来获取加载的内容。具体来说，就是使用link元素的import属性，来获取加载的内容。这一点与iframe完全不同。

```javascript

var content = document.querySelector('link[rel="import"]').import;

```

发生以下情况时，link.import属性为null。

- 浏览器不支持HTML Import
- link元素没有声明`rel="import"`
- link元素没有被加入DOM
- link元素已经从DOM中移除
- 对方域名没有打开CORS

下面代码用于从加载的网页中，选取id为template的元素。

```javascript

var el = linkElement.import.querySelector('#template');

document.body.appendChild(el.cloneNode(true));

```

反过来也一样，被加载的网页中的脚本，不仅可以获取本身的DOM，还可以获取link元素所在的网页的DOM。

```javascript

// 以下代码位于被加载（import）的网页

// importDoc指向被加载的DOM
var importDoc = document.currentScript.ownerDocument;

// mainDoc指向主文档的DOM
var mainDoc = document;

// 将子页面的样式表添加主文档
var styles = importDoc.querySelector('link[rel="stylesheet"]');
mainDoc.head.appendChild(styles.cloneNode(true));

```

上面代码将所加载的网页的样式表，添加进主文档。

被加载的网页的脚本是直接在主网页的上下文执行，因为它的`window.document`指的是主网页的document，而且它定义的函数可以被主网页的脚本直接引用。

### Web Component的封装

对于Web Component来说，HTML Import的一个重要应用是在所加载的网页中，自动登记Custom Element。

```html

<script>
  // 定义并登记<say-hi>
  var proto = Object.create(HTMLElement.prototype);

  proto.createdCallback = function() {
    this.innerHTML = 'Hello, <b>' +
                     (this.getAttribute('name') || '?') + '</b>';
  };

  document.registerElement('say-hi', {prototype: proto});
</script>

<template id="t">
  <style>
    ::content > * {
      color: red;
    }
  </style>
  <span>I'm a shadow-element using Shadow DOM!</span>
  <content></content>
</template>

<script>
  (function() {
    var importDoc = document.currentScript.ownerDocument; //指向被加载的网页

    // 定义并登记<shadow-element>
    var proto2 = Object.create(HTMLElement.prototype);

    proto2.createdCallback = function() {
      var template = importDoc.querySelector('#t');
      var clone = document.importNode(template.content, true);
      var root = this.createShadowRoot();
      root.appendChild(clone);
    };

    document.registerElement('shadow-element', {prototype: proto2});
  })();
</script>

```

上面代码定义并登记了两个元素：\<say-hi\>和\<shadow-element\>。在主页面使用这两个元素，非常简单。

```html

<head>
  <link rel="import" href="elements.html">
</head>
<body>
  <say-hi name="Eric"></say-hi>
  <shadow-element>
    <div>( I'm in the light dom )</div>
  </shadow-element>
</body>

```

不难想到，这意味着HTML Import使得Web Component变得可分享了，其他人只要拷贝`elements.html`，就可以在自己的页面中使用了。

## Polymer.js

Web Components是非常新的技术，为了让老式浏览器也能使用，Google推出了一个函数库[Polymer.js](http://www.polymer-project.org/)。这个库不仅可以帮助开发者，定义自己的网页元素，还提供许多预先制作好的组件，可以直接使用。

### 直接使用的组件

Polymer.js提供的组件，可以直接插入网页，比如下面的google-map。。

```html

<script src="components/platform/platform.js"></script>
<link rel="import" href="google-map.html">
<google-map lat="37.790" long="-122.390"></google-map>

```

再比如，在网页中插入一个时钟，可以直接使用下面的标签。

```html

<polymer-ui-clock></polymer-ui-clock>

```

自定义标签与其他标签的用法完全相同，也可以使用CSS指定它的样式。

```css

polymer-ui-clock {
  width: 320px;
  height: 320px;
  display: inline-block;
  background: url("../assets/glass.png") no-repeat;
  background-size: cover;
  border: 4px solid rgba(32, 32, 32, 0.3);
}

```

### 安装

如果使用bower安装，至少需要安装platform和core components这两个核心部分。

```bash

bower install --save Polymer/platform
bower install --save Polymer/polymer

```

你还可以安装所有预先定义的界面组件。

```bash

bower install Polymer/core-elements
bower install Polymer/polymer-ui-elements

```

还可以只安装单个组件。

```bash

bower install Polymer/polymer-ui-accordion

```

这时，组件根目录下的bower.json，会指明该组件的依赖的模块，这些模块会被自动安装。

```javascript

{
  "name": "polymer-ui-accordion",
  "private": true,
  "dependencies": {
    "polymer": "Polymer/polymer#0.2.0",
    "polymer-selector": "Polymer/polymer-selector#0.2.0",
    "polymer-ui-collapsible": "Polymer/polymer-ui-collapsible#0.2.0"
  },
  "version": "0.2.0"
}

```

### 自定义组件

下面是一个最简单的自定义组件的例子。

```html

<link rel="import" href="../bower_components/polymer/polymer.html">
 
<polymer-element name="lorem-element">
  <template>
    <p>Lorem ipsum</p>
  </template>
</polymer-element>

```

上面代码定义了lorem-element组件。它分成三个部分。

**（1）import命令**

import命令表示载入核心模块

**（2）polymer-element标签**

polymer-element标签定义了组件的名称（注意，组件名称中必须包含连字符）。它还可以使用extends属性，表示组件基于某种网页元素。

```html

<polymer-element name="w3c-disclosure" extends="button">

```

**（3）template标签**

template标签定义了网页元素的模板。

### 组件的使用方法

在调用组件的网页中，首先加载polymer.js库和组件文件。

```html

<script src="components/platform/platform.js"></script>
<link rel="import" href="w3c-disclosure.html">

```

然后，分成两种情况。如果组件不基于任何现有的HTML网页元素（即定义的时候没有使用extends属性），则可以直接使用组件。

```html

<lorem-element></lorem-element>

```

这时网页上就会显示一行字“Lorem ipsum”。

如果组件是基于（extends）现有的网页元素，则必须在该种元素上使用is属性指定组件。

```

<button is="w3c-disclosure">Expand section 1</button>

```

## 参考链接

- Todd Motto, [Web Components and concepts, ShadowDOM, imports, templates, custom elements](http://toddmotto.com/web-components-concepts-shadow-dom-imports-templates-custom-elements/)
- Dominic Cooney, [Shadow DOM 101](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/)
- Eric Bidelman, [HTML's New Template Tag](http://www.html5rocks.com/en/tutorials/webcomponents/template/)
- Rey Bango, [Using Polymer to Create Web Components](http://code.tutsplus.com/tutorials/using-polymer-to-create-web-components--cms-20475)
- Cédric Trévisan, Building an Accessible Disclosure Button – using Web Components](http://blog.paciellogroup.com/2014/06/accessible-disclosure-button-using-web-components/)
- Eric Bidelman, [Custom Elements: defining new elements in HTML](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/)
- Eric Bidelman, [HTML Imports](http://www.html5rocks.com/en/tutorials/webcomponents/imports/)
- TJ VanToll, [Why Web Components Are Ready For Production](http://developer.telerik.com/featured/web-components-ready-production/)
- Chris Bateman, [A No-Nonsense Guide to Web Components, Part 1: The Specs](http://cbateman.com/blog/a-no-nonsense-guide-to-web-components-part-1-the-specs/)
