Gradle从入门到提高系列（一）——入门	

和Maven一样，Gradle只是提供了构建项目的一个框架，真正起作用的是Plugin。Gradle在默认情况下为我们提供了许多常用的Plugin，其中便包括构建Java项目的Plugin，还有War，Ear等。与Maven不同的是，Gradle不提供内建的项目生命周期管理，只是Java的Plugin向Project中添加了许多Task，这些Task依次执行，为我们营造了一种如同Maven般项目构建周期。

现在我们都在谈领域驱动设计，而Gradle本身的领域对象主要有Project和Task。Project为Task提供了执行上下文，所有的Plugin要么向Project中添加了用于配置的Property，要么向Project中添加不同的Task。一个Task表示一个逻辑上较为独立的执行过程，比如编译Java源代码，拷贝文件，打包Jar文件，甚至可以是执行一个系统命令或者调用Ant。另外，一个Task可以读取和设置Project的Property以完成特定的操作。

让我们来看一个最简单的Task，创建一个build.gradle文件，内容如下：
```shell script
task helloWorld << {
println "Hello World!"
}
```
~~这里的“<<”表示向helloWorld中加入执行代码，其实就是groovy代码。Gradle向我们提供了一整套DSL，所以在很多时候我们写的代码似乎已经脱离了groovy，但是在底层依然是执行的groovy。比如上面的task关键字，其实就是一个groovy中的方法，而大括号之间的内容则表示传递给task方法的一个闭包。除了“<<”之外，我们还很多种方式可以定义一个Task，我们将在系列后续的文章中讲到。~~

在gradle5.0以后的版本中 ，“<<”的写法已经舍弃，可以查看[官方文档](https://docs.gradle.org/current/userguide/upgrading_version_4.html#changes_5.0),上面的helloWorld可以写成下面的形式：

```shell script
task helloWorld {
    doLast {
        println "Hello World!"
    }
}
```


在与build.gradle相同的目录下执行：
```shell script
gradle helloWorld
```
命令行输出如下：
```shell script
> Task :helloWorld
Hello World!

BUILD SUCCESSFUL in 699ms
```

在默认情况下，Gradle将当前目录下的build.gradle文件作为项目的构建文件。在上面的例子中，我们创建了一个名为helloWorld的Task，在执行gradle命令时，我们指定执行该helloWorld Task。这里的helloWorld是一个DefaultTask类型的对象，这也定义一个Task时的默认类型，当然我们也可以显式地声明一个Task的类型，甚至可以自定义一个Task类型（我们将在本系列的后续文章中讲到）。

比如，我们可以定义一个用于文件拷贝的Task：
```shell script
task copyFile(type: Copy) {
    from 'xml'
    into 'destination'
}
```
以上copyFile将xml文件夹中的所有内容拷贝到destination文件夹中。这里的两个文件夹都是相对于当前Project而言的，即build.gradle文件所在的目录。

Task之间可以存在依赖关系，比如taskA依赖于taskB，那么在执行taskA时，Gradle会先执行taskB，然后再执行taskB。声明Task依赖关系的一种方式是在定义一个Task的时候：

```shell script
task taskA(dependsOn:taskB) {
    doLast{
        println "executing taskA"
    }
}

```


Gradle在默认情况下为我们提供了几个常用的Task，比如查看Project的Properties，显示当前Project中定义的所有Task等。可以通过一下命令查看Project所有的Task：
```shell script
gradle tasks
```


输出如下：
```shell script

------------------------------------------------------------
Tasks runnable from root project
------------------------------------------------------------

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project '1-gradle-first-try'.
components - Displays the components produced by root project '1-gradle-first-try'. [incubating]
dependencies - Displays all dependencies declared in root project '1-gradle-first-try'.
dependencyInsight - Displays the insight into a specific dependency in root project '1-gradle-first-try'.
dependentComponents - Displays the dependent components of components in root project '1-gradle-first-try'. [incubating]
help - Displays a help message.
model - Displays the configuration model of root project '1-gradle-first-try'. [incubating]
outgoingVariants - Displays the outgoing variants of root project '1-gradle-first-try'.
projects - Displays the sub-projects of root project '1-gradle-first-try'.
properties - Displays the properties of root project '1-gradle-first-try'.
tasks - Displays the tasks runnable from root project '1-gradle-first-try'.

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>

BUILD SUCCESSFUL in 671ms

```

可以看到，除了我们自己定义的copyFile和helloWorld (执行命令```gradle tasks --all``` 可以查看 )之外，Gradle还默认为我们提供了dependencies,projects和properties等Task。
dependencies用于显示Project的依赖信息，projects用于显示所有Project，包括根Project和子Project，而properties则用于显示一个Project所包含的所有Property。

在默认情况下，Gradle已经为一个Project添加了很多Property，我们可以调用以下命令进行查看：
```shell script
gradle properties
```


输出如下：
```shell script
> Task :properties

------------------------------------------------------------
Root project
------------------------------------------------------------

allprojects: [root project '1-gradle-first-try']
ant: org.gradle.api.internal.project.DefaultAntBuilder@b62bcdb
antBuilderFactory: org.gradle.api.internal.project.DefaultAntBuilderFactory@31305b10
artifacts: org.gradle.api.internal.artifacts.dsl.DefaultArtifactHandler_Decorated@7f8c8566
asDynamicObject: DynamicObject for root project '1-gradle-first-try'
baseClassLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@3eb67a30
buildDir: D:\Work\Learn\java\gradle\gradle-learning\1-gradle-first-try\build
buildFile: D:\Work\Learn\java\gradle\gradle-learning\1-gradle-first-try\build.gradle
buildPath: :
buildScriptSource: org.gradle.groovy.scripts.TextResourceScriptSource@202c9d59
buildscript: org.gradle.api.internal.initialization.DefaultScriptHandler@13f7aa97
childProjects: {}
class: class org.gradle.api.internal.project.DefaultProject_Decorated
classLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@84bba3a
components: SoftwareComponentInternal set
configurationActions: org.gradle.configuration.project.DefaultProjectConfigurationActionContainer@5a126f9
configurationTargetIdentifier: org.gradle.configuration.ConfigurationTargetIdentifier$1@4123a541
configurations: configuration container
convention: org.gradle.internal.extensibility.DefaultConvention@6f2e0ce0
copyFile: task ':copyFile'
defaultTasks: []
deferredProjectConfiguration: org.gradle.api.internal.project.DeferredProjectConfiguration@3ae1f155
dependencies: org.gradle.api.internal.artifacts.dsl.dependencies.DefaultDependencyHandler_Decorated@36fb8e1e
dependencyLocking: org.gradle.internal.locking.DefaultDependencyLockingHandler_Decorated@6a2e3c2d
depth: 0
description: null
displayName: root project '1-gradle-first-try'
ext: org.gradle.internal.extensibility.DefaultExtraPropertiesExtension@75787484
extensions: org.gradle.internal.extensibility.DefaultConvention@6f2e0ce0
fileOperations: org.gradle.api.internal.file.DefaultFileOperations@40264827
fileResolver: org.gradle.api.internal.file.BaseDirFileResolver@2ac7b543
gradle: build '1-gradle-first-try'
group:
helloWorld: task ':helloWorld'
identityPath: :
inheritedScope: org.gradle.internal.extensibility.ExtensibleDynamicObject$InheritedDynamicObject@31132b62
layout: org.gradle.api.internal.file.DefaultProjectLayout@779a590
listenerBuildOperationDecorator: org.gradle.configuration.internal.DefaultListenerBuildOperationDecorator@756ca228
logger: org.gradle.internal.logging.slf4j.OutputEventListenerBackedLogger@62167c6b
logging: org.gradle.internal.logging.services.DefaultLoggingManager@4af61370
model: project :
modelIdentityDisplayName: null
modelRegistry: org.gradle.model.internal.registry.DefaultModelRegistry@e96a9b
modelSchemaStore: org.gradle.model.internal.manage.schema.extract.DefaultModelSchemaStore@1004f54c
module: org.gradle.api.internal.artifacts.ProjectBackedModule@6095f407
mutationState: project :
name: 1-gradle-first-try
normalization: org.gradle.normalization.internal.DefaultInputNormalizationHandler_Decorated@41a03603
objects: org.gradle.api.internal.model.DefaultObjectFactory@3b37d0d4
parent: null
parentIdentifier: null
path: :
pluginManager: org.gradle.api.internal.plugins.DefaultPluginManager_Decorated@70c1d89b
plugins: [org.gradle.api.plugins.HelpTasksPlugin@47d8ed5a, org.gradle.buildinit.plugins.BuildInitPlugin@708cb8c, org.gradle.buildinit.plugins.WrapperPlugin@23c0ac9]
processOperations: org.gradle.process.internal.DefaultExecActionFactory$DecoratingExecActionFactory@7c471f15
project: root project '1-gradle-first-try'
projectConfigurator: org.gradle.api.internal.project.BuildOperationCrossProjectConfigurator@7cf58ffa
projectDir: D:\Work\Learn\java\gradle\gradle-learning\1-gradle-first-try
projectEvaluationBroadcaster: ProjectEvaluationListener broadcast
projectEvaluator: org.gradle.configuration.project.LifecycleProjectEvaluator@3de06f43
projectPath: :
projectRegistry: org.gradle.api.internal.project.DefaultProjectRegistry@5fe85074
properties: {...}
providers: org.gradle.api.internal.provider.DefaultProviderFactory_Decorated@1b93432b
repositories: repository container
resources: org.gradle.api.internal.resources.DefaultResourceHandler@4b4995fc
rootDir: D:\Work\Learn\java\gradle\gradle-learning\1-gradle-first-try
rootProject: root project '1-gradle-first-try'
script: false
scriptHandlerFactory: org.gradle.api.internal.initialization.DefaultScriptHandlerFactory@756d71a1
scriptPluginFactory: org.gradle.configuration.ScriptPluginFactorySelector@689bf488
serviceRegistryFactory: org.gradle.internal.service.scopes.ProjectScopeServices$$Lambda$1168/0x00000008018a7840@699068ab
services: ProjectScopeServices
standardOutputCapture: org.gradle.internal.logging.services.DefaultLoggingManager@4af61370
state: project state 'EXECUTED'
status: release
subprojects: []
taskThatOwnsThisObject: null
tasks: task set
version: unspecified

BUILD SUCCESSFUL in 789ms

```

在以上Property中，allprojects表示所有的Project，这里之包含一个根Project，在多项目构建中，它将包含多个Project；buildDir表示构建结果的输出目录；
我们自己定义的helloWorld和copyFile也成为了Project中的Property。另外，Project还包括用于执行Ant命令的DefaultAntBuilder（Property名为ant）和Project的描述属性description。



