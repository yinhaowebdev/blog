title: '学习Spring Cloud(二): IOC'
author: Billy Yin
tags: []
categories: []
date: 2020-11-29 21:20:00
---
当实现某种功能需要几个对象相互作用，也就是需要一个主对象包含其他对象的引用，有两种方式： 
* 主对象内部主动获取所需引用（如： new一个对象）
* 控制反转(Ioc)，通过setter或者构造方法向主对象传入所需引用

> 控制反转是一种思想，而不是技术

Ioc分为两类：
* 依赖注入
* 依赖查找

### 传统编程
```
public Class Sir{
  public void command{
    if(...){
      Student student = new Student();
      student.dothing();
    }else{
      Student2 student = new Student2();
      student.dothing();
    }
  }
} 
```

### Ioc
把控制权交给decide
```
public interface People {
    void dothing();
}
```
``` 
public Class Sir{
  public void Command{
    Decider.decide().dothing();
  }
} 
```
```
public Class Decider{
  public People decide{
    if(...){
      People student = new Student();
    }else{
      People student = new Student2();
    }
    return student;
  }
} 
```
把控制权交给测试类
```
public interface People {
    void dothing();
}
```
``` 
public Class Sir{
  private People people;
   
  public Sir(People people) {
    this.people = people;
  }
  public void Command{
    people.dothing();
  }
} 
```
```
public class Test {

    public static void main(String[] args) {
        if(...){
          Sir sir = new Sir(new Student());
        }
        laowang.mingling();
    }

}
```
### Spring 编程
把控制权交给spring容器
pom.xml
```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>4.3.2.RELEASE</version>
</dependency>

```
application.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="sir" class="com.cmower.java_demo.ioc.Sir">
    <constructor-arg ref="people" />
  </bean>
        
  <bean id="people" class="com.cmower.java_demo.ioc.Student" />

</beans>
```
