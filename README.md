[TOC]

- https://plugins.jetbrains.com/plugin/7299-gradle-dependencies-helper/versions
- https://plugins.jetbrains.com/plugin/10033-gradle-dependencies-and-plugins-helper/versions

### 1、gradle properties

用于指定运行时JVM参数，优先级如下：

- system properties, e.g. when -Dgradle.user.home is set on the command line.
- gradle.properties in GRADLE_USER_HOME directory.
- gradle.properties in project root directory.
- gradle.properties in Gradle installation directory.

[配置信息](https://docs.gradle.org/current/userguide/build_environment.html#sec:gradle_configuration_properties)

### 2、init script

- 它可以用来建立公司内部的配置,如定义公司内部的仓库地址。
- 它可以用来配置一些全局属性,比如配置持续集成服务器的地址等配置。
- 它可以用来提供构建所需要的用户的个人信息,如仓库或数据库的用户名和密码。
- 它可以用来定义开发者机器的环境,比如定义jdk安装在什么位置，android sdk安装在什么位置等等。
- 最重要的功能之一，它可以用来注册一些监听器。比如监听Gradle事件的发生，做一些额外的操作，例如需要对某个项目构建前和构建后做一些操作，又例如对项目的依赖做检测，检测是否含有snapshot包，在release构建中一般来说是禁止依赖snapshot包的，所以这时候就可以扔出一个异常。
- 重定向日志。我们可以将gradle默认的日志进行重定向，甚至我们可以不输出默认日志，自定义如何输出gradle产生的日志信息

具体有如下优先级方式：

- 通过命令行 -I or --init-script参数指定脚本路径  
- 在目录 `USER_HOME/.gradle/` 中创建 `init.gradle` (or `init.gradle.kts` for Kotlin) 文件
- 在目录`USER_HOME/.gradle/init.d/` 创建一个以 `.gradle`结尾的文件 (or .init.gradle.kts for Kotlin)
- 在目录`GRADLE_HOME/init.d/` 创建一个以 `.gradle`结尾的文件 (or .init.gradle.kts for Kotlin)  

`init.gradle`:

```groovy
allprojects {
    repositories {
        def ALIYUN_REPOSITORY_URL = 'https://maven.aliyun.com/repository/public/'
        // 替换 指定仓库
        all { ArtifactRepository repo ->
            if (repo instanceof MavenArtifactRepository) {
                def url = repo.url.toString()
                if (url.startsWith('https://repo1.maven.org/maven2')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL."
                    remove repo
                }
                if (url.startsWith('https://jcenter.bintray.com/')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL."
                    remove repo
                }
            }
        }
        maven {
            url ALIYUN_REPOSITORY_URL
        }
    }
}
```

阿里仓库：https://maven.aliyun.com/mvn/guide

### 3、build.gradle中的repositories的区别

- buildScript块的repositories主要是为了Gradle脚本自身的执行，获取脚本依赖插件。也就是说，buildScript是用来加载Gradle脚本自身需要使用的资源，可以声明的资源包括依赖项、第三方插件、maven仓库地址等。
- 根级别的repositories主要是为了当前项目提供所需依赖包，比如log4j、spring-core等依赖包可从mavenCentral仓库获得。
- allprojects块的repositories用于多项目构建，为所有项目提供共同的所需依赖包。而子项目可以配置自己的repositories以获取自己独需的依赖包。
- subprojects块的repositories用于配置这个项目的子项目。使用多模块项目时，不同模块之间有相同的配置，导致重复配置，可以将相同的部分抽取出来，使用配置注入的技术完成子项目的配置。根项目就像一个容器, subprojects 方法遍历这个容器的所有元素并且注入指定的配置。allprojects是对所有project的配置，包括Root Project。而subprojects是对所有Child Project的配置

### 4、gradle wrapper

https://docs.gradle.org/current/userguide/gradle_wrapper.html

`gradle wrapper <options>`

`gradle wrapper --gradle-version 6.7.1 --distribution-type all`

- `--gradle-version`：声明使用的gradle version
- `--distribution-type`：`all` or `bin`
- `--gradle-distribution-url`

升级gradle version：`./gradlew wrapper --gradle-version 6.8.2`
设置wrapper认证，用于请求需要auth的distributionUrl，如 `distributionUrl=https://username:password@somehost/path/to/gradle-distribution.zip`：
```groovy
systemProp.gradle.wrapperUser=username
systemProp.gradle.wrapperPassword=password
```

### 5、gradle生命周期

#### 5.1、初始化
> Gradle 支持单项目构建和多项目构建。
在这个阶段 Gradle 会确认哪些项目将会参与构建。Gradle 会通过 settings.gradle 确定是多项目还是单项目构建。
Gradle 会为每个项目创建 Project 实例。

#### 5.2、配置
> 在这个阶段执行在初始化阶段中确定的每一个项目的配置脚本，但是并不会执行其中的任务，只会评估任务的依赖性，根据其依赖性创建任务的有向无环图。
> Gradle引入了一个称为随需求变配置的特性，该特性使它能够在构建过程中只配置相关和必要的项目。这在大型多项目构建中非常有用，因为它可以大大减少构建时间。

#### 5.3、执行
> 在这个阶段，Gradle 会识别在配置阶段创建的任务的有向无环图。并按照他们的依赖顺序开始执行。
所有的构建工作都是在这个阶段执行的。如编译源码，生成 .class 文件，复制文件等。

### 6、dependency

#### 6.1、依赖配置（级别（scope））
[java插件定义的configurations](https://docs.gradle.org/current/userguide/java_plugin.html#tab:configurations)：

![configurations](files/imgs/java-plugin-dependency-configurations.png))

api: 使用api引入的依赖，在当前module被其他引入时，这个依赖也可以被使用，即 传递依赖
implementation: 和api不同的是，依赖不可以被使用，即 非传递依赖，只当前module内部使用

```
gradle :{module}:dependencies
# 分情况查看依赖
gradle :{module}:dependencies --configuration compile
gradle :{module}:dependencies --configuration compileOnly
gradle :{module}:dependencies --configuration runtime
gradle :{module}:dependencies --configuration testCompile
gradle :{module}:dependencies --configuration testCompileOnly
gradle :{module}:dependencies --configuration testRuntime
```

#### 6.2、依赖的形式

```groovy

implementation('commons-lang:commons-lang:2.6') {
    because '依赖说明'
}
implementation 'commons-lang:commons-lang:2.6'
implementation group: 'commons-lang', name: 'commons-lang', version: '2.6'
// 文件中的依赖
implementation files('commons-lang.jar', 'libs/commons-lang.jar')
// 文件中所有的依赖
implementation fileTree('libs')
implementation fileTree('libs') {
    include '*.jar'
}
// java-library插件提供
api 'org.springframework:spring-core:2.5', 'org.springframework:spring-aop:2.5'
runtimeOnly(
        [group: 'org.springframework', name: 'spring-core', version: '2.5'],
        [group: 'org.springframework', name: 'spring-aop', version: '2.5']
)
runtimeOnly('org.hibernate:hibernate:3.0.5') {
        transitive = true
}
runtimeOnly(group: 'org.hibernate', name: 'hibernate', version: '3.0.5') {
        transitive = true
}
runtimeOnly group: 'org.hibernate', name: 'hibernate', version: '3.0.5', transitive: true
// project module依赖
implementation project(':pure-gradle-api')

```

#### 6.3、版本配置

- [1.0,), [1.1, 2.0), (1.2, 1.5]
- []: 闭区间
- ()：开区间
- ]1.0, 2.0[ 等价于  (1.0, 2.0)
- 1.3+
- latest.integration, latest.release：the highest version


### 7、插件

- java插件：https://docs.gradle.org/current/userguide/java_plugin.html