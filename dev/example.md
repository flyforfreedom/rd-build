## 示例

### 场景

下面以一个公司即将启动的`项目A`来演示整个持续交付的流程。

##### 项目背景

公司启动了一个项目A，最终除了有对用户的交互的h5，还需要开发相关的移动端app。

##### 相关人员

* 小胖（项目产品）
* 小黑（服务端开发人员）
* 小白（前端开发人员）
* 小明（移动端开发人员）

### 项目准备

1. 项目相关的**所有人员**在一起通过用户故事地图进行需求讨论。最终确定项目的各个阶段以及每个阶段需要做的事情。

2. 在gitlab上创建项目A的群组，群组名以项目名命名即可。在群组下创建相关的项目，包括：

   * `client`：用于处理前端项目
   * `service`：用于处理服务端项目
   * `app`：用于处理移动端项目
   * `product`：用于产品提出需求

   > 创建product的项目的原因是因为产品人员并不清楚当前任务应该分配给前端、服务端还是移动端，所以单独创建了个项目用户产品提出需求，这个项目对于群组下的所有人都有查看权限。

3. 产品将讨论后的最终结果录入到product项目，需要对应的各个阶段创建相应的[里程碑](/dev/issue.md)，在里程碑下创建本阶段的[任务(issues)](/dev/issue.md)。

   > 如果一个项目没有进行区分，如只有一个服务端，那么可以直接在服务端项目里创建需求即可。

   ![创建里程碑](/images/gitlab/1.创建里程碑.png)

   ![创建里程碑](/images/gitlab/2.新建问题.png)

4. 开发人员将产品创建的任务进行认领、分配，在自己的项目（client、service、app）下创建技术方面的需求并通过`#`与product项目下的需求进行关联。

此时我们的需求已经确认、分配完毕，下一步进入开发阶段。

### 项目开发

以服务端开发进行举例，其他开发过程类似。

#### 本地开发环境构建

首先根据[本地环境构建](/dev/develop.md)方法构建本地开发环境，生成项目所需的初始代码、初始数据和环境。

#### 初始化代码仓库

将初始代码提交到代码仓库中，创建master分支。

```shell
cd a_service
git init
git remote add origin git@gitlab.nw.com:A/service.git
git add .
git commit
git push -u origin master
```

gitlab会自动将master分支设置为默认分支、保护分支。

> 因为项目是首次开发，如果是非第一次开发可以跳过以上操作。

#### 分支开发

首先创建相关的功能分支进行开发。可以通过gitlab问题页面进行创建，如下图所示：

![需求分支创建](/images/gitlab/3.新建分支.png)

新建的分支名称为 ：2-credit-task 由“问题编号-问题标题”组成。也可以通过git命令行创建分支：

```shell
git checkout -b 2-credit-task master
```

![创建feature分支.png](/images/branch/创建feature分支.png)
所有的需求分支都是基于`master`分支的。创建分支后，他们用老套路添加提交到各自功能分支上：编辑、暂存、提交：

```shell
git status
git add
git commit
```

#### 分支合并

小黑完成开发后，自己进行自测都OK了。如果团队使用`Pull Requests`，这时候可以发起一个用于合并到`master`分支。如下图所示：

![合并分支](/images/gitlab/4.新建合并请求.png)

否则他可以直接合并到他本地的`master`分支后`push`到中央仓库：

```shell
## 合并分支
git pull origin master
git checkout master
git merge 2-credit-task
git push
## 合并成功并验收完成后删除分支
git branch -d 2-credit-task
```

![img](/images/branch/合并分支.png)

小黑在发起合并请求或者push请求时可以添加 `Closes #2`来关闭相关问题。小黑在将代码push到`master`后就开始触发了集成测试（CI）,如果测试不通过小黑需要在需求分支上修改代码后，继续发起合并请求。当集成测试通过后，会自动构建当前版本镜像，上传到线下镜像仓库，并更新稳定测试环境，小黑的当前功能已经开发完成了。

#### 版本发布

当大家都开发完成自己的需求后，需要发布生产。进入gitlab，找到需要发布版本的CIjob，点击push_online按钮，完成发布。如下图所示：

![发布操作](/images/gitlab/6.发布操作.png)

#### 添加版本标记

添加tag的操作可以选择版本发布前后皆可。主要在一个里程碑的完成代表一个大的版本迭代，一个里程碑中主要细节点也可以添加小版本的tag。UI操作如下图：

![添加tag](/images/gitlab/5.添加tag.png)

也可以进行命令行操作：

```shell
## 给master分支添加tag
git pull origin master
## 如果是补标签则在后面加上版本号即可，例如：git tag -a v1.0 9fceb02
git tag -a v1.0 -m '版本1.0发布'
## 此处注意git push并不会把标签传送到远端服务器上，只有通过显式命令才能分享标签到远端仓库。
git push origin v1.0
```

