# release分支集成

![branchs](./images/branchs.jpg)

当一个或者多个完整的功能在develop分支上合并以及自测之后，如果开发人员希望提交到测试环境进行更高标准的测试，此时可以将代码合并到release分支。

合并流程为具备合并权限的工程师发起合并请求（merge request），将当前的develop分支合并到release分支。

合并相关操作可以参考[新功能开发(feature分支)](start_new_feature.html)中的分支合并流程，不过此时一般不需要再次评审代码，因为这些需要合并的代码在开发人员合并到develop分支的时候已经评审过。

release分支在构建的时候，[版本号维护方式](version_number.html)中的约定，release分支构件出的版本会自动构件为 x.y.z格式的构件，z段版本号会自动增加，并打出相应镜像提交到harbor仓库。

release分支一般不会自动部署到测试环境，需要和测试人员确定后，其中一方进行部署。