谷歌翻译退出中国导致idea中的翻译插件Translation不能使用，可以通过更改为国内的翻译api继续使用，这里讲解如何继续使用谷歌翻译。

# 1.找到能ping通的谷歌服务域名

原理就是谷歌翻译虽然停止服务，但是翻译的api仍然存在，我们找到可以ping通的谷歌域名，通过配置host文件实现恢复Translation的使用。

> 在线ping测试
>
> https://ping.chinaz.com

我这里使用 `google.cn` 获取ip，得到 `114.250.70.34` 

# 2.修改host文件

host文件目录：`C:\Windows\System32\drivers\etc` 

将映射添加到host文件末尾

```
114.250.70.34 translate.googleapis.com
```

