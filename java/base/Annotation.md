

# Annotation概述

## 一个Annotation简单例子

```java
@Entity
public class Book {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
}
```

上面就是mybatis用的几个注解@Entity，@Id。。。



## Annotation 定义

Java 注解（Annotation）又称 Java 标注，你可以理解为提供了为程序元素设置元数据的方法可以来定义了一些元数据（用来描述数据的数据）。

你也可以理解为就是增加了一种功能，给程序添加一些额外信息（Book是一个Entity类，id为PK），使程序便于理解

更容易简化处理（可以通过Entity类，id为PK，来自动生成table，也可以自动生成增删改查语句）。

Annotation不影响程序代码的执行，无论增加、删除Annotation，代码都始终如一的执行。

Annotation的替代品就是property文件，property文件是单独定义在一个文件中，Annotation是直接定义在程序中的。

```properties
<Entity class="Book">
  <property="id" GenerationType="auto"/>
</Entity>
```



## Annotation 能干嘛

1. 在编译阶段，可以检查代码，生成文档，也可以生成source
2. 在运行阶段，可以做校验（Hyper Validator），可以动态执行一些功能（例如，mybatis根据Annotation ，接口，可以动态执行增删改查功能）
3. 可以用来修饰类、接口、字段、方法、参数、构造方法等，往往注解还搭配编译器、程序运行时以及其他工具或者插件使用，用于实现代码功能的增强或者修改程序的行为等操作

## Annotation 原理

利用java反射和动态代理。

# java内置的注解

## 基本注解

@Override，@Deprecated，@SuppressWarnings

## 元注解

@Retention   

- SOURCE（Annotation信息存储在source中只能编译期间用）
- CLASS（Annotation信息存储在class中）
- RUNTIME（Annotation信息存储在class中，并且JVM可以读取Annotation信息用于运行时）

@Target

- @Target(ElementType.ANNOTATION_TYPE)：指定该该策略的Annotation只能修饰Annotation.
- @Target(ElementType.TYPE)   //接口、类、枚举、注解
- @Target(ElementType.FIELD) //成员变量（字段、枚举的常量）
- @Target(ElementType.METHOD) //方法
- @Target(ElementType.PARAMETER) //方法参数
- @Target(ElementType.CONSTRUCTOR)  //构造函数
- @Target(ElementType.LOCAL_VARIABLE)//局部变量
- @Target(ElementType.PACKAGE) ///修饰包定义
- @Target(ElementType.TYPE_PARAMETER) //java8新增，后面Type Annotation有介绍
- @Target(ElementType.TYPE_USE) ///java8新增，后面Type Annotation有介绍
- @Inherited  它所标注的Annotation具有继承性，子类可以继承父类的注解
- @Documented java doc 用

## 新增注解

@SafeVarargs（去除“堆污染”警告），@FunctionalInterface（修饰函数式接口），@Repeatable（）

# Annotation 使用

## Annotation定义

## 运行时获取Annotation

通过java反射获取

```java
Class<?> c = MyMethodAnnotationDemo.class;
Method m = c.getDeclaredMethod("run");

MyMethodAnnotation[] as = m.getDeclaredAnnotationsByType(MyMethodAnnotation.class);

for(MyMethodAnnotation a : as) {
    System.out.println(a);

}
```



## 编译时获取Annotation

使用 JSR269 api 继承javax.annotation.processing.AbstractProcessor重载以下三个方法

```java
@Override
public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv)
@Override  
public Set<String> getSupportedAnnotationTypes()
@Override  
public SourceVersion getSupportedSourceVersion()
```

### 创建MyProcessor

```java
package lhn.annotation.processor;

import java.io.FileOutputStream;
import java.io.PrintStream;
import java.util.LinkedHashSet;
import java.util.Set;

import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.RoundEnvironment;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.Element;
import javax.lang.model.element.ElementKind;
import javax.lang.model.element.Name;
import javax.lang.model.element.TypeElement;

import lhn.annotation.demo.entity.*;

public class MyProcessor extends AbstractProcessor {

	@Override
	public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
		//定义一个文件输出流，用于生成额外的文件
        PrintStream ps = null;
        try{
            //遍历每个被@Entity修饰的class文件,使用RoundEnvironment来获取Annotation信息
            for( Element t : roundEnv.getElementsAnnotatedWith(Entity.class)){
                //获取正在处理的类名
                Name clazzName = t.getSimpleName();
                //获取类定义前的@Entity Annotation
                Entity table = t.getAnnotation(Entity.class);
                //创建文件输出流
                String fileName =clazzName+".txt";
                ps = new PrintStream(new FileOutputStream(fileName));
                 // 执行输出
                 ps.print("class name=" + clazzName);
                 // 输出per的table()的值
                 ps.println(" table name=" + table.tableName() );
                 //获取@Entity修改类的各个属性字段。t.getEnclosedElements()获取该Elemet里定义的所有程序单元
                 for(Element ele : t.getEnclosedElements()){
                     
                     //只处理成员变量上的Annotation，ele.getKind()返回所代表的的程序单元
                     if( ele.getKind() == ElementKind.FIELD){
                        //被id注解修饰的字段
                    	 Column column= ele.getAnnotation(Column.class);
                         if( column != null){
                             String cname =column.cname();
                             String ctype =column.ctype();
                             boolean isPk = column.pk();
                             // 执行输出
                               ps.print(" column name=" + cname);
                               ps.print(" column type=" + ctype);
                               ps.println(" column is pk =" + isPk);
                         }
                     }
                 }// end for
                 ps.println("");
            }// end for 
            
        }catch(Exception e){
            e.printStackTrace();
        }finally {
            if(ps!=null){
                try{
                    ps.close();
                }catch(Exception e){
                    e.printStackTrace();
                }
            }
        }
        return true;
	}
	
	 /** 
     * 这里必须指定，这个注解处理器是注册给哪个注解的。注意，它的返回值是一个字符串的集合，包含本处理器想要处理的注解类型的合法全称 
     * @return  注解器所支持的注解类型集合，如果没有这样的类型，则返回一个空集合 
     */  
    @Override  
    public Set<String> getSupportedAnnotationTypes() {  
        Set<String> annotataions = new LinkedHashSet<String>();  
        annotataions.add(Entity.class.getCanonicalName());  
        annotataions.add(Column.class.getCanonicalName());  
        return annotataions;  
    }  
  
    /** 
     * 指定使用的Java版本，通常这里返回SourceVersion.latestSupported()，默认返回SourceVersion.RELEASE_6 
     * @return  使用的Java版本 
     */
    @Override  
    public SourceVersion getSupportedSourceVersion() {  
        return SourceVersion.latestSupported();  
    }  
	
}
```

### 通过javac在编译时执行MyProcessor获取Annotation信息生成User.txt

target\classes为lhn.annotation.processor.MyProcessor的类路径

```shell
javac -encoding UTF-8 -d target/classes -classpath target\classes -processor lhn.annotation.processor.MyProcessor src/main/java/lhn/annotation/demo/entity/User.java
```

### 通过maven编译时执行MyProcessor获取Annotation信息生成User.txt

### 通过设置eclipse编译时执行MyProcessor获取Annotation信息生成User.txt

# 参考文档

https://www.jianshu.com/p/28edf5352b63

