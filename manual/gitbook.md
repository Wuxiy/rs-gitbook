# gitbook操作手册

#### 第一步，安装nodejs：
 * mac和windows版本安装[下载地址](https://nodejs.org/en/#download)
 * linux通过 `yum -y install nodejs` 安装好后`node -v` `npm -v`查看版本号,能查看到版本号说明nodejs和npm安装OK
 * mac用户也可以通过`homebrew`安装(_推荐方式_)
 1. 首先安装`homebrew`:
  * 可以通过`brew -v` 来看是否安装了`homebrew`,如果能正确显示`homebrew`的版本号，说明homebrew已安装
  * 如果没有安装`homebrew`，下发命令安装即可
        
        
    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" 
       
         
  2. 安装`homebrew`成功后，在终端中安装node，命令如下：
  
      
       brew link node
       
       brew uninstall node
       
       brew install node
       
       
#### 第2步 npm安装gitbook
 * 使用npm安装需要耐心等待,也可以使用淘宝cnpm安装,速度快 `npm install -g gitbook-cli`
 
#### 第3步 使用gitbook

`gitbook`需要2个基本文件：

* README.md
* SUMMARY.md

README.md是关于你的书的介绍，而SUMMARY.md中则包含了书目，即章节结构，它的格式大致是：

```
*  [第1章](c1.md)  
  *  [第1节](c1s1.md)  
  *  [第2节](c1s2.md)  
*  [第2章](c2.md)
```

剩下的东西就很好理解了，你只需要编写相应章节即可。在编辑完`README.md`和`SUMMARY.md`后，你可以运行以下命令：

    gitbook serve
    
`gitbook`首先把你的`markdown`文件编译为HTML文件，并根据`SUMMARY.md`生成书的目录。所有生存的文件都保存在当前目录下的一个名为`_book`的子目录中。完成这些工作后，`gitbook`会作为一个HTTP Server运行，并在4000端口监听HTTP请求。

运行以上命令后，打开浏览器，在地址栏输入：`http://localhost:4000`即可看到你的书页了

#### gitbook的插件支持
`gitbook`可以生成HTML，因此它支持一些外部的JavaScript文件嵌入到HTML中，例如`Google`统计、`Disqus`评论系统等。
比如添加一个评论，建立一个book.json文件，其格式如下：
{  "plugins":  ["disqus"],  "pluginsConfig":  {  "disqus":  {  "shortName":  "NAME-FROM-DISQUS"  }  }  }
把上面的NAME-FROM-DISQUS修改为你在Disqus上的项目名即可。
运行命令：

    gitbook install
    gitbook serve

安装插件并启动.

#### gitbook样式调整

如果发现页面表格被压缩，出现显示不全等现象，添加这个样式到页面最顶部
```
<style>
    .page-inner {
        max-width: 2000px !important;
    }
</style>
```