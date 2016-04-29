# 简单介绍

Gradle的出现可以说是JVM世界在构建领域的一个里程碑式的飞跃，Gradle有如下优点：

* 像Ant一样通用灵活
* 像Maven一样Switchable, build-by-convention frameworks a la Maven. But we never lock you in!
* Very powerful support for multi-project builds.
* Very powerful dependency management (based on Apache Ivy).
* Full support for your existing Maven or Ivy repository infrastructure.
* Support for transitive dependency management without the need for remote repositories or pom.xml and ivy.xml files.
* Ant tasks and builds as first class citizens.
* Groovy build scripts.
* A rich domain model for describing your build.


Example 4.1. Executing multiple tasks

build.gradle
task compile << {
    println 'compiling source'
}

task compileTest(dependsOn: compile) << {
    println 'compiling unit tests'
}

task test(dependsOn: [compile, compileTest]) << {
    println 'running unit tests'
}

task dist(dependsOn: [compile, test]) << {
    println 'building the distribution'
}
Output of gradle dist test
> gradle dist test
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs
Each task is executed only once, so gradle test test is exactly the same as gradle test.
