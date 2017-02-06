---
layout: post
title:  "自定义注解"
crawlertitle: "自定义注解"
summary: "自定义注解"
date:   2015-12-01 23:09:47 +0700
categories: posts
tags: '编程基础'
author: hewking
---

> Annotion 自jdk 5.0 引入，在java中方方面面均有涉及，本文在使用上做相应记录

### what is 注解
   Annotion 是java提供的一种元程序中的元素关联任何信息和任何元数据的途径和方法。Annotion 是接口，程序可以通过反射来获取指定程序元素的注解对象，然后通过注解对象来获取注解里面的元数据。
    Annotion 是jdk 5.0 之后引入的，可以用于创建文档，跟踪代码的依赖性，甚至执行编译检查，从某些方面看，annotion就像修饰符一样被使用，应用于包，类型，构造方法，方法，成员变量，参数，本地变量的申明中。这些信息被存储在Annotion 的 “name=value”结构对中。

### 自定义注解
使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口，由编译程序自动完成其他细节。在定义注解时，不能继承其他的注解或接口。@interface用来声明一个注解，其中的每一个方法实际上是声明了一个配置参数。方法的名称就是参数的名称，返回值类型就是参数的类型（返回值类型只能是基本类型、Class、String、enum）。可以通default来声明参数的默认值。
        自定义注解格式：
 public @interface 注解名{注解体}
                
```

所有基本数据类型
（int,float,boolean,byte,double,char,long,short)
                2.String类型
                3.Class类型
                4.enum类型
                5.Annotation类型
                6.以上所有类型的数组 Annotation类型里面的参数该怎么设定:
                第一,只能用public或默认(default)这两个访问权修饰.例如,String value();这里把方法设为defaul默认类型；　 　
                第二,参数成员只能用基本类型byte,short,char,int,long,float,double,boolean八种基本数据类型和 String,Enum,Class,annotations等数据类型,以及这一些类型的数组.例如,String value();这里的参数成员就为String;　　
                第三,如果只有一个参数成员,最好把参数名称设为"value",后加小括号.
```
Gender 性别注解：

```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Gender {
	String value() default "";
	
	public enum GenderType{
		Male("男"),Female("女"),Other("中性");
		
		private String gendarStr;
		
		private GenderType(String str){
			this.gendarStr = str;
		}
	}
	
	GenderType gender() default GenderType.Male;
}

```

### 注解元素的默认值
    注解元素必须有确定的值，在定义注解的默认值中指定，或者在使用注解时指定，非基恩类型的注解元素的值不可为null。因此，使用空字符串或0作为默认值，是一种常用的做法。这个约束使处理器难以表现一个元素存在货缺失的状态，在每个注解的声明中，所有元素都存在，并且都具有相应的值，为了绕开这个约束，只能定义一些特殊的值，比如空串或者负数，表示某个元素不存在。

### 注解处理类（java.ang.reflect.AnnotateElement）
      注解元素java使用接口代表程序元素前面的注解，该接口是所有Annotion类型的父接口。除此之外，Java在java.lang.reflect 包下新增了AnnotatedElement接口，该接口代表程序中可以接受注解的程序元素，该接口主要有如下几个实现类：
        Class：类定义
        Constructor：构造器定义
        Field：累的成员变量定义
        Method：类的方法定义
        Package：类的包定义
当一个Annotation被定义为运行时Annotation后，改注解才是运行时可见的，当class文件被装载时被保存在class文件中的Annotation才会被虚拟姐读取。 AnnotatedElement   
接口提供了以下四个方法来访问Annotation的信息：
        方法1：<T extends Annotation> T getAnnotation(Class<T> annotationClass): 返回改程序元素上存在的、指定类型的注解，如果该类型注解不存在，则返回null。
        方法2：Annotation[] getAnnotations():返回该程序元素上存在的所有注解。
        方法3：boolean is AnnotationPresent(Class<?extends Annotation> annotationClass):判断该程序元素上是否包含指定类型的注解，存在则返回true，否则返回false.
        方法4：Annotation[] getDeclaredAnnotations()：返回直接存在于此元素上的所有注释。与此接口中的其他方法不同，该方法将忽略继承的注释。（如果没有注释直接存在于此元素上，则返回长度为零的一个数组。）该方法的调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响。
我们为前面定义好的自定义注解写一个简单的处理器

```

public class AnnotionProcessor {
	public static void getInfo(Class<?> clazz) {
		Field fields[] = clazz.getDeclaredFields();
		for(Field field : fields) {
			if(field.isAnnotationPresent(Gender.class)) {
				Gender gender = field.getAnnotation(Gender.class);
				System.out.println(gender.gender().toString() + ">>>");
			}
		}
	}
}
```

简单使用上述示例：

```

public class Person {
	@Gender(gender = GenderType.Male)
	public String gender;
}

public class CustomAnnotionTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		AnnotionProcessor.getInfo(Person.class);
		System.out.println(">>>");
	}

}
```

打印结果：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1394860-65ce53688b16c927.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


