---
title: Vue相关的一些心得
date: 2018-06-19 14:31:07
tags: Vue
category: 开发
---

最近捣鼓了个[翻译质量检查报告](http://www.cfpa.team/TransQualityControl/)，用到的是[Vue.js](https://cn.vuejs.org/index.html)。虽然 Bilibili 专栏助手用到的也是 Vue，但这么多天过去了，还是比较手生的，于是借这个机会复习了一下 Vue 的基本内容。这次写这篇博客，也是希望自己之后不要忘记一些东西。

### 项目需求

后端通过`Python`脚本，现在已经为我们生成好了三个 json 文件，存放在`/json`目录下，分别是：

- `format.json`，负责存放格式符检查数据。
- `duplicate.json`，负责存放重名 key 检查数据。
- `branch.json`，负责存放语言文件串行检查数据。

我们需要通过读取这些文件，对其中存放的数据以表格给出的形式进行渲染。

### 导入分析

在处理数据之前，我们来看一看这些数据该如何导入。  
按照常规的思路，我们一般会采用`XMLHttpRequest`，或者是`fetch`，进行`Ajax`式的传输。但对于这样的一个纯静态的网站，在我看来，使用`Ajax`是一件非常划不来的事（旁白：划不来？？？）。因此，我开始考虑`import`。

#### 何为 Import？

使用`Nodejs`的玩家对 import 应该已经不陌生了。虽然很多人现在使用的依然是诸如`const fs = require('fs')`的语句，但从`Typescript`和`TSLint`的使用来看，很多人已经开始使用`import`的语句，进行原生的模块导入了。

我们来看一眼[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)上对`import`的解释：

> The `import` statement is used to import bindings which are exported by another module. Imported modules are in [strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) whether you declare them as such or not. The `import` statement cannot be used in embedded scripts.

这里限制了`import`语句引入的模块均在严格模式下执行，而对于我们的需求而言，我们只是要借助`import`引入一些变量（数组或对象），因此严格模式对我们的影响不大。

#### 如何 Import？

如果你是在`html`中想要直接使用，你必须按照下面的这个格式书写：

```html
<!DOCTYPE html>
<html>
  <head>
  ...
  </head>
  <body>
  ...
  <script type="module">
  import { part } from "module";
  </script>
  <body>
</html>
```

而如果是想要在外部`js`使用`import`，则需要按照下面的方式书写：

```html
<script type="module" src="./js/index.js"></script>
```

#### 如何修改输入以能够读入模块？

我们把所有的`json`格式文件的后缀名都修改为`.js`，并且在相应的文件开头加上这样一行：

```js
// format.js
export let formats = [
  ...
]

// duplicate.js
export let duplicates = [
  ...
]

// branch.js
export let branches = [
  ...
]
```

然后，我们就可以这样的方式引入了：

```html
<script type="module">
  import { formats } from "/json/format.js";
  import { duplicates } from "/json/duplicate.js";
  import { branches } from "/json/branch.js";
</script>
```

### 绑定 Vue

在完成了前期准备之后，我们就可以正式开始使用`Vue`了。首先是针对这三类数据，分别创建三个`Vue`对象：

```js
// 为了方便起见，省略html的内容，只保留<script>标签内的js语句
import { formats } from "/json/format.js";
import { duplicates } from "/json/duplicate.js";
import { branches } from "/json/branch.js";

let format = new Vue({
  el: "#format",
  data: {
    formats: formats
  }
});

let duplicate = new Vue({
  el: "#duplicate",
  data: {
    duplicates: duplicates
  }
});

let branch = new Vue({
  el: "#branch",
  data: {
    branches: branches
  }
});
```

然后，需要对`html`中的一些部分赋上 id。下面只展示了一个例子，其余二者同理：

```html
<div class="sub">
  <h2>格式符检查报告</h2>
  <table cellspacing="0" id="format">
    <tr>
      <th style="width:3%">编号</th>
      <th style="width:8%">模组ID</th>
      <th style="width:10%">key</th>
      <th>英文文本</th>
      <th>中文文本</th>
      <th style="width:8%">Weblate</th>
    </tr>
    ...
  </table>
</div>
```

这样，我们就可以让这几个 html 标签下的部分获取到对应`Vue`中的数据了。

### 渲染数据

渲染数据的过程中，我们使用的是`v-for`，它可以根据一定的格式（数组或对象均可），进行相同元素的渲染。

#### 单层渲染

格式符检查报告和语言文件串行检查报告的格式都比较简单，是基础的单层结构，这里就只贴出前者的代码：

```html
<div class="sub">
  <h2>格式符检查报告</h2>
  <table cellspacing="0" id="format">
    <tr>
      <th style="width:3%">编号</th>
      <th style="width:8%">模组ID</th>
      <th style="width:10%">key</th>
      <th>英文文本</th>
      <th>中文文本</th>
      <th style="width:8%">Weblate</th>
    </tr>
    <tr v-for="(format, index) in formats">
      <td style="width:3%">{{ index + 1 }}</td>
      <td style="width:8%">{{ format.modid }}</td>
      <td style="width:10%">{{ format.key }}</td>
      <td>{{ format.en_us }}</td>
      <td>{{ format.zh_cn }}</td>
      <td style="width:8%">
        <a
          :href="'https://weblate.sayori.pw/translate/langpack/' + format.modid + '/zh_cn/?q=' + format.key + '&context=on'"
          >[点击进入]</a
        >
      </td>
    </tr>
  </table>
</div>
```

#### 双层渲染

在渲染重名 key 检查报告时，遇到了一个比较棘手的问题：表格有两层嵌套。这就使得我们必须使用两次`v-for`，以完成两层的渲染。  
`v-for`如果想要进行多层嵌套，必须使用`<template>`进行嵌套，于是最后的代码如下：

```html
<div class="sub">
  <h2>重名key检查报告</h2>
  <table cellspacing="0" id="duplicate">
    <tr>
      <th style="width:3%">编号</th>
      <th style="width:10%">key</th>
      <th style="width:8%">模组ID</th>
      <th>英文文本</th>
      <th>中文文本</th>
      <th style="width:8%">Weblate</th>
    </tr>
    <template v-for="(value, index) in duplicates">
      <tr>
        <td style="width:3%" :rowspan="value.items.length 1">
          {{ index + 1 }}
        </td>
        <td style="width:10%" :rowspan="value.items.length 1">
          {{ value.key }}
        </td>
      </tr>
      <template v-for="(v, i) in value.items">
        <tr>
          <td style="width:8%">{{ v.mod }}</td>
          <td>{{ v.en_us }}</td>
          <td>{{ v.zh_cn }}</td>
          <td style="width:8%">
            <a
              :href="'https://weblate.sayori.pw/translate/langpack/' + v.mod + '/zh_cn/?q=' + value.key + context=on'"
              >[点击进入]</a
            >
          </td>
        </tr>
      </template>
    </template>
  </table>
</div>
```

### 最后的启示

`Vue.js`是我相对比较熟悉的前端框架了。虽然它本身对`Typescript`的支持一般，但这也是我第一个用于工程的前端框架。在高考完咕咕咕以来，我也很久没有碰过专栏助手了。  
归根结底，前端框架还是给人用的，因此就会产生诸如`v-for`之类的方便的东西。但正因为是人，所以也会有很多玄学的问题。诸如像上面的重名 key 检查报告中，遇到的怎么也不明白的样式错误，最后发现是不小心删了一个`</table>`……

### BGM

{% meting "421160925" "netease" "song" "autoplay" %}
