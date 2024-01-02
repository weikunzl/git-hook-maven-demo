# git-hook-maven-demo

## Setup
1. 创建githook模板文件“prepare-commit-msg”，放在.githooks目录下。
```shell
#!/bin/sh

commitMsgFile=$1
commitSource=$2
commitMsgContent=$(cat "$commitMsgFile")
regString='^\[[0-9]{6,10}\](build|ci|docs|feat|fix|perf|refactor|style|test):.*$'

echo commitMsgFile "${commitMsgFile}"
echo commitSource "${commitSource}"

# merge 不做此校验
if [ "${commitSource}" = merge ];then
    exit 0
fi

# 检验是否符合格式
if [[ $commitMsgContent =~ $regString ]]
then
    exit 0
fi

# 不符合格式进行提示
echo "------------------"
echo "您的提交信息不符合规范格式： '$commitMsgContent'"
echo "正确格式为：[故事编号]标识:commit message"
echo "您可以使用一下标识:"
echo "build :影响构建系统或外部依赖关系的更改（示例范围：pom，npm)"
echo "ci : 更改我们的持续集成文件和脚本（示例范围：Jenkins)"
echo "docs : 仅文档更改"
echo "feat : 一个新功能"
echo "fix : 修复错误"
echo "perf : 改进性能的代码更改"
echo "refactor : 代码更改，既不修复错误也不添加功能"
echo "style : 不影响代码含义的变化（空白，格式化，缺少分号等）"
echo "test : 添加缺失测试或更正现有测试"

exit 1
```
2. pom.xml添加插件参考下面代码。
3. git-build-hook-maven-plugin 添加installHooks >  prepare-commit-msg 节点指定模板文件。*** 插件会在执行 ```mvn install``` 是自动将文件copy到项目的.git/hooks项目***
4. exec-maven-plugin 添加权限配置，通过命令（chmod 755）给文件配置上可执行权限。
```xml
<plugins>
            <plugin>
                <groupId>com.rudikershaw.gitbuildhook</groupId>
                <artifactId>git-build-hook-maven-plugin</artifactId>
                <version>3.4.1</version>
                <configuration>
                    <installHooks>
                        <prepare-commit-msg>.mvn/.githooks/prepare-commit-msg</prepare-commit-msg>
                    </installHooks>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>install</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>3.1.0</version>
                <executions>
                    <execution>
                        <id>script-chmod</id>
                        <phase>package</phase>
                        <goals>
                            <goal>exec</goal>
                        </goals>
                        <configuration>
                            <executable>chmod</executable>
                            <arguments>
                                <argument>755</argument>
                                <argument>${basedir}/.mvn/.githooks/prepare-commit-msg</argument>
                            </arguments>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
```
## 使用与验证方法
Step 1. 配置完成后执行 mvn install，使命令生效。

Step 2. 提交信息填写格式为 ```first commit``` ，会出现提交错误信息，证明githook已经正常工作。

Step 3. 提交信息填写格式为 ```^\[[0-9]{6,10}\](build|ci|docs|feat|fix|perf|refactor|style|test):.*$'``` commit顺利完成正常提交。
例如：[000001]feat: first commit
