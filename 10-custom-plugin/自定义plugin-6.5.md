Gradle学习系列之(十)——自定义Plugin

在Plugin中，我们可以向Project中加入新的Task，定义configurations和property等。我们3种方法可以自定义Plugin，这些方法和自定义Task类型的3种方法相似。
在接下来的例子中，我们将分别通过这3种方法来创建一个DateAndTimePlugin，该Plugin定义了2个Task，分别用于输出系统当前的日期和时间，另外，我们可以配置日期和时间的输出格式。

 

（一）在build.gradle文件中直接定义Plugin

和在build.gradle文件中定义Task类型一样，我们可以将对Plugin的定义直接写在build.gradle中：

```groovy
apply plugin: DateAndTimePlugin

//覆盖DateAndTimePluginExtension
dateAndTime {
    timeFormat = 'HH:mm:ss.SSS'
    dateFormat = 'MM/dd/yyyy'
}

class DateAndTimePlugin implements Plugin<Project> {
    void apply(Project project) {

        project.extensions.create("dateAndTime", DateAndTimePluginExtension)

        project.task('showTime') {
            doLast {
                println "Current time is " + new Date().format(project.dateAndTime.timeFormat)
            }
        }

        project.tasks.create('showDate') {
            doLast {
                println "Current date is " + new Date().format(project.dateAndTime.dateFormat)
            }
        }
    }
}

class DateAndTimePluginExtension {
    String timeFormat = "MM/dd/yyyyHH:mm:ss.SSS"
    String dateFormat = "yyyy-MM-dd"
}

```
 

每一个自定义的Plugin都需要实现Plugin<T>接口，事实上，除了给Project编写Plugin之外，我们还可以为其他Gradle类编写Plugin。该接口定义了一个apply()方法，
在该方法中，我们可以操作Project，比如向其中加入Task，定义额外的Property等。

在上例中，我们在DateAndTimePlugin中向Project添加了2个Task，一个名为showTime，一个名为showDate。请注意创建这2个Task所使用的不同方法，更多的创建Task的方法，请参考本系列这篇文章。

每个Gradle的Project都维护了一个ExtenionContainer，我们可以通过project.extentions进行访问，比如读取额外的Property和定义额外的Property等。
在DateAndTimePlugin中，我们向Project中定义了一个名为dateAndTime的extension，并向其中加入了2个Property，分别为timeFormat和dateFormat，
他们又分别用于showTime和showDate。在使用该Plugin时，我们可以通过以下方式对这两个Property进行重新配置：
```groovy
dateAndTime {
    timeFormat = 'HH:mm:ss.SSS'
    dateFormat = 'MM/dd/yyyy'
}
```

执行 ```gradle showDate showTime```,输出：
```shell script
> Task :showDate
Current date is 10/16/2020

> Task :showTime
Current time is 17:47:38.815

```
 

（二）在当前工程中定义Plugin

在当前工程中的buildSrc/src/main/groovy/davenkin目录下创建DateAndTimePlugin.groovy文件，将build.gradle中定义DateAndTimePlugin的代码
提取到给文件中，但是除去对DateAndTimePluginExtension的定义，因为我们将在另外一个单独的文件中定义DateAndTimePluginExtension。

```groovy
package davenkin

import org.gradle.api.Plugin
import org.gradle.api.Project

class DateAndTimePlugin implements Plugin<Project> {
    void apply(Project project) {

        project.extensions.create("dateAndTime", DateAndTimePluginExtension)

        project.task('showTime') {
            doLast {
                println "Current time is " + new Date().format(project.dateAndTime.timeFormat)
            }
        }

        project.tasks.create('showDate') {
            doLast {
                println "Current date is " + new Date().format(project.dateAndTime.dateFormat)
            }
        }
    }
}
```

再创建DateAndTimePluginExtension.groovy：

```groovy
package davenkin

class DateAndTimePluginExtension {
    String timeFormat = "MM/dd/yyyyHH:mm:ss.SSS"
    String dateFormat = "yyyy-MM-dd"
}
```
 

这里，我们将2个类文件都放在了davenkin包下。Gradle在执行时，会自动扫描buildSrc目录，并会在执行Task之前构建该目录下的内容。在build.gradle文件中，
在apply该Plugin时，我们需要声明对该Plugin的全名称，即包含包名：

```groovy
apply plugin: davenkin.DateAndTimePlugin

dateAndTime {
    timeFormat = 'HH:mm:ss.SSS'
    dateFormat = 'MM/dd/yyyy'
}

```

执行“gradle showDate showTime”，命令行输出如下：

```shell script
> Task :showDate
Current date is 10/16/2020

> Task :showTime
Current time is 17:54:08.136

```

（三）在单独的项目中创建Plugin

新建一个项目，将（二）中buildSrc目录下的内容拷贝到该项目下，定义该项目的build.gradle文件如下：
```groovy
apply plugin: 'groovy'
apply plugin: 'maven'

version = 1.0
group = 'davenkin'
archivesBaseName = 'datetimeplugin'
repositories.mavenCentral()

dependencies {
    compile gradleApi()
    compile localGroovy()
}

uploadArchives {
    repositories.mavenDeployer {
        repository(url: 'file:../lib')
    }
}
```
 
此外，我们还可以为该Plugin重新命名，如果我们希望将该Plugin命名为time，那么我们需要在src/main/resources/META-INF/gradle-plugins目录下创建名为time.properties的文件，内容如下： 
```shell script
implementation-class = davenkin.DateAndTimePlugin
```

在执行“gradle uploadArchives”时，Gradle会将该Plugin打包成jar文件，然后将其上传到上级目录下的lib目录中（../lib）。之后，在客户端的build.gradle文件中，我们需要做如下定义：

```groovy
buildscript {
    repositories {
        maven {
            url 'file://D:/Work/Learn/java/gradle/gradle-learning/10-custom-plugin/create-plugin-in-standalone-project/lib'
        } }
    dependencies {
        classpath group: 'davenkin', name: 'datetimeplugin',
                version: '1.0'
    }
}
apply plugin: 'time'

dateAndTime {
    timeFormat = 'HH:mm:ss.SSS'
    dateFormat = 'MM/dd/yyyy'
}
```

首先我们配置repository以执行lib目录，然后声明对DateAndTimePlugin的依赖，再apply该Plugin，此时我们应该使用“time”作为该Plugin的名称，最后对该Plugin进行配置。