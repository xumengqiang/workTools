# workTools
工作中遇到的一些方法总结


## 判断中文

```
if(/[\u4E00-\u9FA5]/g.test(value)){console.log('不可输入汉字')}
```