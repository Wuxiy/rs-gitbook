# 代码提交流程

## 开发人员流程

1. 开发人员在自己的分支上编写代码，如果要依赖一些环境，可以自行启动一些容器并完成依赖构建。
2. 开发人员认为一个完成的功能完成后，merge代码到develop分支，提交的同时，需要把pom.xml里面的版本号最后一段+1并进一步自测；如果是前端人员，需要把项目src目录下的version.xml里面的版本最后一段+1，以保证版本号区别和未来的溯源。
3. 自测通过后，如果计划提交到测试环境让测试人员测试，则由具有合并到master分支的的人员进行code review，并把代码合并到master分支，合并时需要注意git注释，需要说明实现的功能(jira版本号)，以及该提交可能会影响到的范围。合并的同时需要将master分支pom.xml文件最后一段值+1；如果是前端人员，需要把项目src目录下的version.xml里面的版本最后一段+1，以保证版本号区别和未来的溯源。
4. 代码合并到master分支之后，由具有构建镜像权限的人员使用CI/CD工具（目前是jenkins pipeline)进行构建，并检查构建结果，是否正确上传到harbor仓库等。
5. 具有构建权限的人员需要在确认4正确完成之后，邮件通知测试人员上面的所有构建版本号，以便测试人员进行测试。

## 测试人员流程

1. 测试人员在收到开发人员提交的测试版本之后，如果需要更新测试环境，则需要更新目前测试环境的版本编号，目前的方式是更新gitlab仓库里的对应项目，修改版本编号并提交。
2. 测试人员在测试环境的jenkins里面构建测试环境。
3. 测试人员在系统后台检查版本号是否正确，如果正确，则说明正确的程序版本已经部署到了测试环境。
4. 测试人员按照测试流程验证功能。

# 版本号说明

1. 目前版本号为x.y.z三段格式，后面跟-DEVELOP或者-RELEASE。
2. 在最终上线正式版上线之前，x会一直为0.
3. 目前每一个sprint, y递增。
4. z为最小的版本号，每次代码合并均需要递增。