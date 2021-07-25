

title: 高效工具分享
categories:
  - 工具
tags: 工具
date: 2021-07-01 22:52:21
---
# 1. 简介


> 分享一些 `chrome` 脚本, 及创建教程, 
> 利用操作系统的搜索引擎, 可以快速打开你经常去的网站
> 同理, 你也可以用此方法, 去简化更多事情



# 2. 演示


![2021-07-01 15.40.29.gif](https://www.caijy.top//2021-07-01%2015.40.29.gif)

<!--more-->

# 3. 命令


## 3.1 使用 `shell` 带的 `open -a` 命令


```bash
open -a /Applications/Google\ Chrome.app https://www.bilibili.com/
```

> 解释:
>
> `open -a` 打开指定的 app
> `Google\ Chrome.app`
> chrome 名字有空格, \ 避免空格, 同时也可以加引号使用 `open -a "Google Chrome.app"`
> app 后面跟 网站地址, 即可

<img src="https://www.caijy.top//%E6%88%AA%E5%B1%8F2021-07-11%20%E4%B8%8B%E5%8D%8812.13.45.png" alt="截屏2021-07-11 下午12.13.45" style="zoom:50%;" />

## 3.2 使用 chrome 自带命令


> `/Applications/Google Chrome.app/Contents/MacOS` 文件夹下有一个 `Google Chrome`
> 这是一个可执行文件, 它也可以直接打开指定网站, 也有一些参数可以使用
> 具体看下面

<img src="https://www.caijy.top//%E6%88%AA%E5%B1%8F2021-07-11%20%E4%B8%8B%E5%8D%8812.19.36.png" alt="截屏2021-07-11 下午12.19.36" style="zoom:50%;" />

<img src="https://www.caijy.top//%E6%88%AA%E5%B1%8F2021-07-11%20%E4%B8%8B%E5%8D%8812.20.22.png" alt="截屏2021-07-11 下午12.20.22" style="zoom:50%;" />


```bash
# 直接打开网站, 如果此时已有 chrome 窗口, 则以 tag 形式加入
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome https://www.baidu.com

# 新窗口打开网站
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --new-window https://www.baidu.com

# 无痕模式打开网站, 如果此时已有 chrome 无痕窗口, 则以 tag 形式加入
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --incognito https://www.baidu.com

# 新窗口, 无痕模式打开网站
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --new-window --incognito https://www.baidu.com
```


# 4. 脚本文件创建


- 创建一个文件, 以 `.sh` 结尾
- 右键文件 --> 显示简介 --> 打开方式 --> 选择 **终端 **(相当于默认打开软件的设置)
- 然后命令行给予该文件可执行权限, 也可以给该文件夹下所有的文件可执行权限
- command + space 聚焦搜索 该文件, 打开即可



> 文件创建, 打开方式

><img src="https://www.caijy.top//%E6%88%AA%E5%B1%8F2021-07-11%20%E4%B8%8B%E5%8D%8812.24.37.png" alt="截屏2021-07-11 下午12.24.37" style="zoom:50%;" />


> 可执行权限

<img src="https://www.caijy.top//%E6%88%AA%E5%B1%8F2021-07-11%20%E4%B8%8B%E5%8D%8812.26.34.png" alt="截屏2021-07-11 下午12.26.34" style="zoom:50%;" />


# 5. 脚本分享


- 谷歌翻译
   - ```java
     open -a /Applications/Google\ Chrome.app https://translate.google.cn/
     ```
- 有道词典
   - ```java
     open -a /Applications/Google\ Chrome.app https://www.youdao.com/
     ```
- 百度地图
   - ```
     open -a /Applications/Google\ Chrome.app https://www.youdao.com/
     ```

json 解析工具网站

```
open -a /Applications/Google\ Chrome.app https://www.bejson.com/
```

# 7. alfred


> 使用alfred可以达到同样的效果，并且不需要脚本配置



#### [参考alfred配置](https://www.jianshu.com/p/e9f3352c785f)，看 Web Search部分即可。