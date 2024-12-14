# Gradle Plugin

#### Groovy 两个语法点

- getter / setter
  - 每个 field，Groovy 会自动创建它的 getter 和 setter 方法，从外部可以直接调用，并且在使用 object.fieldA 来获取值或者使用 object.fieldA = newValue 来赋值的时候，实际上会自动转而调用 object.getFieldA() 和 object.setFieldA(newValue)
- 字符串中单双引号
  - 单引号是不带转义的，而双引号内的内容可以使用 "string1${var}string2"的方式来转义

#### Gradle Plugin

##### 什么是 Gradle Plugin？

```groovy
// 官方插件
apply plugin: 'com.android.application'
```

本质：把逻辑独立的代码抽取和封装

###### Plugin 的最基本写法

写在 build.gradle 里：

```groovy
class PluginDemo implements Plugin<Project>{
	@override
	void apply(Project project){
        def author = 'rengwuxian'
        println "Hello ${author}"
	}
}

apply plugin: PluginDemo
```

```
// 运行可以发现 plugin 没有报错，且日志正常输出
gradlew 
```

###### Extension

```groovy
class ExtensionDemo{
    def author = 'Kai'
}

class PluginDemo implements Plugin<Project>{
	@override
	void apply(Project project){
        def extension = new ExtensionDemo()
        println "Hello ${extension.author}."
        
		def extension = project.extensions.create('hencoder',ExtensionDemo)
        // 由于是顺序执行但是配置在后面，需要稍后执行
        project.afterEvaluate{
            println "Hello ${extension.author}."
        }
	}
}
apply plugin: PluginDemo

// 动态配置
hencoder {
    author 'renwuxian'
}
```

###### 正式的 Plugin 项目：自定义 Plugin 代码写在 buildSrc 目录下

- 配置是死套路，具体如下

- main/resources/META-INF/gradle-plugins/*.properties 中的 * 是插件的名称，例如 *.properties 是 com.hencoder.plugindemo.properties，最终应用插件的代码就应该是：

  ```groovy
  apply plugin: 'com.hencoder.plugindemo'
  ```

- *.properties 中只有一行，格式是：

  ```properties
  implementation-class=com.hencoder.plugin_demo.DemoPlugin
  ```

  其中等号右边指定了 Plugin 具体是哪个类，具体的 groovy 文件和 build.gradle 内写法一致

- Plugin 和 Extension 写法和在 build.gradle 里的写法一样

- 关于 buildSrc 目录

  - 这是 gradle 的一个特殊目录，这个目录的 build.gradle 会自动被执行，即使不配置进 setting.gradle
  - buildSrc 的执行早于任何一个 project，也早于 setting.gradle。它是一个独立的存在
  - buildSrc 所配置出来的 Plugin 会被自动添加到编译过程中每一个 project 的 classpath，因此它们才可以直接使用 apply plugin: 'xxx' 的方式来便捷应用这些 plugin
  - setting.gradle 中如果配置了 ':buildSrc'，buildSrc 目录就会被当做是子 Project，因此会被执行两遍。所以在 setting.gradle 里面应该删掉 ':buildSrc' 的配置

- Tips

  可以使用 println "xxx" 和 gradlew 配合测试

#### Transform

- 是什么？：

  是由 Android 提供的，在项目构建过程中把编译后的文件（jar 文件和 class 文件）添加自定义的中间处理过程的工具

- 怎么写？

  - 先新建 build.gradle 添加依赖

    ```groovy
    // 因为 buildSrc 早于任何一个 project 执行，因此需要自己添加仓库
    repositories {
        google()
        jcenter()
    }
    dependencies {
        // 项目代码使用
        implementation 'com.android.tools.build:gradle:3.2.1'
  }
    ```

  - 然后继承 com.android.build.api.transform.Transform，创建一个子类
  
    ```groovy
    public class TransformDemo extends Transform {
        
        // 对应的 task 名称
        @Override
        String getName() {
            return "hencoderTransform"
        }
    
        @Override
        boolean isIncremental() {
            return false
        }
    
        // 具体的转换过程，作用：比如注入方法计时方法
        @Override
        void transform(TransformInvocation transformInvocation) throws TransformException, InterruptedException, IOException {
            def inputs = transformInvocation.inputs
            def outputProvider = transformInvocation.outputProvider
            inputs.each {
                // jarInputs：各个依赖所编译成的 jar 文件
                it.jarInputs.each {
                    File dest = outputProvider.getContentLocation(it.name, it.contentTypes, it.scopes, Format.JAR)
                    FileUtils.copyFile(it.file, dest)
                }
                // directoryInputs：本地 project 编译成的多个 class 文件存放的目录
                it.directoryInputs.each {
                    File dest = outputProvider.getContentLocation(it.name, it.contentTypes, it.scopes, Format.DIRECTORY)
                    FileUtils.copyDirectory(it.file, dest)
                }
            }
        }
    }
    ```
    
  - 还能做什么：修改字节码：上面的这段代码只是把编译完的内容原封不动搬运到目标位置，没有实际用处。要修改字节码，需要引入其他工具，例如：javassist
  
  - javassist

