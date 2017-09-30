# sbt入门
## 什么是sbt
sbt是一个simple build tools，可以进行scala与java的项目管理，支持增量编译，内置scala console
## 如何开始
### 安装配置初始化
先安装，安装方式这里就不写了，国内墙比较高，如果没有梯子的用户，请最好将源改成oschina的
具体做法是：
* 在用户家目录下的`.sbt`文件夹下定义一个名为`repositories`的配置文件
* 在文件中加入以下内容
```repositories
[repositories]
local
osc: http://maven.oschina.net/content/groups/public/
typesafe: http://repo.typesafe.com/typesafe/ivy-releases/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext], bootOnly
sonatype-oss-releases
maven-central
sonatype-oss-snapshots
```
### 创建项目
接着创建工程目录，或者选用ide，我通常使用idea，会自动生成工程目录夹以及原始的配置，如果非ide则需要手工在工程目录下创建`build.sbt`文件，手工创建目录，结构类似![srctree](http://7xnz7j.com1.z0.glb.clouddn.com/srctree.png)
其中main主要放工程代码，test放测试代码，然后运行sbt，命令为`sbt`进入sbt中
* 可以查看`scalaSource`来查看scala源码要求的位置，
* java的也是对应的`javaSource`
* 其中里面的配置文件放在对应的`resources`中，可运行`show unmanagedResources `查看，
* 运行`console`进入scala的Repl中

### 加入依赖
依赖加入也比较简单
可以一条条的加就像这样
```scala
name := "learnsbt"

version := "1.0"

scalaVersion := "2.11.7"

//                                      组织                    库                                   版本
libraryDependencies += "org.scala-js" % "scalajs-library_2.11" % "0.6.6"


```
具体搜索我是通过[mvnrepository.com](http://mvnrepository.com/)

当然还有一起加的方法：

```scala

name := "HbaseTest"

version := "1.0"

scalaVersion := "2.11.7"

libraryDependencies ++= Seq(
  "org.apache.hadoop" % "hadoop-common" % "2.6.1" ,
  "org.apache.hbase" % "hbase-client" % "1.1.2",
  "org.apache.hbase" % "hbase-common" % "1.1.2" ,
  "ch.qos.logback" % "logback-classic" % "1.1.3",
  "com.typesafe.scala-logging" % "scala-logging_2.11" % "3.1.0"
)

```
到这里就可以基本使用了
记得修改配置后需要`reload`或者`update`。
### 写的类运行
执行`run` ，如果有多个类就会出现一个列表，输入对应数字选择即可。
### 写的测试运行
执行`test`，方法同上`run`
这里就基本实现了小白级别的扫盲了



## 进阶
### 第一说操作符
`:=` 是给key分配一个初始表达式，覆盖原来的任何配置
`+=`是追加一个值到key
`++=`是追加一个队列，队列中放值，将值追加到key
### 再说task，里面可以定义task，task可以是执行外部的命令，先简单一点
```scala
name := "learnsbt"

version := "1.0"

scalaVersion := "2.11.7"

//定义了task的类型为Unit，key值叫helloTask
val helloTask = taskKey[Unit]("say hello")

helloTask := {
  println("Hello")
}


```

运行结果
```result
>reload
[info] Loading project definition from /home/ctao/WorkSpace/scala/learnsbt/project
[info] Set current project to learnsbt (in build file:/home/ctao/WorkSpace/scala/learnsbt/)
> helloTask
Hello
[success] Total time: 0 s, completed Jan 24, 2016 5:28:15 PM
```
当然你也可以定义taskKey的返回类型是String，例如：
```scala
//定义了task的类型为String，key值叫helloTask
val helloTask = taskKey[String]("say hello")

helloTask  := "hello"

```
运行结果是
```result
> reload
[info] Loading project definition from /home/ctao/WorkSpace/scala/learnsbt/project
[info] Set current project to learnsbt (in build file:/home/ctao/WorkSpace/scala/learnsbt/)
> helloTask
[success] Total time: 0 s, completed Jan 24, 2016 5:35:18 PM
```
奇怪，结果呢，要看结果就执行 `show taskKey`
```result
> show helloTask
[info] hello
[success] Total time: 0 s, completed Jan 24, 2016 5:36:29 PM

```
下面我们定义多个task，task之间是*并行执行的* ，这点很关键，看下面的例子
先定义三个task，第一个是hello，第二个是hi，第三个是bye
```scala
val helloTask = taskKey[String]("say hello")

val hiTask = taskKey[String]("say hi")

val byeTask = taskKey[String]("say bye")

```
首先，我们让三个task内部执行只是休眠2s，如果是并行那么结果会是2s，如果串行那么就是6s

```scala
val helloTask = taskKey[String]("say hello")

val hiTask = taskKey[String]("say hi")

val byeTask = taskKey[String]("say bye")

val runAllTask = taskKey[Unit]("run all")

runAllTask := {
  println(helloTask.value)
  println(hiTask.value)
  println(byeTask.value)

}

helloTask := {
  val hello = "hello"
  println(s"hello task run : $hello")
  Thread sleep 2000
  hello
}


hiTask := {
  val hi = "hi"
  println(s"hi task run : $hi")
  Thread sleep 2000
  hi
}

```

```result

> runAllTask
hi task run : hi
bye task run : bye
hello task run : hello
hello
hi
bye
[success] Total time: 2 s, completed Jan 24, 2016 5:46:12 PM

```

结果是2s，验证了我们的说法，下面我们bye要依赖hellotask执行，
具体代码为
```scala

val helloTask = taskKey[String]("say hello")

val hiTask = taskKey[String]("say hi")

val byeTask = taskKey[String]("say bye")

val runAllTask = taskKey[Unit]("run all")

runAllTask := {
  println(helloTask.value)
  println(hiTask.value)
  println(byeTask.value)

}

helloTask := {
  val hello = "hello"
  println(s"hello task run : $hello")
  Thread sleep 2000
  hello
}


hiTask := {
  val hi = "hi"
  println(s"hi task run : $hi")
  Thread sleep 2000
  hi
}


byeTask := {
  val hi = hiTask.value
  val bye = "bye"
  println(s"hi task value is $hi")
  println(s"bye task run : $bye")
  Thread sleep 2000
  bye
}
```
结果为：
```result

> runAllTask
hi task run : hi
hello task run : hello
hi task value is hi
bye task run : bye
hello
hi
bye
[success] Total time: 4 s, completed Jan 24, 2016 5:49:06 PM

```
由此可看出在没有依赖关系的时候是并行，但有依赖关系就是串行，如果我们再设置hi依赖hello，那么就是6s
__
*注意*：如果互相依赖可能造成死锁，比如a依赖b的value，b里面又依赖a的value，那么就是鸡生蛋和蛋生鸡的问题了，要避免这一点
___
你会说这么简单的东西有个什么用，当然有用了， 比如我们要创建一个版本信息，值来源于git中的headcommit中的sha，那么我们就可以
```scala
val gitHeadCommitSha = taskKey[String]("determine the current git commit SHA")
gitHeadCommitSha := Process("git rev-parse HEAD").lines.head
```
执行`show gitHeadCommitSha`
```result
> show gitHeadCommitSha
[info] 961e628ee67985f75d40833143c7e60e0b70ad03
[success] Total time: 0 s, completed Jan 24, 2016 5:57:16 PM

```

就获取到git中的状态信息了，下面我们将创建一个写版本的task，如下
```scala
val gitHeadCommitSha = taskKey[String]("determine the current git commit SHA")

val makeVersionProperties = taskKey[Seq[File]]("Makes a version.properties file.")

gitHeadCommitSha := Process("git rev-parse HEAD").lines.head

makeVersionProperties := {
  val propFile = (resourceManaged in Compile).value / "version.properties"
  val content = s"version=${gitHeadCommitSha.value}"
  IO.write(propFile, content)
  Seq(propFile)
}

```

我们运行task，将看到
```result

> show makeVersionProperties
[info] List(/home/ctao/WorkSpace/scala/learnsbt/target/scala-2.11/resource_managed/main/version.properties)
[success] Total time: 0 s, completed Jan 24, 2016 6:00:50 PM

```
查看文件`version.properties`
```result
version=961e628ee67985f75d40833143c7e60e0b70ad03
```
剩下的就自己diy了
###多工程
有人说我不止一个module，那么应该怎么搞，就像这样
```scala
name := "learnsbt"

version := "1.0"

scalaVersion := "2.11.7"

lazy val sbtlearn1 = Project("learnsbt1", file("learnsbt1")).
  settings(libraryDependencies += "ch.qos.logback" % "logback-classic" % "1.1.3")
lazy val sbtlearn2 = Project("learnsbt2", file("learnsbt2")).
  settings(fork := true)
lazy val sbtlearn3 = Project("learnsbt3", file("learnsbt3")).
  dependsOn(sbtlearn1)
```
idea中sbt会生成对应的dir，执行 `tree -L 3`
```tree
├── build.sbt
├── learnsbt1
│   ├── project
│   ├── src
│   │   ├── main
│   │   └── test
│   └── target
│       ├── resolution-cache
│       ├── scala-2.10
│       └── streams
├── learnsbt2
│   ├── project
│   ├── src
│   │   ├── main
│   │   └── test
│   └── target
│       ├── resolution-cache
│       ├── scala-2.10
│       └── streams
├── learnsbt3
│   ├── project
│   ├── src
│   │   ├── main
│   │   └── test
│   └── target
│       ├── resolution-cache
│       ├── scala-2.10
│       └── streams

```
我们查看build的jar包，会发现2中没有，而1有logback，3则由于依赖1则有logback和1，因此后面我们只需要定义一个通用，再定义一些依赖就好了：
![buildjar](http://7xnz7j.com1.z0.glb.clouddn.com/depend.png)

### 排除jar包
有的时候我们需要排除jar包，类似maven中的<exclusions>
那么我们可以这样做：
```scala

libraryDependencies ++= Seq(
  "org.apache.hadoop" % "hadoop-common" % "2.6.1" excludeAll ExclusionRule(name = "slf4j-log4j12"),
  "org.apache.hbase" % "hbase-client" % "1.1.2" excludeAll ExclusionRule(name = "slf4j-log4j12"),
  "org.apache.hbase" % "hbase-common" % "1.1.2" excludeAll ExclusionRule(name = "slf4j-log4j12"),
  "ch.qos.logback" % "logback-classic" % "1.1.3",
  "com.typesafe.scala-logging" % "scala-logging_2.11" % "3.1.0"
)

```
由于slf4j有logback和log4j两个实现，我们需要把log4j排除去掉

### 屏蔽源码：
有的时候我们要过滤源码，那么就是
```scala
lazy val ctaokafka = preownedKittenProject("ctaokafka").
  dependsOn(common).
  settings(libraryDependencies ++= Seq(Library.reactiveKafka
  )).settings(CommonSettings.projectSettings)
//  .settings(
//    excludeFilter in(Compile, unmanagedSources) := "*.scala"
//  )
//  .settings(
//    includeFilter := NothingFilter
//  )

```
被注释掉的地方可以将source屏蔽引入，两个是实现的同一个功能

### 最后给出一个我现在喜欢用的配置
首先project定义`CommonSettings`
```scala

import sbt.Keys._
import sbt._
import sbt.plugins.JvmPlugin

object CommonSettings extends AutoPlugin {

  override def requires: JvmPlugin.type = plugins.JvmPlugin


  override def trigger: PluginTrigger = allRequirements


  override def projectSettings: Seq[Def.Setting[Boolean]] = Seq(
    parallelExecution in Test := false,
    //    resolvers ++= Dependencies.resolvers,
    fork in Test := true,
    fork in run := true
  )


}
```
和一个`Dependencies`
```scala
import sbt._
import sbt.Keys._


object Dependencies {

  val core = Seq(
    Library.`scala-async`,
    Library.logback,
    Library.slf4j,
    Library.`play-json`,
    Library.scalatest
  )


  //  val resolvers = DefaultOptions.resolvers(snapshot = true) ++ Seq(
  //    "scalaz-releases" at "http://dl.bintray.com/scalaz/releases" // play-test -> specs2 -> scalaz-stream
  //  )
}


object Version {

  val akkaStream = "2.0.1"

  val reactiveKafka = "0.8.2"

  val scala = "2.11.7"

  val `play-json` = "2.4.6"

  val jline = "0.9.94"

  val logback = "1.1.3"

  val `akka-slf4j` = "2.3.12"

  val slf4j = "1.7.12"

  val `scala-async` = "0.9.5"

  val scalatest = "3.0.0-M14"
}


object Library {
  val akkaStream = "com.typesafe.akka" % "akka-stream-experimental_2.11" % Version.akkaStream

  val reactiveKafka = "com.softwaremill.reactivekafka" %% "reactive-kafka-core" % Version.reactiveKafka

  val `play-json` = "com.typesafe.play" % "play-json_2.11" % Version.`play-json`

  val jline = "jline" % "jline" % Version.jline

  val logback = "ch.qos.logback" % "logback-classic" % Version.logback

  val `akka-slf4j` = "com.typesafe.akka" %% "akka-slf4j" % Version.`akka-slf4j`

  val slf4j = "org.slf4j" % "log4j-over-slf4j" % Version.slf4j

  val `scala-async` = "org.scala-lang.modules" % "scala-async_2.11" % Version.`scala-async`

  val scalatest =  "org.scalatest" % "scalatest_2.11" % Version.scalatest

}
```

然后build.sbt:
```scala
name := "CtaoReactive"

scalaVersion := Version.scala

val gitHeadCommitSha = taskKey[String]("determine the current git commit SHA")

val makeVersionProperties = taskKey[Seq[File]]("Makes a version.properties file.")

inThisBuild(gitHeadCommitSha := Process("git rev-parse HEAD").lines.head)
lazy val common = preownedKittenProject("common").
  settings(
    makeVersionProperties := {
      val propFile = (resourceManaged in Compile).value / "version.properties"
      val content = s"version=${gitHeadCommitSha.value}"
      IO.write(propFile, content)
      Seq(propFile)
    },
    libraryDependencies ++= Seq(Library.akkaStream, Library.`akka-slf4j`)
  ).settings(CommonSettings.projectSettings)

lazy val ctaokafka = preownedKittenProject("ctaokafka").
  dependsOn(common).
  settings(libraryDependencies ++= Seq(Library.reactiveKafka
  )).settings(CommonSettings.projectSettings)
//  .settings(
//    excludeFilter in(Compile, unmanagedSources) := "*.scala"
//  )
//  .settings(
//    includeFilter := NothingFilter
//  )

/**
  * 创建通用模板
  *
  * @param name 工程名
  * @return
  */
def preownedKittenProject(name: String): Project = {
  Project(name, file(name)).
    settings(
      version := "1.0",
      organization := "ctao",
      libraryDependencies ++= Dependencies.core,
      scalaVersion := Version.scala
    )
}


```



