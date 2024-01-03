# git-hook-maven-demo
原理是利用git原本自带的hook功能。 

Git本地的hook功能是一种机制，它允许您在特定的Git操作（例如提交、推送、合并等）发生时自动触发自定义脚本或命令。这使您可以在这些操作发生之前或之后执行额外的逻辑或验证。 

Git本地的hook是以脚本的形式存在于Git仓库的.git/hooks目录中。这个目录包含了多个示例 hook 脚本文件，每个文件代表一个特定的Git操作。 常用的Git本地hook：pre-commit 、 prepare-commit-msg 、 pre-push

## 配置步骤
1. 拷贝 git hook 模板文件```.mvn/.githooks/prepare-commit-msg```，放在您的项目的.mvn/.githooks目录下。
2. 添加插件git-build-hook-maven-plugin插件。参考pom.xml中的配置
   > 添加installHooks >  prepare-commit-msg 节点指定模板文件。*** 插件会在执行 ```mvn install``` 是自动将文件复制到项目的.git/hooks项目***
3. 添加插件exec-maven-plugin插件，参考pom.xml中的配置。
   > 权限配置，通过命令（chmod 755）给文件配置上可执行权限。
4. 配置完成后执行 mvn install，使命令生效
   > prepare-commit-msg 的模板文件会被复制到```.git/hooks```目录下，使 git hook 功能生效

## Notes: 使用与验证
1. 使用错误格式提交：提交信息为 ```first commit``` ，会出现提交错误信息，证明githook已经正常工作。

2. 使用正确格式提交：提交信息为 ```[000001]feat: first commit```，成功提交
   > 格式修改```prepare-commit-msg```文件中的正则表达式。
   > 默认正则为```^\[[0-9]{6,10}\](build|ci|docs|feat|fix|perf|refactor|style|test):.*$'``` 。
