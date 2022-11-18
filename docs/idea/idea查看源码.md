# idea如何查看源码

首先这个项目应该是基于 Maven 的，我们可以利用 Maven 下载需要查看的源码

在没有下载源码之前，用 `ctry+鼠标左键` 点进去的是 `.class` 文件而不是 `.java` 文件，那么我们想要查看 `.java` 文件应该怎么办？

## 1.设置Maven

可以设置在 Maven 加载 jar 包的时候同时加载 sources

![image-20220520091511994](https://raw.githubusercontent.com/walls1717/image/master/202210121942677.png)

这样 sources 会下载到本地仓库对应的AVG当中

这样我们 `ctry+鼠标左键` 就会查看到 `.java` 文件

## 2.直接下载

项目已经创建的话，可以直接下载 sources

![image-20220520091722282](https://raw.githubusercontent.com/walls1717/image/master/202210121942679.png)

这样也能实现查看到 `.java` 

## 3.在.class文件上方点击download sources

这样的操作不会下载所有的sources，只会下载所需要的sources