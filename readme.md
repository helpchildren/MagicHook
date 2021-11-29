## 简介

**MagicHook**：**基于ASM和自定义Gradle插件实现的一款编译期插桩的插件，可以进行方法切片式开发，也可以进行一些耗时统计等性能优化相关的统计等，目前支持以下功能：**

1. **可自定义配置需要插桩的类或方法，也可以一键开启所有类和方法插桩。**
2. **可自定义需要忽略插桩的方法****。**
3. **可自定义****方法****插桩****实现类，不配置有默认实现**。

## 效果展示

原始代码：

```java
@HookMethod
public inttest(int num1, int num2)  {
int a = num1 + num2;
return a;
}
```
实际编译插桩后代码：

```java
public int add(int num1, int num2) throws InterruptedException {
MethodHookHandler.enter(this,"com.sesxh.magichooktool.MainActivity","test","[int, int]","int",num1,num2);
int a = num1 + num2;
MethodHookHandler.exit(a,this,"com.sesxh.magichooktool.MainActivity","test","[int, int]","int",num1,num2);
return a;
}
```
默认实现的方法插桩，简单的打印了日志：

```plain
 ╔══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
    ║ [Thread]:main
    ║ [Class]:com.sesxh.magichook.MainActivity
    ║ [Method]:test
    ║ [ArgsType]:[int, int]
    ║ [ArgsValue]:[6,7]
    ║ [Return]:13
    ║ [ReturnType]:int
    ║ [Time]:1 ms
    ╚══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
```

可以看出，这样的话方法名，运行线程，当前对象，入/出参数和耗时情况就都一目了然啦。当然还可以做一些别的事情，例如hook点击事件等等，可自行实现。

## 使用方法

项目根目录：build.gradle 添加以下代码

```groovy
buildscript {
    repositories {
        google()
        jcenter()
        maven{
            url 'http://192.168.126.254:8081/repository/maven-releases/'
        }// 添加maven库地址
    }
    dependencies {
    classpath "com.android.tools.build:gradle:4.1.1"
        // 添加Gradle插件
        classpath 'com.sesxh.android.plugins:magichook:1.0.0'
    }
}
allprojects {
    repositories {
        google()
        jcenter()
        maven{
            url 'http://192.168.126.254:8081/repository/maven-releases/'
        }// 添加maven库地址
    }
}
```
然后项目 module 中启用插件，可以是`application`也可以是`library`

```groovy
apply plugin: 'sesxh.android.plugins'
magichook {
enable = true //是否启用
//true：全部插桩 false：只有注解插桩
all = true
// 下面是非必要配置，无特殊需求可直接删除
//指定插桩那些外部引用的jar，默认空，表示只对项目中的class插桩
jarRegexs = [".*androidx.*"]
//指定插桩那些类文件，默认空
classRegexs = [".*view.*"]
   //指定哪些方法需要插桩，优先级高于注解
methodRegexs = [ ".*init.*"]
//编译时是否打印log
log = true
//是否用插桩后的jar包替换项目中的jar包，慎用
replaceJar = false
//自定义方法统计实现类,不指定默认使用自带实现方式
impl = "com.sesxh.magichook.MyMagicHookHandler"
}
注意：更改此配置需要执行如下命令：Clean Project,然后Make Project才能生效，等到Make Project成功，在build可以看到如下输出：
┌------------------------┐
|      开始魔法Hook之旅      |
└------------------------┘
[Config]:{"all":false,"impl":"com.sesxh.magichook.MyMagicHookHandler","log":true,"enable":true,"replaceJar":false,"jarRegexs":[],"methodRegexs":[],"classRegexs":[]}
[class]MainActivity$DataBean.class is injected.
[class]MainActivity.class is injected.
[class]MyMagicHookHandler.class is injected.
[class]MySimpleHookHandler.class is injected.
┌------------------------┐
|      魔法Hook  √    |
└------------------------┘
```


添加类库依赖：

```groovy
dependencies {
implementation'com.sesxh.android.plugins:magicplugin:1.0.0'
}
```

