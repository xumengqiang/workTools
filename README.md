# workTools
工作中遇到的一些方法总结


## 判断中文

```js
if(/[\u4E00-\u9FA5]/g.test('你好')){console.log('不可输入汉字')}
```

## 日期格式化

```js
dateFmt(timeStamp = Date.now(), format = 'YYYY-MM-DD hh:mm:ss') {
    let date = new Date(timeStamp);
    let formatNumber = n => (n < 10 ? '0' + n : n);
    let str = format
        .replace('YYYY', date.getFullYear())
        .replace('MM', formatNumber(date.getMonth() + 1))
        .replace('DD', formatNumber(date.getDate()))
        .replace('hh', formatNumber(date.getHours()))
        .replace('mm', formatNumber(date.getMinutes()))
        .replace('ss', formatNumber(date.getSeconds()));
    return str;
},

console.log(dateFmt());  //2022-08-22 17:30:27

```


## 获取文件的临时链接

#### window.URL.createObjectURL

**语法**
- bold参数是一个file对象或者bold对象。
- objectURL是生成的对象URL，通过这个URL，可以获取到所指定文件的完整内容。

```js
// 接口请求获取二进制流
// 注意接口的配置，一定要设置 responseType: 'blob'
export function fileDownload() {
    return request({
        url: `xxxxxxxxx`,
        method: 'GET',
        responseType: 'blob',
    });
}

let res = await fileDownload();

const binaryData = [res.data];

// 一些使用场景的例子

// 1、获取blob链接----预览 
// type类型可以更换对应的文件类型

const url  = window.URL.createObjectURL(new Blob(binaryData, { type: 'application/pdf' }));
window.open(url);

// 2、获取blob链接----下载

const blob = new Blob(binaryData);
const fileName = `${event.name}`;
const elink = document.createElement("a");
elink.download = fileName;
elink.style.display = "none";
elink.href = window.URL.createObjectURL(blob);
document.body.appendChild(elink);
elink.click();
window.URL.revokeObjectURL(elink.href); // 释放URL 对象
document.body.removeChild(elink);

```

## 节流和防抖

#### 节流 （规定时间内 只触发一次）
- 节流策略（throttle），控制事件发生的频率，如控制为1s发生一次，甚至1分钟发生一次。与服务端(server)及网关(gateway)控制的限流 (Rate Limit) 类似。

- 作用： 高频率触发的事件,在指定的单位时间内，只响应第一次。

#### 节流的应用场景

- 鼠标连续不断地触发某事件（如点击），单位时间内只触发一次；
- 监听滚动事件，比如是否滑到底部自动加载更多，用throttle来判断。例如：懒加载；
- 浏览器播放事件，每个一秒计算一次进度信息等

```js
// 方式一：使用时间戳 （时间戳版本会优先执行，点击立马执行一次）
export const throttle1 = (fn, delay = 1000) => {
  //距离上一次的执行时间
  let lastTime = 0;
  return function () {
    let _this = this;
    let _arguments = arguments;
    let now = new Date().getTime();
    //如果距离上一次执行超过了delay才能再次执行
    if (now - lastTime > delay) {
      fn.apply(_this, _arguments);
      lastTime = now;
    }
  };
};

// 方式二: 使用定时器 （定时器版本会后置执行，点击需等待delay时间之后执行）
export const thorttle2 = (fn, delay = 1000) => {
  // 接收定时器的地址
  let timer = null;
  return function () {
    let _this = this;
    let args = arguments;
    // 如果等到了delay才能执行并清空定时器
    if (!timer) {
      timer = setTimeout(function () {
        timer = null;
        fn.apply(_this, args);
      }, delay);
    }
  };
};

// vue2中使用
change: throttle((val) => { console.log(val); }, 1000)  // change是你的方法名,val接收的参数

// vue3中使用
const change = thorttle((val) => { console.log(val);}, 1000) // change是你的方法名,val接收的参数
```

#### 防抖 （多次触发 只执行最后一次）

- 防抖策略（debounce）是当事件被触发后，延迟n秒后再执行回调，如果在这n秒内事件又被触发，则重新计时。

- 作用： 高频率触发的事件，在指定的单位时间内，只响应最后一次，如果在指定的时间内再次触发，则重新计算时间。

#### 防抖的应用场景
- 登录、发短信等按钮避免用户点击太快，以致于发送了多次请求，需要防抖
- 调整浏览器窗口大小时，resize 次数过于频繁，造成计算过多，此时需要一次到位，就用到了防抖
- 文本编辑器实时保存，当无任何更改操作一秒后进行保存

```js
export const debounce = (fn, delay = 1000) => {
  // 定时器
  let timer = null;
  // 将debounce处理结果当作函数返回
  return function () {
    // 保留调用时的this上下文
    let context = this;
    // 保留调用时传入的参数
    let args = arguments;
    // 每次事件被触发时，都去清除之前的旧定时器
    if (timer) {
      clearTimeout(timer);
    }
    // 设立新定时器
    timer = setTimeout(function () {
      fn.apply(context, args);
    }, delay);
  };
};

// vue2中使用
change: debounce((val) => { console.log(val); }, 1000)  // change是你的方法名 ,val接收的参数

// vue3中使用
const change = debounce((val) => { console.log(val);}, 1000) // change是你的方法名,val接收的参数
```