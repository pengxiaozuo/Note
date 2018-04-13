# Android 中的ProGuard

[TOC]

## 官方例子

[Proguard官网-例子和一些外部库已经默认的配置](https://www.guardsquare.com/en/proguard/manual/examples)

[Proguard手册](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html)

## 什么是Proguard

- Proguard是一个集文件压缩、优化、混淆和校验等功能的工具
- 它检测并删除无用的类、变量、方法和属性
- 它优化字节码并删除无用指令
- 它通过将类名、变量名和方法名重命名为无意义的名称实现混淆效果

## 开启混淆

在app的build.gradle文件中配置minifyEnable为true即可

```gradle {.line-numbers}

buildTypes{
    release {
        minifyEnable true
        proguardFiles getDefaultProguardFiles('proguard-android.text'), 'proguard-rules.pro'
    }
}

```

## 混淆常见配置

### -keep

keep 用来保留Java元素不进行混淆

- keep
- keepclassmembers
- keepclasseswithmembers

下面先看下现象，后面有匹配规则

#### keep:可以保留指定的类名以及成员

*Person类:*

```java {.line-numbers}
package com.peng.proguarddemo;
public class Person {

    public Person(String name, int age, String address) {
        this.name = name;
        this.age = age;
        this.address = address;
    }

    public Person() {
    }

    private String name;
    private int age;
    private String address;

    //public getter setter ...

    @Override
    public String toString() {/*...*/ }
}
```

在其他地方使用了Person类

```java
Person person = new Person();
````

不配置任何规则直接混淆

- 类名被重命名
- 未使用的方法被删除（包括有参数的构造器）
- 成员和参数被重命名

*Person类被混淆之后:*

```java {.line-numbers}
package com.peng.proguarddemo;
public class a
{
    private String a;
    private int b;
    private String c;

    public String toString()
    {
        StringBuilder localStringBuilder = new StringBuilder();
        localStringBuilder.append("Person{name='");
        localStringBuilder.append(this.a);
        localStringBuilder.append('\'');
        localStringBuilder.append(", age=");
        localStringBuilder.append(this.b);
        localStringBuilder.append(", address='");
        localStringBuilder.append(this.c);
        localStringBuilder.append('\'');
        localStringBuilder.append('}');
        return localStringBuilder.toString();
    }
}
```

**保持类名不被混淆:**

```proguard
-keep public class com.peng.proguarddemo.Person
```

*添加规则被混淆之后:*

```java {.line-numbers}
package com.peng.proguarddemo;
public class Person
{
    private String a;
    private int b;
    private String c;

    public String toString()
    {
        StringBuilder localStringBuilder = new StringBuilder();
        localStringBuilder.append("Person{name='");
        localStringBuilder.append(this.a);
        localStringBuilder.append('\'');
        localStringBuilder.append(", age=");
        localStringBuilder.append(this.b);
        localStringBuilder.append(", address='");
        localStringBuilder.append(this.c);
        localStringBuilder.append('\'');
        localStringBuilder.append('}');
        return localStringBuilder.toString();
    }
}

```

**某个类的子类类名不被混淆:**

```proguard
-keep public class * extends com.peng.proguarddemo.Person

#-keep public class * implements xx.xx.IxxInterface 不混淆某个类的实现类
```

*Worker类:*

```java {.line-numbers}
package com.peng.proguarddemo;
public class Worker extends Person {
    private int salary;

    public int getSalary() {
        return salary;
    }

    public void setSalary(int salary) {
        this.salary = salary;
    }

    @Override
    public String toString() {
        return "Worker{" +
                "salary=" + salary +
                "} " + super.toString();
    }
}
```

*添加规则被混淆之后:*
配置了Person类的子类不被混淆类名之后Worker的类名没有被重命名

```java {.line-numbers}
package com.peng.proguarddemo;
public class Worker  extends Person
{
    private int a;

    public String toString()
    {
        StringBuilder localStringBuilder = new StringBuilder();
        localStringBuilder.append("Worker{salary=");
        localStringBuilder.append(this.a);
        localStringBuilder.append("} ");
        localStringBuilder.append(super.toString());
        return localStringBuilder.toString();
    }
}
```

**不混淆某个类:**

```proguard
-keep public class com.peng.proguarddemo.Person {*;}
```

*添加规则被混淆之后:*
可以看到配置了不混淆某个类之后类名、构造器、成员都没有被删除和重命名

```java {.line-numbers}
public class Person
{
    private String address;
    private int age;
    private String name;

    public Person() {}

    public Person(String paramString1, int paramInt, String paramString2)
    {
        this.name = paramString1;
        this.age = paramInt;
        this.address = paramString2;
    }

    public String getName()
    {
        return this.name;
    }

    //getter...

    public void setName(String paramString)
    {
        this.name = paramString;
    }

    //setter...

    public String toString()
    {
        StringBuilder localStringBuilder = new StringBuilder();
        localStringBuilder.append("Person{name='");
        localStringBuilder.append(this.name);
        localStringBuilder.append('\'');
        localStringBuilder.append(", age=");
        localStringBuilder.append(this.age);
        localStringBuilder.append(", address='");
        localStringBuilder.append(this.address);
        localStringBuilder.append('\'');
        localStringBuilder.append('}');
        return localStringBuilder.toString();
    }
}
```

**保留某个包下面的类的类名不被混淆:**

- 包下的类类名都保持原样
- 不包括包下的子包

```proguard
-keep public class com.peng.proguarddemo.*
```

**保留某个包下面的类不被混淆:**

- 类名和成员都保持原样
- 不包括包下的子包

```proguard
-keep public class com.peng.proguarddemo.* {*;}
```

**保留某个包下面的类以及子包下的类的类名不被混淆:**

- 包下的类名不被混淆
- 包下的子包的包名不被混淆
- 子包下的类名不被混淆
- 子包下的子包名保持原样
- 子包下的子包的类名保持原样
- 子子孙孙无穷尽

```proguard
-keep public class com.peng.proguarddemo.**
```

**不混淆某个包下的所有类和接口:**

*这个配置，就是这个包下的东西，你在IDE里面看到啥样，混淆之后还是啥样，没动它*

- 包下的类名和成员不被混淆
- 包下的子包的包名不被混淆
- 子包下的类和成员不被混淆
- 子包下的子包名保持原样
- 子包下的子包的类和成员保持原样
- 子子孙孙无穷尽

```proguard
-keep public class com.peng.proguarddemo.**{*;}
```

**不混淆包下的public成员:**

- 包下的类名和public成员不被混淆
- 包下的子包的包名不被混淆
- 子包下的类和public成员不被混淆
- 子包下的子包名保持原样
- 子包下的子包的类和public成员保持原样
- 子子孙孙无穷尽

```proguard
-keep public class com.peng.proguarddemo.entity.**{public *;}
```

**类中的成员列表（大括号里面的东西）:**

- \<init\> 构造器
- \<fields\> 匹配任意字段
- \<methods\> 匹配任意方法
- \* 匹配任意成员

    > Note that the above wildcards don't have return types. Only the \<init> wildcard has an argument list.

```proguard
-keep public class com.peng.proguarddemo.**{
    <init>(...);#构造器
    <fields>;#字段
    <methods>;#方法
    *;#所有成员
}
```

**成员名称可使用的正则表达式:**

- `?`代表方法中的任意单个字符.
- `*`代表方法中的任意多个字符.

**成员类型可使用的通配符:**

- `%`表示任意基本类型(int,char等,但是不包括void).
- `?`表示类名中的任意单个字符.
- `*`表示类名中的任意多个字符,不包括分隔符(.).
- `**`表示类名中的任意多个字符,包括分隔符(.).
- `***`表示任意类型.
- `...`表示任意多个任意类型的参数.

**类名匹配可使用的通配符:**

- `?`表示类名中的任意单个字符,但是不包括分割符(.)
     > For example, "mypackage.Test?" matches "mypackage.Test1" and "mypackage.Test2", but not "mypackage.Test12".
- `*`表示类名中的任意多个字符,但是不包括分隔符(.)
    > For example, "mypackage.*Test*" matches "mypackage.Test" and "mypackage.YourTestApplication", but not "mypackage.mysubpackage.MyTest". Or, more generally, "mypackage.*" matches all classes in "mypackage", but not in its subpackages.
- `**`表示类名中的任意多个字符,包括分隔符(.)
    > For example, "\**.Test" matches all Test classes in all packages except the root package. Or, "mypackage.\**" matches all classes in "mypackage" and in its subpackages.

#### keepclassmembers:只能保留住成员而不能保留住类名

**保留所有符合规则的类的字段:**

- 使用较少，，匹配规则和keep相同

```proguard
-keepclassmembers  public class com.peng.proguarddemo.** {
    <fields>;
}
```

#### keepclasseswithmembers:可以根据成员找到满足条件的所有类而不用指定类名,可以保留类名和成员名

**保留所有符合规则的类的类名和本地方法:**

- 使用较少，匹配规则和keep相同

下面的这个是使用最多的用法,包下包括子包..下的所有的类都会保持类名和本地方法不被混淆

```proguard
-keepclasseswithmembers  public class com.peng.proguarddemo.** {
    native <methods>;
}
```

## 哪些不应该混淆

### 已经有的配置

**Android已经提供了很多的默认配置:**

```gradle {.line-numbers}
buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
}
```

> **proguard-android.txt:**

```proguard {.line-numbers}
# This is a configuration file for ProGuard.
# http://proguard.sourceforge.net/index.html#manual/usage.html
#
# This file is no longer maintained and is not used by new (2.2+) versions of the
# Android plugin for Gradle. Instead, the Android plugin for Gradle generates the
# default rules at build time and stores them in the build directory.

#包明不混合大小写
-dontusemixedcaseclassnames
#不跳过非公共的库的类
-dontskipnonpubliclibraryclasses
#混淆时记录日志
-verbose

# Optimization is turned off by default. Dex does not like code run
# through the ProGuard optimize and preverify steps (and performs some
# of these optimizations on its own).
#不优化输入的类文件
-dontoptimize
#关闭预校验
-dontpreverify
# Note that if you want to enable optimization, you cannot just
# include optimization flags in your own project configuration file;
# instead you will need to point to the
# "proguard-android-optimize.txt" file instead of this one from your
# project.properties file.
#保护注解
-keepattributes *Annotation*
-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService

# For native methods, see http://proguard.sourceforge.net/manual/examples.html#native
#保持所有拥有本地方法的类名及本地方法名
-keepclasseswithmembernames class * {
    native <methods>;
}

# keep setters in Views so that animations can still work.
# see http://proguard.sourceforge.net/manual/examples.html#beans
#不混淆View的子类的get和set方法
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
}

# We want to keep methods in Activity that could be used in the XML attribute onClick
#Activity的子类中的onClick方法不被混淆
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}

# For enumeration classes, see http://proguard.sourceforge.net/manual/examples.html#enumerations
#保持枚举类
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

#保持序列号的类的Creator
-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}
#保持R文件静态字段不被混淆
-keepclassmembers class **.R$* {
    public static <fields>;
}

# The support library contains references to newer platform versions.
# Don't warn about those in case this app is linking against an older
# platform version.  We know about them, and they are safe.

-dontwarn android.support.**

#keep相关注解
# Understand the @Keep support annotation.
-keep class android.support.annotation.Keep

-keep @android.support.annotation.Keep class * {*;}

-keepclasseswithmembers class * {
    @android.support.annotation.Keep <methods>;
}

-keepclasseswithmembers class * {
    @android.support.annotation.Keep <fields>;
}

-keepclasseswithmembers class * {
    @android.support.annotation.Keep <init>(...);
}

```

### 其他自定义配置

在proguard-rules.pro中配置