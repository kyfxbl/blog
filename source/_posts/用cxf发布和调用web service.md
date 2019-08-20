title: 用cxf发布和调用web service
date: 2013-09-24 11:00
categories: java
---
最近我们的系统需要和一个第三方系统对接，对接的方式是通过web service，所以就学习了一下这方面的东西。用CXF来做web service是比较简单的，本文就简单介绍一下如何一步步发布web service，以及调用现有的web service。
<!--more-->

# 发布web service

第1步是创建一个普通的java接口，然后用@WebService注解声明这是一个web service

```
package com.huawei.framework.webservice;

import javax.jws.WebService;

import com.huawei.framework.model.User;

@WebService
public interface HelloWorld {

	String sayHi(User user);

}
```

可以看到，这个接口非常普通，不需要继承什么额外的接口，只需要注解一个@WebService就可以 

第2步当然是需要给这个接口提供一个实现类

```
package com.huawei.framework.webservice;

import com.huawei.framework.model.User;

public class HelloWorldImpl implements HelloWorld {

	public String sayHi(User theUser) {
		return "Hello " + theUser.getName() + " ,your age is "
				+ theUser.getAge();
	}

}
```

这个类更加简单，连注解都省了，但是如果这个类实现了不止一个接口，那么就需要加上@WebService注解，不过一般不会这样 

第3步就到了cxf出场的时候了，需要在spring配置文件中加上cxf的配置

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
       						http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
        					http://cxf.apache.org/jaxws 
        					http://cxf.apache.org/schemas/jaxws.xsd">

	<import resource="classpath:META-INF/cxf/cxf.xml" />
	<import resource="classpath:META-INF/cxf/cxf-servlet.xml" />

	<jaxws:endpoint id="helloWorld"
		implementorClass="com.huawei.framework.webservice.HelloWorldImpl"
		address="/HelloWorld" />

</beans>
```

这里只写了CXF所需的配置，spring配置文件的其他内容已省略。用到的元素是jaxws:endpoint，implementorClass属性就是我们提供的实现类，然后address属性是这个web service对外暴露的路径 

最后第4步是在web.xml中加载cxf

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="MyFramework" version="3.0">

	<display-name>MyFramework</display-name>

	<context-param>
    	<param-name>contextConfigLocation</param-name>
        <param-value> 
        	WEB-INF/beans.xml, 
            WEB-INF/cxf.xml
        </param-value> 
   	</context-param>

    <listener>  
        <listener-class>  
            org.springframework.web.context.ContextLoaderListener  
        </listener-class>  
    </listener>  

	<servlet>
		<servlet-name>framework</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>  
        	<param-name>contextConfigLocation</param-name>  
        	<param-value>WEB-INF/spring-mvc.xml</param-value>  
   		</init-param>  
   		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>framework</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

	<servlet>  
        <servlet-name>CXFServlet</servlet-name>  
        <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>  
        <load-on-startup>2</load-on-startup>  
    </servlet>  

    <servlet-mapping>  
        <servlet-name>CXFServlet</servlet-name>  
        <url-pattern>/webservice/*</url-pattern>  
    </servlet-mapping> 

</web-app>
```

这里写的比较复杂，是为了兼容spring mvc。这里把
```
/webservice/*
```

作为对外暴露的路径，和上面的web service address要结合起来，比如说

```
http://ip:port/app_name/webservice/HelloWorld
```

就会指向这个例子里声明的web service接口 

通过以上4步，就发布了一个web service，在浏览器里输入http://ip:port/appName/webservice，就会看到这个sayHi的web service了，并且cxf已经自动生成了wsdl文件 

# 调用已有的web service 

第1步是需要“获得”web service的对应接口类，这里的“获得”有多种情况。在条件允许的情况下，比如2个子系统都是自己项目组开发的，或者对方愿意提供，那么直接拷贝过来就可以了。但一般是没有这么方便的。。

那么这种情况下，就是wsdl发挥作用的时候了。只要对方发布了web service，就一定会有一个wsdl文件，那么可以根据这个wsdl文件来手写还原java接口类和所需的实体类（比如本例中的User模型类），还有一个办法就是用wsdl2java工具，生成所需的类 

在这个例子中，我直接把HelloWorld.java拷贝过来就行了，注意拷贝过来的时候，@WebService注解还是需要的 第2步是配置cxf，这里用到的是jaxws:client元素，和上面的jaxws:endpoint是对应的

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:jaxws="http://cxf.apache.org/jaxws" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
						http://www.springframework.org/schema/beans/spring-beans.xsd
						http://cxf.apache.org/jaxws 
						http://cxf.apache.org/schemas/jaxws.xsd">

	<bean id="propertyConfigurer"
		class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="locations">
			<list>
				<value>classpath:webservice_address.properties</value>
			</list>
		</property>
	</bean>

	<jaxws:client id="helloClient" serviceClass="com.huawei.webservice.client.HelloWorld"
		address="${helloworld}" />

</beans>
```

其中serviceClass属性，就是指向HelloWorld所在的位置，address是目标web service的地址，也就是http://ip:port/appName/webservice/HelloWorld 

这里有一个额外的东西，就是把所有的address，放在单独的webserviceAddress.properties文件里管理，这样如果用到的web service比较多时，可以集中维护，不需要修改spring配置文件 

第3步就是执行，十分简单

```
public class Main {

	public static void main(String[] args) {

		ApplicationContext context = new ClassPathXmlApplicationContext(
				"cxf.xml");
		HelloWorld client = (HelloWorld) context.getBean("helloClient");

		User user = new User();
		user.setName("zhengshengdong");

		String message = client.sayHi(user);
		System.out.println(message);
	}

}
```

上面可以看到，首先是启动Spring容器，然后像获取普通的bean一样，用getBean()方法实例化HelloWorld接口，然后直接在其上调用sayHi()方法即可 

在这个过程中，CXF对底层的复杂操作进行了封装，包括将实体数据封装成soap格式的消息，然后塞到http的request post里，发送到目标web service。然后从返回的http response中取出soap格式的消息，再反向解析，生成实体对象