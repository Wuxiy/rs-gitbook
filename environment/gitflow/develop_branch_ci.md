# develop分支集成

![branchs](./images/branchs.jpg)

当一个或者多个完整的功能通过代码评审并合并到开发分支以后，具备合并权限的开发工程师可以将develop分支上的项目部署到开发服务器上，供前端工程师或其他需要联调的工程师调用。

develop分支在构建的时候，[版本号维护方式](version_number.html)中的约定，develop分支构件出的版本会自动构件为 x.y.z-develop格式的构件，并打出相应镜像提交到harbor仓库。

同时，develop环境一般会自动部署，方便开发者调用调试。