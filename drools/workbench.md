### 2020年5月25日
@小仙女：感觉到最近你的心情不太好，好想在你身边陪伴你，给你煮红糖水。不用太焦虑，我会一直陪伴你、支持你、疼爱你，努力让生活越过越好。我爱你，宝贝。

### 安装
1. workbench的数据：在目录$WORKING_DIRECTORY/.niogit，例如：wildfly-14.0.1.Final/bin/.niogit，它可以通过系统变量-Dorg.uberfire.nio.git.dir修改
2. 在生产环境，要给数据存储目录做备份

### 使用
1. workbench可以创建Fact，指定包名。也可以从已有的jars中导入使用。
2. 一个创建好的规则项目，可以build and deploy到workbench自带的maven仓库上，更改项目的pom文件可以改仓库地址。
3. workbench有自带的权限管理，通过$JBOSS_HOME/bin/add-user.sh可动态添加用户;角色有admin，analyst,developer,manager,user
4. workbench的结构式1个space对多个project。一个project只能对应一个space。space一般对应一个业务部门。project对应一个业务项目。
5. workbench底层默认使用git来管理space和project的。一个project就是一个Git项目，可以有不同的分支，可以看历史提交版本。
6. workbench可以导入jar到自己的maven仓库，仓库路径是$WORKING_DIRECTORY/repositories/kie，可以通过系统变量-Dorg.guvnor.m2repo.dir修改
7. workbench编辑规则文件的时候，同一时间只有一个用户能编辑规则文件。

