---
title: Java注解与注解处理器
date: 2018-04-18 15:43:06
tags:
- Java
- 注解
- 注解处理器
layout:
updated:
comments: true
categories:
- Java
permalink:
---
{% asset_img Java注解与注解处理器.jpg %}

> 版权声明：本文为 *冬夏* 原创文章，可以随意转载，但请注明出处。

<!-- more -->

# 元注解 #
## @Retention ##
@Retention：保留期，表示注解的存活时间，取值为：
  * RetentionPolicy.SOURCE：注解只在源码阶段保留，编译后忽略，主要起到提示、警告的作用，如@Override
  * RetentionPolicy.CLASS：注解保留到编译后的class文件中，运行时忽略，即不会加载到虚拟机中，但是可以通过读文件的形式获取class文件中的注解，该类注解可以在编译时进行规则检查或自动生成某些代码，如@NonNull
  * RetentionPolicy.RUNTIME：注解保留到运行时，可以在运行时获取，可以用作动态条件判断等功能，如@Inherited、@Documented

## @Documented ##
@Documented:文档，含有该注解的注解能够被javadoc生成java文档

## @Target ##
@Target：表示该注解的作用范围，取值为：
  * ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
  * ElementType.CONSTRUCTOR 可以给构造方法进行注解
  * ElementType.FIELD 可以给属性进行注解
  * ElementType.LOCAL_VARIABLE 可以给局部变量进行注解
  * ElementType.METHOD 可以给方法进行注解
  * ElementType.PACKAGE 可以给一个包进行注解
  * ElementType.PARAMETER 可以给一个方法内的参数进行注解
  * ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举

## @Inherited ##
@Inherited：表示该注解是否可以被继承，当父类使用了可继承的注解，子类默认也含有该注解

## @Native ##
@Native：表示变量来自本地代码

## @Repeatable ##
@Repeatable：Java 1.8新增，代表注解的类可以有多个取值

# 自定义注解 #
## 语法 ##
Java中，可以使用@interface关键字创建自定义注解：

    @Documented
    @Target(ElementType.METHOD)
    @Inherited
    @Retention(RetentionPolicy.RUNTIME)
    public @interface MethodInfo {
        String author() default "";
        String date();
        int version() default 1;
        String comments();
    }

### 注意事项 ###

* 使用@interface自定义注解时，自动继承了 java.lang.annotation.Annotation 接口
* 注解中的方法，实际是声明了一个配置参数，参数名称就是方法名，返回值类型智能是基本类型、Class、String、enum、Annotation以及前面类型的数组。可以通过default来声明参数默认值。
* 注解中的方法，不允许使用project、private修饰符，也无需加上public修饰符，这一点和接口类似。
* 自定义注解需要在前面加上元注解，用以表示注解的使用方式及范围。

## 使用例子 ##

    @MethodInfo(comments = "test method", author = "dongxiawu", date = "2018-04-18", version = 1)
    public String testMethod() {
        return "test method";
    }
### 注意事项 ###
* 接口中有默认值的属性，在使用时可以忽略，而没有默认值的属性则不能忽略，否则会报错。

# 注解解析 #

## 基本原理 ##
在代码中解析注解需要使用Java的反射机制，但只能解析 Retention 为 RetentionPolicy.RUNTIME 的注解。

## 例子 ##
    public class Main {
        public static void main(String[] args){
            for (Method method : Main.class.getMethods()){
                if (method.isAnnotationPresent(MethodInfo.class)){
                    MethodInfo methodInfo = method.getAnnotation(MethodInfo.class);

                    System.out.println("comments: " + methodInfo.comments() + " author: " + methodInfo.author()
                            + " date:" + methodInfo.date() + " version: " + methodInfo.version());
                }
            }
        }

        @MethodInfo(comments = "test method", author = "dongxiawu", date = "2018-04-18", version = 1)
        public String testMethod() {
            return "test method";
        }
    }

# 注解处理器 #

在运行时进行注释解析只能解析只能解析 Retention 为 RetentionPolicy.RUNTIME 的注解，对于  Retention 为 RetentionPolicy.SOURCE 和 RetentionPolicy.CLASS 的注解由于分别在编译完成后和加载到JVM时消除了，所以不能在运行时获取到，这时就需要注解处理器，在编译器是能够获取注解并进行相应操作。

## 概念 ##

注解处理器(Annotation Processor)是javac内置的一个用于编译时扫描和处理注解(Annotation)的工具。简单的说，在源代码编译阶段，通过注解处理器，我们可以获取源文件内注解(Annotation)相关内容。

## 用途 ##

由于注解处理器可以在程序编译阶段工作，所以我们可以在编译期间通过注解处理器进行我们需要的操作。比较常用的用法就是在编译期间获取相关注解数据，然后动态生成.java源文件（让机器帮我们写代码），通常是自动产生一些有规律性的重复代码，解决了手工编写重复代码的问题，大大提升编码效率。

另外一点好处就是由于是在编译器对代码进行处理，所以对代码的运行效率没有影响。（如果是在运行时对注解进行处理，如Retention 为 RetentionPolicy.RUNTIME 的注解，则会占用运行时间）

## 使用方法 ##

1. 定义自己的注解处理器，首先要继承 Processor 接口，不过为了不实现接口的全部方法，可以选择继承 AbstractProcessor 抽象类，并实现 process 方法。

2. 在编译期间使用自己定义的注解处理器，主要有两种方法：
  1. 使用 `-processor` 选项指定注解处理器的位置，如 `javac -processor Myprocessor Main.java`
  2. 将注解处理器打包成jar包，并采用`-processorpath`等选项指定路径，如`javac -processorpath processor.jar Main.java`，同时还需要在 jar 包中添加自定义的 processor 类名信息，以便编译过程中能够找到正确的 Processor，具体方法为：在 META-INF/services/ 文件夹下新建 javax.annotation.processing.Processor 文件，并在其中添加对应的 processor 类名信息。

注：也可以使用 gradle 自动构建，如google 的 `com.google.auto.service:auto-service` 库

## 使用例子 ##

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.SOURCE)
    @Documented
    public @interface Persistent {
        String table();
    }

    @Target(ElementType.FIELD)
    @Retention(RetentionPolicy.SOURCE)
    @Documented
    public @interface Id {
        String column();
        String type();
        String generator();
    }

    @Documented
    @Target(ElementType.FIELD)
    @Retention(RetentionPolicy.SOURCE)
    public @interface Property {
        String column();
        String type();
    }

    @Persistent(table = "persion_inf")
    public class Persion {
        @Id(column = "person_id", type = "integer", generator = "identity")
        private int id;
        @Property(column = "person_name", type = "string")
        private String name;
        @Property(column = "person_age", type = "integer")
        private int age;

        public Persion(){

        }

        public Persion(int id, String name, int age){
            this.id = id;
            this.name = name;
            this.age = age;
        }

    }

    @SupportedSourceVersion(SourceVersion.RELEASE_8)
    @SupportedAnnotationTypes({"Persistent","Property","Id"})
    public class MyProcessor extends AbstractProcessor {

        Elements elementUtils;

        @Override
        public synchronized void init(ProcessingEnvironment processingEnv) {
            super.init(processingEnv);
            elementUtils = processingEnv.getElementUtils();
        }

        @Override
        public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {

            PrintStream ps = null;
            try {
                for (Element t : roundEnv.getElementsAnnotatedWith(Persistent.class)){
                    Name clazzName = t.getSimpleName();
                    Persistent per = t.getAnnotation(Persistent.class);
                    ps = new PrintStream(new FileOutputStream(clazzName + ".txt"));
                    ps.println("table: " + per.table());

                    for (Element f : t.getEnclosedElements()){
                        if (f.getKind() == ElementKind.FIELD){
                            Id id = f.getAnnotation(Id.class);
                            if (id != null){
                                ps.println("id: " +" name: " + f.getSimpleName() +" column: " + id.column()
                                        + " type: " + id.type() + " generator: " + id.generator());
                            }

                            Property property = f.getAnnotation(Property.class);
                            if (property != null){
                                ps.println("property: " +" name: " + f.getSimpleName() +" column: " + property.column()
                                        + " type: " + property.type());
                            }
                        }
                    }
                }
            }catch (Exception e){
                e.printStackTrace();
            }finally {
                if (ps != null){
                    try {
                        ps.close();
                    }catch (Exception e){
                        e.printStackTrace();
                    }

                }
            }

            return true;
        }
    }

编译命令 `javac -processor MyProcessor Persion.java`

结果输出一个文件，内容为：

    table: persion_inf
    id:  name: id column: person_id type: integer generator: identity
    property:  name: name column: person_name type: string
    property:  name: age column: person_age type: integer

## 注意事项 ##
自定义注解处理器前可以加上三个注解

* @SupportedSourceVersion()：支持版本
* @SupportedAnnotationTypes()：支持的注解
* @SupportedOptions()：支持的选项
