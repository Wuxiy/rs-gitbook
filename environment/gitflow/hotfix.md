# 线上bug修复(hotfix分支)


![branchs](./images/branchs.jpg)

- 蓝色圆点所在的线为源码的主线（master）
- 蓝色方形指向的节点是每一个发布版本的标签（tag）
- 紫色圆点所在的线为开发分支线（develop）
- 红色圆点所在的线为发布版本bug修复线（hotfix）

一般bug修复的流程都是测试人员会首先创建jira任务，然后开发人员会依据该任务编号从master分支拉取代码进行bug修复。

需要强调的是，master分支是唯一能够真正反映线上环境情况的分支，同时，master分支不支持任何push操作。只允许从hotfix分支或者release分支合并。

##### 1. 负责修复故障的开发人员基于master分支创建修复分支, 命名规则参考分支命名规则:[分支管理说明](branch_manage.html)

```start
git checkout -b hotfix/jira003 remotes/origin/master
```

##### 2. 工程师推送该分支到服务器上，以便协作开发，以及开发完之后合并到master，develop分支中进行进一步开发环境测试。

```push
git push --set-upstream origin hotfix/jira003
```

##### 3. 此时工程师可以开始bug修复，并提交修改到hotfix/jira003分支，当修复完成并自测后，可以发起merge request到master分支，此处需要进行代码评审，评审通过之后可以完成合并，然后集成到相关验证环境，测试完成之后，完成上线工作，所有的合并流程参考[new feature的合并流程](start_new_feature.html)。