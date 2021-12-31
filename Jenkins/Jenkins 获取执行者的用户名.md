# Jenkins 获取执行者的用户名

 jenkins执行job时可以看到console output中第一行打印出了当前执行人。我们有时候需要获取这个构建用户参数。查看jenkins环境变量，发现并没有这个参数。查看jenkins环境变量信息：http://localhost:8080/**env-vars.html。**

## 安装 build-user-vars-plugin.hpi 插件来实现

点击 [下载地址](./https://updates.jenkins-ci.org/down load/plugins/build-user-vars-plugin/) 即可下载到本地

接着在插件管理页面点击高级，上传刚下好的插件，重启jenkins即可。

## 使用

方式一、在任务配置页面，`Build Environment`中，勾选 `Set jenkins user build variables`复选框，即可获取登陆的用户信息。

方式二、在流水线中使用以下方式：

```
node(){
    wrap([$class: 'BuildUser']){
        def user = env.BUILD_USER
        currentBuild.description=user
        echo "BUILD_USER: ${user}"
    }
}
```

即可在输出界面看到用户名



## 参考链接

[插件github链接](./https://github.com/jenkinsci/build-user-vars-plugin)

