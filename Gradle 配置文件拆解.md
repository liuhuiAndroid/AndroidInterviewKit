# Gradle 配置文件拆解

#### gradle 是什么?

gradle 利用 Groovy 语言，创造了一种 DSL，但它本身不是语言，它是构建工具

#### 怎么构建?

按照 gradle 的规则构建

- build.gradle

  - buildscript：配置 plugin，里面是稍后执行的代码
    - repositories：配置 plugin 依赖的仓库地址
    - dependencies：配置 plugin 依赖
      - classpath：相当于 add('classpath','xxx')，指定要使用的 plugin
  - allprojects：配置 module
    - repositories：配置 module 依赖的仓库地址

- app.gradle

  - 区分 debug 和 release 版本，debug 不写也是默认有的

    ```java
    // main 同级建立 debug 包，下面放 BuildTypeUtils.java，写 debug 的代码
    // main 同级建立 release 包，下面放 BuildTypeUtils.java，写 release 的代码
    MainActivity 添加 BuildTypeUtils#drawBadge 方法，分别在 debug 包和 release 包下有不同的实现
    ```

- setting.gradle

- gradle-wrapper

- gradle 语法 

#### 闭包

- 相当于可以被传递的代码块

#### buildType 和 produceFlavors

Android 打渠道包

```groovy
// 风味维度
flavorDimensions 'china', 'nation'
// 产品风味
productFlavors {
	free{
		dimension 'china'
	}
	china{
		dimension 'nation'
	}
}
```

#### compile，implementation 和 api

- implementation：不会传递依赖
- compile / api：会传递依赖；api 是 compile 的替代品，效果完全相同
- 当依赖被传递时，二级依赖的改动会导致0级项目重新编译；当依赖不传递时，二级依赖的改动不会导致0级项目重新编译

#### gradle-wrapper

帮助安装 gradle，运行 gradle 项目

```
gradle wrapper
gradlew assemble
```

#### gradle 的项目结构

- 单 project

- 多 project：由 setting.gradle 配置多个

  查找 settings 的顺序：

  - 当前目录
  - 兄弟目录 master
  - 父目录

#### task

- 使用方法

  ```
  gradlew taskName
  ```

- task 的结构

  ```
  task taskName{
  	初始化代码
  	doFirst{
  		task 代码
  	}
  	doLast{
  		task 代码
  	}
  }
  ```

- doFirst doLast 和普通代码段的区别

  - 普通代码段：在 task 创建过程中就会被执行，发生在 configuration 阶段
  - doFirst 和 doLast：在 task 执行过程中被执行，发生在 execution 阶段。如果用户没有直接或间接执行 task，那么它的 doFirst doLast 代码不会被执行
  - doFirst doLast 都是 task 代码，其中 doFirst 是往任务队列的前面插入代码，doLast 是往任务队列的后面插入代码

- task 的依赖

  可以使用 task taskA(dependsOn: taskB) 的形式来指定依赖。指定依赖后，taskA 会在自己执行前先执行自己依赖的 taskB
  
- task 实际例子：增加配置的版本名和版本号

#### gradle 执行的生命周期

- 三个阶段
  - 初始化阶段：执行 setting.gradle，确定主 project 和子 project
  - 定义阶段：执行每个 project 的 build.gradle，确定出所有 task 所组成的有向无环图
  - 执行阶段：按照上一阶段所确定出的有向无环图来执行指定的 task
- 在阶段之间插入代码
  - 一二阶段之间
    
    - setting.gradle 的最后
    
  - 二三阶段之间

    ```groovy
    afterEvaluate{
    	插入代码
    }
    ```

