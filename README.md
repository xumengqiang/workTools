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
## vue移动端适配rem

#### 什么是rem？
- rem是响应式布局的一种，是相对于根元素(即html元素)font-size计算值的倍数的一个css单位。因为rem的特性相对长度单位，常被用来做移动适配，pc端页面不推荐使用rem
- rem布局的本质是等比缩放，一般是基于宽度针对设计稿的宽度 利用js动态获取屏幕的宽度
- rem是相对于根元素的字体大小的单位，也就是html的font-size大小，浏览器默认的字体大小是16px，所以默认的1rem=16px

#### 如何根据设计图设计rem比例
```js
// rem中的核心公式

/**
 * document.documentElement.style.fontSize  html的字体大小
 * document.documentElement.clientWidth     可视窗口的大小
 * 7.5 设计图的宽度除以100  (设计图默认按750计算)
 */
document.documentElement.style.fontSize = document.documentElement.clientWidth  / 7.5 + 'px'

如果可视窗口是375，设计图纸是750,
那么1rem=（375px 除以 750px ）乘以100是为了好计算px
这里的1rem=50px

如果一个九宫格中的图标是60px宽，70px高，那么就是60 除以 计算出来的px值（比例大小 +‘px）就等于图标宽度所需要的rem值，高度也一样
也就是60px 除以 50px =1.2rem
```


```js
// 安装两个插件
amfe-flexible
postcss-px2rem-exclude

// main.js配置
// 移动端引入amfe-flexible依赖 并在postcss.config.js中打开px2rem插件
import 'amfe-flexible';

// postcss.config.js中配置
module.exports = {
    plugins: {
        autoprefixer: {},
        //移动端项目开启此插件  将px转为rem
        'postcss-px2rem-exclude': {
            remUnit: '75', // 为了方便计算  1rem = 75px
            //白名单
            exclude: /node_modules/i,
        },
    },
};

```

## 身份证校验

```js
export const isIdCard = card => {
    // 身份证号
    if (!card) return true;
    let num = card.toUpperCase();
    // 身份证号码为15位或者18位，15位时全为数字，18位前17位为数字，最后一位是校验位，可能为数字或字符X。
    if (!/(^\d{15}$)|(^\d{17}([0-9]|X)$)/.test(num)) {
        return false;
    }
    // 校验位按照ISO 7064:1983.MOD 11-2的规定生成，X可以认为是数字10。
    // 下面分别分析出生日期和校验位
    let len;
    let re;
    let birthday;
    let sex;
    len = num.length;
    if (len == 15) {
        // 获取出生日期
        birthday = `19${card.substring(6, 8)}-${card.substring(8, 10)}-${card.substring(10, 12)}`;
        // 获取性别
        sex = parseInt(card.substr(14, 1)) % 2 == 1 ? 'M' : 'F';

        re = new RegExp(/^(\d{6})(\d{2})(\d{2})(\d{2})(\d{3})$/);
        let arrSplit = num.match(re);

        // 检查生日日期是否正确
        let dtmBirth = new Date(`19${arrSplit[2]}/${arrSplit[3]}/${arrSplit[4]}`);
        let bGoodDay;
        bGoodDay = dtmBirth.getYear() == Number(arrSplit[2]) && dtmBirth.getMonth() + 1 == Number(arrSplit[3]) && dtmBirth.getDate() == Number(arrSplit[4]);
        if (!bGoodDay) {
            return false;
        } else {
            // 将15位身份证转成18位
            // 校验位按照ISO 7064:1983.MOD 11-2的规定生成，X可以认为是数字10。
            let arrInt = new Array(7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2);
            let arrCh = new Array('1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2');
            let nTemp = 0;
            let i;

            num = `${num.substr(0, 6)}19${num.substr(6, num.length - 6)}`;
            for (i = 0; i < 17; i++) {
                nTemp += num.substr(i, 1) * arrInt[i];
            }
            num += arrCh[nTemp % 11];
        }
    } else if (len == 18) {
        // 获取出生日期
        birthday = `${card.substring(6, 10)}-${card.substring(10, 12)}-${card.substring(12, 14)}`;
        // 获取性别
        sex = parseInt(card.substr(16, 1)) % 2 == 1 ? 'M' : 'F';

        re = new RegExp(/^(\d{6})(\d{4})(\d{2})(\d{2})(\d{3})([0-9]|X)$/);
        let arrSplit = num.match(re);

        // 检查生日日期是否正确
        let dtmBirth = new Date(`${arrSplit[2]}/${arrSplit[3]}/${arrSplit[4]}`);
        let bGoodDay;
        bGoodDay = dtmBirth.getFullYear() == Number(arrSplit[2]) && dtmBirth.getMonth() + 1 == Number(arrSplit[3]) && dtmBirth.getDate() == Number(arrSplit[4]);
        if (!bGoodDay) {
            return false;
        } else {
            // 检验18位身份证的校验码是否正确。
            // 校验位按照ISO 7064:1983.MOD 11-2的规定生成，X可以认为是数字10。
            let valnum;
            let arrInt = new Array(7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2);
            let arrCh = new Array('1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2');
            let nTemp = 0;
            let i;
            for (i = 0; i < 17; i++) {
                nTemp += num.substr(i, 1) * arrInt[i];
            }
            valnum = arrCh[nTemp % 11];
            if (valnum != num.substr(17, 1)) {
                return false;
            }
        }
    }
    return {
        birthday,
        sex,
    };
};

// 正确返回
let res = isIdCard('370902197612012131')
console.log(res); // {birthday: '1976-12-01', sex: 'M'}

// 错误返回
let res = isIdCard('370902197612012130')
console.log(res); // false
```

## 车牌号验证方法

```js
export const isCar = Number => {
    let vehicleNumber = Number.toUpperCase();
    var xreg = /^[京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼使领A-Z]{1}[A-Z]{1}(([0-9]{5}[DF]$)|([DF][A-HJ-NP-Z0-9][0-9]{4}$))/;

    var creg = /^[京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼使领A-Z]{1}[A-Z]{1}[A-HJ-NP-Z0-9]{4}[A-HJ-NP-Z0-9挂学警港澳]{1}$/;

    if (vehicleNumber.length == 7) {
        return creg.test(vehicleNumber);
    } else if (vehicleNumber.length == 8) {
        return xreg.test(vehicleNumber);
    } else {
        return false;
    }
};
// 校验
let res = isCar('京A12345')   // true
let res = isCar('京A123456')  // false
let res = isCar('京AD12347')  // true
let res = isCar('京AB12347')  // false
```
## vue绘制二维码

```js
1、安装插件
npm install qrcodejs2
2、页面中使用
<div id="qrCode"></div>
import QRCode from 'qrcodejs2';

// 绘制二维码；text绘制的内容
bindQRCode(text) {
    // codeHtml.innerHTML = '';是为了防止重复绘制
    const codeHtml = document.getElementById('qrCode');
    codeHtml.innerHTML = '';
    new QRCode('qrCode', {
        text,
        width: 200,
        height: 200,
        colorDark: '#333333', //二维码颜色
        colorLight: '#ffffff', //二维码背景色
        correctLevel: QRCode.CorrectLevel.L, //容错率，L/M/H
    });
},
```

## vue中使用keep-alive


#### 简介
keep-alive 是 Vue 的内置组件，当它包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。
和 transition 相似，keep-alive 是一个抽象组件：它自身不会渲染成一个 DOM 元素，也不会出现在父组件链中。

#### 作用
在组件切换过程中将状态保留在内存中，防止重复渲染DOM，减少加载时间及性能消耗，提高用户体验性。

#### 原理
在 created 函数调用时将需要缓存的 VNode 节点保存在 this.cache 中／在 render（页面渲染） 时，如果 VNode 的 name 符合缓存条件（可以用 include 以及 exclude 控制），则会从 this.cache 中取出之前缓存的 VNode 实例进行渲染。
（VNode：虚拟DOM，其实就是一个JS对象）
#### 使用


| 参数名  | 值 | 描述 |
|--------|-----|-----|
|include | 字符串或正则表达式 | 只有名称匹配的组件会被缓存。|
|exclude | 字符串或正则表达式 | 任何名称匹配的组件都不会被缓存。|
|max     | 数字 | 最多可以缓存多少组件实例。|

- 注意: include/exclude 值是组件中的 name 命名，而不是路由中的组件 name 命名；

#### 需求
- A页面进入B页面不需要缓存
- B页面进入C页面不需要缓存
- C页面进入B页面需要缓存

#### 生命周期函数

|名称 | 描述 |
|-----|------|
|activated| 在 keep-alive 组件激活时调用， 该钩子函数在服务器端渲染期间不被调用。|
|deactivated | 在 keep-alive 组件停用时调用，该钩子在服务器端渲染期间不被调用。|

- 页面第一次进入，钩子的触发顺序：created->mounted->activated
- 退出时触发 deactivated
- 当再次进入（前进或者后退）时，只触发activated

```js
// vuex.js
const state = {
    cachedViews: [], // 缓存view，keepalive是AppMain组件下的
};
const mutations = {
    // 添加缓存
    ADD_CACHED_VIEW: (state, view) => {
        if (state.cachedViews.includes(view.name)) {
            return;
        }
        state.cachedViews.push(view.name);
    },
    // 移除缓存
    DEL_CACHED_VIEW: (state, view) => {
        const index = state.cachedViews.indexOf(view.name);
        index > -1 && state.cachedViews.splice(index, 1);
    },
};
const actions = {
    // 添加缓存view
    addCachedView({ commit }, view) {
        return new Promise(resolve => {
            commit('ADD_CACHED_VIEW', view);
            resolve();
        });
    },
    // 删除缓存view
    delCachedView({ commit }, view) {
        return new Promise(resolve => {
            commit('DEL_CACHED_VIEW', view);
            resolve();
        });
    },
};

// layout.vue 
<keep-alive :include="cachedViews" :max="15">
    <router-view :key="key" />
</keep-alive>

import { mapGetters } from 'vuex';

computed: {
    ...mapGetters(['cachedViews']), 
    //刷新标识
    key() {
        return this.$route.path;
    },
},

// 监听路由变化，为keepAlive：true 的路由自动添加到缓存数组
watch: {
    $route: {
        immediate: true,
        async handler(route) {
            const {
                name,
                meta: { keepAlive },
            } = route;
            if (name && keepAlive) {
                await this.$store.dispatch('addCachedView', route);
            }
        },
    },
},

// route.js
{
    path: '/place_supervision',
    component: Layout,
    children: [
        {
            path: '',
            name: 'place_supervision',
            component: () => import('@/views/place/place_supervision.vue'),
            meta: { title: '监管的场所', keepAlive: true },
        },
    ],
},

// place_supervision.vue

async beforeRouteLeave(to, from, next) {
    // 去这些界面我需要缓存
    const routerArr = ['/c页面', '/d页面'];
    if (!routerArr.includes(to.path)) {
        // 不在缓存列表中，从cachedViews缓存列表中移除
        await this.$store.dispatch('delCachedView', from);
    }
    next();
},
```

## Eruda调试工具

- 连续点击十次出现调试工具

```js
// main.js

// 增加调试信息
(function () {
    var src = "https://cdn.jsdelivr.net/npm/eruda@2.5.0/eruda.min.js";
    if (sessionStorage.getItem("active-eruda") != "true") return;
    document.write("<scr" + 'ipt src="' + src + '"></scr' + "ipt>");
    document.write("<scr" + "ipt>eruda.init();</scr" + "ipt>");
})();

// 页面中添加一个点击事件

var count = 0; //记录点击次数
// 打开调试器
showEruda() {
    if (count >= 10) {
        var eruda = sessionStorage.getItem('active-eruda');
        if (!eruda || eruda === 'false') {
            sessionStorage.setItem('active-eruda', 'true')
        } else {
            sessionStorage.setItem('active-eruda', 'false')
        }
        location.reload()
    }
    count++
},
```
## css选择器种类
```js

1、交集选择器
div.red {color:red}
<div class="red"></div>

2、并集选择器
.red,p,div,span{color:green}

3、子元素选择器
div > span {color:red}

4、后代元素选择器
div span {color:red}

5、选择下一个兄弟（后面紧挨着的一个）
p + span {color:red}

6、选择下一个兄弟（后面所有的）
p ~ span {color:red}

7、 属性名选择器
p[title] {color:red}
p[title='abc'] {color:red}
p[title^=abc] {color：red}
p[title$=abc] {color:red}
p[title*=abc] {color:red}

```
