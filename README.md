# mp-practice

微信小程序开发相关实践，开发过程中遇到的一些问题和踩过的坑，目前正在完善中，欢迎大家反馈更好的想法与方案。


## 样式

注意某些组件默认含有 `::after`、`::before` 样式，如需重置组件样式，也需要重置 `::after`、`::before`。如：`button` 组件。

避坑点：

在样式文件 `wxss` 中不能直接引用本地文件(如图片)路径，但可以引用 `url`。

Note:

不要尽信在微信开发者工具上看到的效果，最后效果以真机调试为准。

## 关于 text 组件样式问题

text组件比较特殊，`<text>内容</text>` 标签包含起来的内容如果含有换行符，页面就也换行显示。

也就是说 `wxml` 中的换行符也会引起实际显示的差异，因此在含有 `text` 组件的页面标签中也要注意换行，特别是自动格式化后有的标签自动换行对齐之后引起问题。


以下两段代码页面渲染效果不一样：
```
<text>haha</text>

<text>
haha
</text>
```

比较好的实施方案是：如果能用`<view>`代替`<text>`则用`<view>`代替。

## background-image & image 组件

页面元素可以使用线上 `url` 地址作为背景图片来代替 `<image/>` 组件使用，缺点是不再能使用 `<image/>` `组件的bindload`、`binderror` 事件及 `mode` 属性。

## 关于 `iphoneX` 适配

Safe Area 大小:

```     
竖屏
    - top : 44.0
    - left : 0.0
    - bottom : 34.0
    - right : 0.0
横屏
    - top : 0.0
    - left : 44.0
    - bottom : 21.0
    - right : 44.0
```

Refs:

> https://www.jianshu.com/p/1432a94ef66f
> http://www.cnblogs.com/lolDragon/p/7795174.html

## 关于授权后登录失败

Note:

1. 小程序需要先绑定微信开放平台才能成功发起解密接口
2. appid & appsecret 一致且正确

## showToast 弹框提示

`wx.showToast` 当 `icon` 为 `none` 时， `title` 不限字数，否则最多 7 个汉字。

## 关于圆角图片渲染

圆角图片在某些机型下渲染时会先显示直角，然后闪现到圆角。

目前未找到原因。但在图片的样式中加上 `will-change:transform;` 可以解决可以解决这个问题。

## localStorage

接口 `wx.getStorageSync(key)` 根据key获取存储在 `localStorage` 中的数据。

如果 `key` 不存在则返回 `''`，但在某些机型则会抛出异常。

远程调试时 `localStorage` 的数据可能是旧的。新版本的小程序开发工具已经可以选择 `localStorage` 源。

## 远程调试

* 需要电脑和手机处于同一网络
* 特别需要注意 `localStorage` 是否为被调试者的数据
* 有时需要先删除已有开发版和体验版程序才能进行远程调试

## 特殊场景调试

### 扫描二维码 & 小程序码调试：

有两种方式：

1. 通过工具里编译下拉列表中选择「通过二维码编译」
2. 添加新的编译模式，启动参数里添加 `scene=value`，`value` 为 `encodeURIComponent` 结果

## 拦截页面返回问题

小程序的返回操作无法被拦截。

小程序可以开启全屏模式，然后自定义返回按钮，在返回按钮的事件上进行判断，再进一步决定是否调用 `wx.navigateBack` 进行返回。

app.json
```
{
  "window": {
    "navigationStyle": "custom" // 全屏模式
  }
}
```

**然而……**

如果用户通过实体键返回，依然无能为力。

对于表单操作页面，返回时可以先将表单数据缓存，再次进入表单页时提示并自动填入上次缓存结果。

[论坛帖子](https://developers.weixin.qq.com/blogdetail?action=get_post_info&docid=000e027b6540a80702a68f08b5b400&highline=%E6%8B%A6%E6%88%AA%20%E8%BF%94%E5%9B%9E)

## 防重复跳转 / 点击

页面跳转相关API在快速点击时进行跳转时会重复触发，官方回复这是一个已知问题，并表示会在后续版本中进行改善。然而……然后……没有然后……

[论坛帖子](https://developers.weixin.qq.com/blogdetail?action=get_post_info&docid=000a4a12b682389682467dc0d5b400)

目前解决页面重复跳转最简单方案为加一个延时：

```html
<button bindtap="gotoPageA">跳转页面A</button>
```

```js
Page({
  gotoPageA(e) {
    setTimeout(() => {
      wx.navigateTo({
        url: '../pageA/pageA',
      })
    }, 100);
  }
})
```

按钮的防重复点击也可以通过一个变量进行控制：

```html
<button bindtap="duplicateClick">防重复点击按钮</button>
```

```js
Page({
  fetchingData() {
    if (!this.fetching) {
      console.log('fetching data...')
      setTimeout(() => {
        this.fetching = false;
        console.log('fetching data over')
      }, 3000)
    }
    this.fetching = true;
  },
  duplicateClick(e) {
    this.fetchingData();
  }
})
```

最后还有一个简单粗暴的方式实现防重复点击：在点击按钮后直接 `wx.showLoading` 弹出遮罩，缺点是对用户不太友好。

[wechatide://minicode/0NFGLQmq6FZS](wechatide://minicode/0NFGLQmq6FZS)

## setData

```
setData 函数用于将数据从逻辑层发送到视图层（异步），同时改变对应的 this.data 的值（同步）
```

setData是最容易引发性能问题的接口，使用时注意：

* 不要频繁 setData
* 每次 setData 不要传递大量新数据，单次设置的数据不能超过1024kB
* 页面隐藏后不要再 setData

如：进度条组件，onPageScroll事件。

页面展示进度条组件时，因为需要动态展示进度变化，因为需要频繁的通过 `setData` 向页面传输最新的进度值。页面滚动事件同理，因为页面滚动事件会频繁触发，如果在此事件中需要向页面传输数据，也会存在频繁调用问题。

解决方案：

* 尽量避免在频繁触发的事件中调用 `setData`，如果一定要调用，可以在代码中控制调用的频率。
* 避免 `setData`大对象，只传输必要的值。如传输进度数据时，不传输整个对象 `progressData`，改用 `progressData.currentPercent`。

如果你有更好的方案或想法，欢迎反馈。

Note：

* `setData` 使用不当会造成页面卡顿，只有页面展示时需要用到的数据才使用 `setData`，中间的缓存数据可以挂载到页面实例 `this` 上，而不是 `this.data`。
* 直接修改 `this.data` 而不调用 `this.setData` 是无法改变页面的状态的，还会造成数据不一致。

[官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/performance/tips.html)

## 关于ES6语法兼容：`instanceof`、`padStart`

微信小程序的ES6 API部分有兼容问题及不在同的机型上有不同反应，避免使用罕见API，如 `instanceof` 和 `padStart()`

### instanceof 示例

```js
class ImgFormatError extends Error {
  constructor(msg){
    super(msg);
    this.name = 'ImgFormatError'
  }
}

let err = new ImgFormatError('.gif not allowed.');
console.log('err instanceof ImgFormatError =>', err instanceof ImgFormatError);

// => 小程序返回 false 
// => 浏览器返回 true
```

使用 `instanceof` 退化方案：
```js
function ImgFormatError(message) {
  this.message = message
  this.name = 'ImgFormatError';
}
ImgFormatError.prototype = new Error();
```

[wechatide://minicode/bI9viQmD6yZe](wechatide://minicode/bI9viQmD6yZe)

### padStart

`padStart is not a function`

使用自定义方法替代：

```js
function padLeft(v, str, len) {
  if (v.length >= len) {
    return v;
  }

  let r = '';
  let dis = len - v.length;
  for (let i = 0; i < dis; i++) {
    r += str;
  }
  return r + v;
}
```

## 在页面中调用方法

- 页面中事件绑定可以直接使用页面实例的方法……但，在页面中无法调用页面实例方法进行计算并显示，只能进行简单数学运算与字符串拼接。

- 如要在页面渲染时调用特定的函数进行计算，需要使用小程序脚本语言 `WXS`（WeiXin Script）。

- 使用 `WXS` 可能会带来部分代码的重复及维护复杂度，请酌情使用。

- 在 `WXS` 中使用 `let` 声明变量会报错（不要问我为什么知道），因为 `WXS` 不是 `javascript`。[参考WXS官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxs/)
 
#### 示例：

tool.wxs

```js
function sayHello(str1, str2) {
  // 变量声明使用 `var`
  var hello = str1 + str2;
  return hello.toUpperCase();
}

module.exports = {
  sayHello: sayHello
};
```

index.js
```js
Page({
  data: {
    foo: 'hello',
    bar: 'world',
    num: 10
  },
  sayHello(){
    let hello = this.data.foo + this.data.bar;
    return hello.toUpperCase();
  },
  sayHelloTap(e){
    wx.showToast({
      title: this.data.foo,
      icon: 'none'
    })
  }
})
```

index.wxml
```html
<button bindtap="sayHelloTap">绑定事件sayHelloTap</button>


<view>调用变量的 toString(失败)：{{ num.toString() }}</view>
<view>调用字符串的 toUpperCase(失败)：{{ foo.toUpperCase() }}</view>
<view>调用 sayHello() 方法显示返回值(失败)：{{ sayHello() }}</view>

<view>简单数学运算(成功)：{{ (num + 1) * 2 }}</view>
<!-- helloworld  -->
<view>直接使用字符串拼接(成功)：{{ foo + bar }}</view>

<!-- 导入tool.wxs定义的模块，并命名为 pageTool  -->
<wxs src="./tool.wxs" module="pageTool" />
<!-- WORLDHELLO  -->
<view>调用WXS定义的方法(成功)：{{ pageTool.sayHello('world', 'hello') }}</view>
```

[wechatide://minicode/mXklPQmx6QZQ](wechatide://minicode/mXklPQmx6QZQ)

## 页面间传值

页面之间传值有如下几种场景：

1. 父页面向子页面传递数据（跳转到新页）
2. 子页面向父页面传递数据（点返回时）
3. 兄弟页面之间传递数据（tabBar页面间）

页面之间传值有如下几种方式：

1. 通过 `url` 参数
2. 通过 `localStorage`
3. 通过全局变量，挂载到 `app` 或 `wx`
4. 通过事件 `PubSub`
5. 通过引用页面栈中的页面实例直接赋值或操作
6. 通过 wacter

以下几种方式假设 `pageA` 与 `pageB` 为同级的两个 `tabBar` 页，`pageC` 为子页面——

### 方式1：`url` 参数传值

pageA
```js
Page({
  gotoChildPageC(e){
    wx.navigateTo({
      url: '../pageC/pageC?pageA=hello',
    })
  },
  gotoChildPageCWidthObject(e) {
    let data = JSON.stringify({
      hello: 'world',
      foo: {
        bar: 'foo bar'
      }
    });
    wx.navigateTo({
      url: `../pageC/pageC?pageAObject=${data}`,
    })
  },
  // 失败
  gotoChildPageCWidthBigObject(e){
    let data = []
    for (let i = 0; i < 100000; i++) {
      data.push(Math.random())
    }
    data = JSON.stringify(data)
    wx.navigateTo({
      url: `../pageC/pageC?pageABigObject=${data}`,
    })
  }
})
```

pageC
```js
Page({
  onLoad: function(options){
    let data = options.pageA;
    console.log('data from parent page A:', data);

    if (options.pageAObject) {
      let data2 = JSON.parse(options.pageAObject);
      console.log('object data from parent page A:', data2);
    }

    if (options.pageABigObject) {
      let data3 = JSON.parse(options.pageABigObject);
      console.log('big object data from parent page A:', data3);
    }
  }
})
```

**优点：**
- 简单方便

**缺点：**
- 只能传递简单数据，复杂对象数据需要 `JSON.stringify`
- 有最大数据量限制：`invokeWebviewMethod 数据传输长度为 1927463 已经超过最大长度 1048576B（1MB）`
- 不能在子页面跳转到父页面（即返回）的情况使用（**wx.navigateBack无法加参数**）
- 不能在跳转到 `tabBar` 页时使用，即兄弟页面之间也无法传值：`wx.switchTab: url 不支持 queryString`

### 方式2：`localStorage` 传值

pageA
```js
Page({
  // 跳转到子页面
  gotoChildPageCWithBigLocalStorage(e){
    let data = app.getBigData();
    wx.setStorageSync('lsPageA', data)
    wx.navigateTo({
      url: '../pageC/pageC',
    })
  },
  // 跳转到兄弟页面
  gotoSlibingPageBWidthLocalStorage(e){
    wx.setStorageSync('lsPageASibling', 'hello world')
    wx.switchTab({
      url: '../pageB/pageB',
    })
  },

  // 子页面传递过来的数据
  onShow: function () {
    let localStoragePageC = wx.getStorageSync('lsPageC');
    if (localStoragePageC) {
      console.log('big localStorage data from child page C:', localStoragePageC);
      wx.removeStorageSync('lsPageC');
    }
  }
})
```

pageB
```js
Page({
  // 兄弟页面传递过来的数据
  onShow: function () {
    let localStoragePageASlibling = wx.getStorageSync('lsPageASibling');
    if (localStoragePageASlibling) {
      console.log('localStorage data from sibling page A:', localStoragePageASlibling);
      wx.removeStorageSync('lsPageASibling');
    }
  }
})
```

pageC
```js
Page({
  // 跳转到父页面
  gotoParentPageAWithBigLocalStorage(e){
    let data = app.getBigData();
    wx.setStorageSync('lsPageC', data)
    wx.navigateBack();
  }，
  // 父页面传递过来的数据
  onShow: function () {
    let localStoragePageA = wx.getStorageSync('lsPageA');
    if (localStoragePageA) {
      console.log('big localStorage data from parent page A:', localStoragePageA);
      wx.removeStorageSync('lsPageA');
    }
  }
})

```

**优点：**
- 简单易用
- 三种页面传值场景下均可使用

**缺点：**
- 需要及时清除数据，以免造成问题，有一点维护量
- `localStorage` 在某些机型上可能出现读写失败

**注意：**

* `localStorage` 也有存储限制：`setStorageSync:fail exceed storage max size 10Mb`，但实际开发过程不应该传过大数据。

### 方式3：全局变量传值

pageA
```js
Page({
  gotoChildPageCWithGlobal(e){
    app.parentPageAGlobalData = 'hello world';
    wx.navigateTo({
      url: '../pageC/pageC',
    })
  },
  gotoSiblingPageBWithGlobal(e){
    app.siblingPageAGlobalData = 'hello world';
    wx.switchTab({
      url: '../pageB/pageB',
    })
  },
  onshow(){
    if (app.childPageCGlobalData) {
      console.log('global data from child page C:', app.childPageCGlobalData);
      app.childPageCGlobalData = '';
    }
  }
})
```

pageB
```js
Page({
  // 兄弟页面传递过来的数据
  onShow: function () {
    if (app.siblingPageAGlobalData) {
      console.log('global data from sibling page A:', app.siblingPageAGlobalData);
      app.siblingPageAGlobalData = '';
    }
  }
})
```

pageC
```js
Page({
  gotoParentPageAGlobal(e){
    app.childPageCGlobalData = 'hello world from child page C';
    wx.navigateBack();
  },
  onShow: function () {
    if (app.parentPageAGlobalData) {
      console.log('global data from parent page A:', app.parentPageAGlobalData);
      app.parentPageAGlobalData = '';
    }
  }
})
```


**优点：**
- 简单易用
- 直接操作内存，速度更快
- 三种页面传值场景下均可使用

**缺点：**
- 全局变量污染，有一定的维护量

### 方式4：`PubSub` 传值

pageA
```js
Page({
  onLoad:function(options){
    pubSub.on('EMIT_TO_PAGE_A', (data) => {
      console.log('Received data: ', data)
    });
  }
})
```

pageC
```js
Page({
  gotoParentPageAPubSub(e){
    pubSub.emit('EMIT_TO_PAGE_A', 'hello world from child page c');
    wx.navigateBack();
  }
})
```

**优点：**
- 页面间频繁通信的时候比较实用
- 事件频繁触发并传递数据时比较实用

**缺点：**
- 需要注意事件重复绑定的问题
- 不能在父页面跳转到子页面时使用
- 不能在 `tabBar` 页面使用

### 方式5：通过引用页面栈中的页面实例直接赋值或操作

app.js
```
App({
  // 存储页面栈中的页面实例
  pagePool:{}
})
```

pageA
```js
Page({
  preOperation(data){
    console.log('page A do something first:', data);
  },
  onLoad: function(options){
    app.pagePool.pageA = this;
  }
})
```

pageC
```js
Page({
  gotoParentPageAPageInstance(e){
    app.pagePool.pageA.preOperation('data from child page c');
    console.log('page C do another thing after page A preOperation');
    wx.navigateBack();
  }
})
```

**优点：**
- 直接拿到页面实例，直接对页面进行操作，没有中间变量

**缺点：**
- 不能在父页面跳转到子页面时使用（因为此时页面栈中还没有子页面的实例）
- 以不同的场景进入小程序，页面栈的顺序可能不一致，需要单独处理

综上所述：

- 在跳转到新页面并只需要传递简单值的情况下，使用 `url` 参数传值最优；
- 在需要子页面向父页面传值或不依赖事件触发的情况下，使用 `全局变量` 传值最优，需要专门维护全局变量；
- 依赖于事件及页面频繁通信的情况下，使用 `PubSub` 发布订阅的方式传值最优；
- 特殊页面跳转场景，如需要在跳转之前对目标页面先做某些预置操作则可以使用 `引用目标页面实例的方法` 直接进行操作。

[wechatide://minicode/IFaALVmT6ZZj](wechatide://minicode/IFaALVmT6ZZj)

如果你有更好的方案或想法，欢迎反馈。

## 异步调用 & 回调 & Promise

小程序API除了少数同步接口外，大部分都是异常接口，如果使用原生接口，将会掉入所谓的 `回调地狱`。

```js
wx.getStorage({
    key: 'loginResult',
    success: function(res) {
        console.log('登录成功，取数据');
        wx.request({
            url: 'https://test.com/getData',
            data: 'token',
            success: function(res){
                console.log('获取数据成功，setData');
            }
        })
    },
    fail: function(res) {
        wx.login({
            success: function(res) {
                if (res.code) {
                    //发起网络请求
                    wx.request({
                        url: 'https://test.com/onLogin',
                        data: {
                            code: res.code
                        },
                        success: function(res){
                            wx.setStorage({
                                key: 'loginResult',
                                data: res.data,
                                success: function(res) {
                                    console.log('登录成功，取数据');
                                    wx.request({
                                        url: 'https://test.com/getData',
                                        data: 'token',
                                        success: function(res){
                                            console.log('获取数据成功，setData');
                                        }
                                    })
                                }
                            })
                        }
                    })
                } else {
                    console.log('登录失败！' + res.errMsg)
                }
            }
        });
    }
})
```

还好小程序支持 `Promise`，我们只要将异步接口封装一下，就可以简化代码调用：

```js
function wxMethod() {
    let promise = new Promise(function(resolve, reject){
        wx.method({
            success(res) {
                return resolve(res.data);
            },
            fail(reason) {
                return reject(reason);
            }
        })
    });
    // wxMethod 返回的是一个Promise对象
    return promise;
}


// 注意 `return`
getStorage()
.then(res => {
    // 取数据
    return request(res.token);
}).catch(res => {
    return login();
}).then(res => {
    // 调登录业务接口
    return setStorage(res.loginResult);
}).then(res => {
    // 取数据
    return request(res.token);
}).catch(res => {
    console.log('登录失败');
})
```

:smile: 成功将 `三角形` 代码变成 `流线形` 代码。


使用 `Promise` 注意：

- 不要忘记 `return`。
    * 要想链式掉用，必须保证前一个函数也 `return` 一个 `Promise` 对象
    * `then`和`catch` 代码中如果没有写 `return` 相当于 `return undefined`，经过 `Promise` 包装实际相当于 `return Promise.resolve(undefined)`
- 不要忘记 `reject` 情形，否则内部的异常会被 `吞掉`，在外层即便写了 `catch` 也不管用，会使调试异常困难。

## 关于 `this` 和 `箭头函数`

```js
Page({
  sayHello(){
    console.log('I am a page method.');
  },
  
  tap1(e) {
    // this.sayHello is not a function: setTimeout回调中的 this 已经不是 Page 实例对象
    setTimeout(function(){
      this.sayHello();
    }, 1000);
  }
})
```

在小程序生命周期内，经常要使用 `this` 访问小程序应用实例对象及页面实例对象，但因为回调函数的存在，`this` 的指向不总是指向小程序应用实例或页面实例。

此时有两种解决方案：

### 1. 使用中间变量

```js
Page({
  sayHello(){
    console.log('I am a page method.');
  },
  
  tap2(e) {
    // 定义中间变量，保证调用页面方法时拿到的始终都是 Page 的实例对象
    let self = this;
    setTimeout(function () {
      self.sayHello();
    }, 1000);
  }
})
```

### 2. 使用箭头函数

```js
Page({
  sayHello(){
    console.log('I am a page method.');
  },
  
  tap3(e) {
    // 箭头函数中的 this 即 Page 实例对象
    setTimeout(() => {
      setTimeout(() => {
        setTimeout(() => {
          this.sayHello();
        }, 1000);
      }, 1000);
    }, 1000);
  }
})
```

[wechatide://minicode/wbt7NQmA66Zc](wechatide://minicode/wbt7NQmA66Zc)

**Note:**

* 使用中间变量 `self` 简单可行，也不失为一个解决方案，但带来了不必要的中间变量赋值操作，代码多了容易扰乱视线；另外对一些简单调用场景，不需要定义中间变量时又可以直接使用 `this`，时间久了就会存在 `self` 和 `this` 混用的的状况。如果中间变量使用了诸如 `that` 的名字就更是混乱了。

* 使用箭头函数的好处显而易见，避免了中间变量的干扰，使代码更简洁不混乱。但如果存在多级回调时，中间的某个回调使用的不是箭头函数，同样会导致 `this` 的指向发生改变。

个人看法：

箭头函数更好。因为多级回调的情况正是需要尽量避免的地方，使用 `Promise` 将异步调用封装起来，正好避免掉入 `回调地狱`。同时因为没有了 `self` 的中间变量，每次在箭头函数中可以放心使用 `this`，而不必使用额外 `脑开销` 来判断 **这个`this`到底指向谁？**。

如果你有更好的方案或想法，欢迎反馈。

## 在视频上覆盖圆角元素问题

现象：

部分 `iPhone` 上无法在视频上显示圆角元素。

原因：

`cover-view` 圆角 `border-radius` 与 `padding` 样式不能同时使用。

解决办法：

在 `cover-view` 的子元素外面再套一个过渡层 `cover-view`，将 `padding` 改为 `margin` 应用在过渡层上。


代码：

```html
<!-- 修改前 -->
<cover-view style="border-radius: 10rpx;padding: 0 16rpx;">我是圆角元素</cover-view>


<!-- 修改后 -->
<cover-view style="border-radius: 10rpx;">
    <cover-view style="margin: 0 16rpx;">我是圆角元素</cover-view>
</cover-view>
```

## 视频上圆角元素循环动画问题

视频上的圆角元素做循环动画时出现：图片变形及旋转不绕图片中心旋转问题。

`Android` 解决:

去掉元素圆角，使用 `png` 代替后，并修改旋转中心 `transform-origin: 50% 50%` 可解决，但 `iPhone` 依然存在问题。

## 关于自定义 tabBar

- 注意 icon 的格式不能为 svg 等较新格式
- icon 的文件大小不能超过 `40Kb`

## 如果禁止分享出去的卡片二次分享？

在分享页面使用如下方法：

```
// 这样
wx.updateShareMenu({
    withShareTicket: true,
    success() {
    }
})

// 或者这样   
wx.showShareMenu({
    withShareTicket: true
})
```

注：这个方法只能禁止分享到群里的卡片，无法禁止分享给个人的卡片再次分享。

## 关于 AppSecret

注意保管好 `AppSecret`，轻易不要重置。如果已重置，须将相关受影响的功能检查一遍。

`AppSecret` 改变好会带来如下影响：

- 小程序登录接口调用失败
- 生成二维码等后端需要 `access_token` 的接口
- 绑定的阿拉丁账号

## 关于API上报数据

1. 注意数字类型数据的上报，如果是从页面参数中获取的，会默认上报为字符串
2. 在页面 `onLoad` 中直接上报通常会失败，建议加setTimeout

## IDE

小程序自带的 `微信Web开发者工具` 缺陷和 bug 比较多，还不是很成熟，建议只在最后必要的时候使用，其它大部分时候可以使用自己称手的 `IDE` 来进行开发。
