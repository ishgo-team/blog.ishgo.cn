title: 说说怎么部署静态网站到github上
date: 2013-09-18 00:01:09
author: Barbery  
categories: [其他]
tags: [github pages,部署,HTML]
---
1. 首先, 第一步当然要先到github上建立一个repo. 什么? 你找不到哪里创建?看下图   
![看图](http://ww4.sinaimg.cn/large/6915c7dcgw1e8pwx8oth8j20ax05ct8q.jpg)

2. 填写完必要的信息, 创建完成后, 需要设置一下, 才能让repo里面的html文件publish出去. 废话少说, 看图, 点击下面这个图标进入setting页面  
<!-- more -->
![setting](http://ww1.sinaimg.cn/large/6915c7dcgw1e8px21o73lj20qc0cogn8.jpg)

3. 进入设置页面后, 滚下去.. 找到下图这个位置, 点击红框按钮, 即可转为github page 模式.. 
![github page](http://ww4.sinaimg.cn/large/6915c7dcgw1e8px3q3uj5j20jk04hgm6.jpg)

4. 默认什么都不用填, 直接点continue进入下一步, 选择模板这里随便选就可以了, 点击下图这个图标, 正式生成github page
![generate github page](http://ww1.sinaimg.cn/large/6915c7dcgw1e8px6mi02kj20ql05emxq.jpg)  
------  
到这里, github page已经生成啦… 
![finish](http://ww1.sinaimg.cn/large/6915c7dcgw1e8px9oaqy6j20lp09lta1.jpg)

你会发现, 里面有了html的静态文件, 和多了一个叫gh-pages的分支, 这些都是git默认生成的..  访问你 `username.gitpage.io/repoName/`(这里username为你的github用户名, repoName为repo名称,也就是project名) 看下


`关于以后部署的话, 这里要注意一下:`   
部署上来的静态资源一定要部署到`gh-pages`这个分支上, 你访问页面时访问的就是这个分支的内容.. git只允许访问这个分支的内容, 所以千万不要默认传到master分支上了.. 还有就是github pages 上会有静态资源缓存, 所以更新上去的静态文件, 可能需要等个5~10分钟才能更新..  

`最后说说绑定域名..`  
github pages 的绑定域名非常简单, 只要在gh-pages根目录下, 创建一个`CNAME`文件, 然后里面填入你想要绑定地址.. 还是看图直观点.. 下图是我们blog的CNAME文件  
![](http://ww4.sinaimg.cn/large/6915c7dcgw1e8pxm0rr6rj20ho06kmxl.jpg)

然后别忘记还要设置域名的CNAME记录… 直接做个CNAME记录指向`username.github.io`就可以了, 例如本博客的cname记录
![](http://ww4.sinaimg.cn/large/6915c7dcgw1e8pxpypkfnj20ek019t8k.jpg)  


ok, 到这里, 大功告成, 打完收工...

    