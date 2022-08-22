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

### window.URL.createObjectURL

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

