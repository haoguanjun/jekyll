---
layout: post
description: create a local npm server.
---

## Create a local npm server

### npm 


### how to publish package to npm
要打包程序，首先要配好各项设置，这些设置都由程序包根目录下的package.json指定。package.json的内容必须是严格的JSON格式，也就是说：
* 字符串要用双引号括起来，而不能用单引号；
* 属性名一定要加双引号；
* 最后一个属性后千万不要多加一个逗号。
配置对象的属性很多，具体可以参阅这里，这里列一下常用的项目：
* name：程序包名，不能跟已有的程序包重复。
* version：版本号。
* description：一段简短的介绍。
* author：作者信息。包含name、email、url三项属性。
* bin：如果程序中有可执行文件（主要是命令行里面调用的），就在这里指定，可以指定多个。
* main：使用require调用本程序包时的程序入口。
* dependencies：依赖的程序包，可以指定版本号。

配置好package.json后，可以先在本地打包安装一次，测试程序运作是否正常，安装命令为：

  npm install <本地路径>

另外，还有一条潜规则要注意，如果你希望程序包中的可执行程序在Node.JS的环境中运行，那么，请在程序入口文件的最前面加上这样一行：

  #!/usr/bin/env node
  
如果没有这一行，它将以系统默认的方式打开，而不是在Node.JS的环境中运行。
要把程序包发布到npm，还需要先注册一个帐号。npm并没有提供网页版的注册向导。注册也要通过命令行来进行：
  
  npm adduser
  
执行此命令后，会依次出现输入用户名、Email、密码的提示，输入好之后等待一会儿就可以了.

  [root@~/wade/nodejs/pv-tj]# npm adduser
  Username: billfeller
  Password: 
  Email: (this IS public) 531958936@qq.com

准备工作都做好了，执行下面的命令就可以发布程序包.

  npm publish <本地路径>
  
 如果要更新程序包，只要修改一下package.json中的版本号，再重新执行发布命令就可以了。

### see also
* [如何使用npm发布Node.JS程序包](http://heeroluo.net/article/detail/103){:target="_blank"}
* [npm document](https://docs.npmjs.com/){:target="_blank"}
* [Sinopia | 从零开始搭建npm仓库](https://sanwen8.cn/p/1f0pl01.html){:target="_blank"}
* [sinopia in GitHub](https://github.com/rlidwka/sinopia#override-public-packages){:target="_blank"}
* [如何使用npm打包发布nodejs程序包](http://blog.csdn.net/billfeller/article/details/41295533){:target="_blank"}
