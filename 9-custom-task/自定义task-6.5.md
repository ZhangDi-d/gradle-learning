Gradle学习系列之(九)——自定义Task类型

在前面的文章中我们讲到，Gradle本身只是一个架子，真正起作用的是Task和Plugin。要真正了解Task和Plugin的工作机制并熟练运用，学会自定义Task类型和Plugin是大有裨益的。

Gradle中的Task要么是由不同的Plugin引入的，要么是我们自己在build.gradle文件中直接创建的。在默认情况下，我们所创建的Task是DefaultTask类型，
该类型是一个非常通用的Task类型，而在有些时候，我们希望创建一些具有特定功能的Task，比如Copy和Jar等。还有时候，我们希望定义自己创建的Task类型，
在本文中，我们以定义一个简单的HelloWorldTask为例，讲解如何自定义一个Task类型，并且如何对其进行配置。

在Gradle中，我们有3种方法可以自定义Task类型。
 

（一）在build.gradle文件中直接定义

我们知道，Gradle其实就是groovy代码，所以在build.gradle文件中，我们便可以定义Task类。

```groovy
class HelloWorldTask extends DefaultTask {
    @Optional
    String message = 'I am davenkin'

    @TaskAction
    def hello(){
        println "hello world $message"
    }
}

task hello(type:HelloWorldTask)


task hello1(type:HelloWorldTask){
   message ="I am a programmer"
}
```


在上例中，我们定义了一个名为HelloWorldTask的Task，它需要继承自DefaultTask，它的作用是向命令行输出一个字符串。@TaskAction表示该Task要执行的动作，
即在调用该Task时，hello()方法将被执行。另外，message被标记为@Optional，表示在配置该Task时，message是可选的。在定义好HelloWorldTask后，
我们创建了两个Task实例，第一个hello使用了默认的message值，而第二个hello1在创建时重新设置了message的值。

在执行hello时，命令行输出如下：

```shell script

> Task :hello
hello world I am davenkin

```

 

在执行hello1时，命令行输出如下：

```shell script
> Task :hello1
hello world I am a programmer
```
 

（二）在当前工程中定义Task类型

在（一）中，我们在build.gradle中直接定义了Task的类型，这样将Task的定义和使用混在一起。在需要定义的Task类型不多时，我们可以采用这种方法，
但是在项目中存在大量的自定义Task类型时，这就不见得是中好的做法了。一种改进方法是在另外的一个gradle文件中定义这些Task，然后再apply到build.gradle文件中。
这里，我们将使用另一种方法：在buildSrc目录下定义Task类型，Gradle在执行时，会自动地查找该目录下所定义的Task类型，并首先编译该目录下的groovy代码以供build.gradle文件使用。

在当前工程的buildSrc/src/main/groovy/davenkin目录下创建HelloWorldTask.groovy文件，将（1）中对HelloWorldTask的定义转移到该文件中：

```groovy
package davenkin
import org.gradle.api.*
import org.gradle.api.tasks.*

class HelloWorldTask extends DefaultTask {
    @Optional
    String message = 'I am davenkin'

    @TaskAction
    def hello(){
        println "hello world $message"
    }
}
```

 

这里，我们将HelloWorldTask定义在了davenkin包下，因此在build.gradle文件中引用该Task时，我们需要它的全名称：

```groovy
task hello(type:davenkin.HelloWorldTask)


task hello1(type:davenkin.HelloWorldTask){
    message ="I am a programmer"
}
```

 

以上的hello和hello1与（1）中的hello和hello1完成的功能相同。

 

（三）在单独的项目中定义Task类型

虽然（2）中的Task定义与build.gradle分离开了，但是它依然只能应用在当前工程中。如果我们希望所定义的Task能够用在另外的项目中，那么（2）中的方法便不可行的，
此时我们可以将Task的定义放在单独的工程中，然后在所有使用Task的工程中通过声明依赖的方式引入这些Task。

创建另外一个项目，将（2）中buildSrc目录下的内容考到新建项目中，由于该项目定义Task的文件是用groovy写的，因此我们需要在该项目的build.gradle文件
中引入groovy Plugin。另外，由于该项目的输出需要被其他项目所使用，因此我们还需要将其上传到repository中，在本例中，我们将该项目生成的包含了Task定义
的jar文件上传到了本地的文件系统中。最终的build.gradle文件如下：

```groovy
apply plugin: 'groovy'
apply plugin: 'maven'
apply plugin: 'idea'

version = '2.0'
group = 'dizhang'
archivesBaseName = 'hellotask'

repositories.mavenCentral()

dependencies {
    compile gradleApi()
    compile localGroovy()
}
uploadArchives {
    repositories.mavenDeployer {
        //repository(url: 'file:../lib')
        repository(url: 'file://D:/Work/Learn/java/gradle/gradle-learning/9-custom-task/create-task-in-standalone-project/lib/')
    }
}
```

 

执行“gradle uploadArchives”，所生成的jar文件将被上传到~~上级目录的lib(../lib)文件夹~~ 绝对路径 D:/Work/Learn/java/gradle/gradle-learning/9-custom-task/create-task-in-standalone-project/lib/中。

在使用该HelloWorldTask时，客户端的build.gradle文件可以做以下配置：

```groovy
buildscript {
    repositories {
        maven {
            url 'file://D:/Work/Learn/java/gradle/gradle-learning/9-custom-task/create-task-in-standalone-project/lib/'
        }

    }


    dependencies {
        classpath group: 'dizhang', name: 'hellotask', version: '2.0'
    }
}


task hello(type: davenkin.HelloWorldTask){
    message ="I am a programmer!!!!!"
}

```

 

首先，我们需要告诉Gradle到何处去取得依赖，即配置repository。另外，我们需要声明对HelloWorldTask的依赖，该依赖用于当前build文件。之后，对hello的创建与（2）中一样。