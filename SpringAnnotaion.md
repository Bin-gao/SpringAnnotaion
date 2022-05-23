## Spring 注解

### 大纲

==容器==

容器作为整个专栏的第一大部分，内容包括：

- AnnotationConfigApplicationContext
- 组件添加
- 组件赋值
- 组件注入
- AOP
- 声明式事务

==扩展原理==

扩展原理作为整个专栏的第二大部分，内容包括：

- BeanFactoryPostProcessor
- BeanDefinitionRegistryPostProcessor
- ApplicationListener
- Spring容器创建过程

==Web==

Web作为整个专栏的第三大部分，内容包括：

- servlet3.0
- 异步请求

### 容器

在Spring容器的底层，最重要的功能就是IOC和DI，也就是控制反转和依赖注入。

#### 一、Spring IOC和DI

![image-20220517132412444](Spring注解.assets/image-20220517132412444.png)

##### 1. 通过XML配置文件注入JavaBean

##### 2. 通过注解注入JavaBean

在工程的com.meimeixia.bean包下创建一个Person类，作为测试的JavaBean，代码如下所示。

```java
package com.meimeixia.bean;

public class Person {
	
	private String name;
	private Integer age;
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Integer getAge() {
		return age;
	}
	public void setAge(Integer age) {
		this.age = age;
	}
	public Person(String name, Integer age) {
		super();
		this.name = name;
		this.age = age;
	}
	public Person() {
		super();
		// TODO Auto-generated constructor stub
	}
	@Override
	public String toString() {
		return "Person [name=" + name + ", age=" + age + "]";
	}
	
}

```

在项目的com.meimeixia.config包下创建一个MainConfig类，并在该类上添加@[Configuration](https://so.csdn.net/so/search?q=Configuration&spm=1001.2101.3001.7020)注解来标注该类是一个Spring的配置类，也就是告诉Spring它是一个配置类，最后通过@Bean注解将Person类注入到Spring的IOC容器中。

```java
package com.meimeixia.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.meimeixia.bean.Person;
/**
 * 以前配置文件的方式被替换成了配置类，即配置类==配置文件
 *
 */
// 这个配置类也是一个组件 
@Configuration // 告诉Spring这是一个配置类
public class MainConfig {

	// @Bean注解是给IOC容器中注册一个bean，类型自然就是返回值的类型，id默认是用方法名作为id
	@Bean
	public Person person() {
		return new Person("liayun", 20);
	}
	
}

```

我们修改MainTest类中的main方法，以测试通过注解注入的Person类，如下所示。

```java
package com.meimeixia;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.meimeixia.bean.Person;
import com.meimeixia.config.MainConfig;

public class MainTest {

	public static void main(String[] args) {
//		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
//		Person person = (Person) applicationContext.getBean("person");
//		System.out.println(person);
		
		ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
		Person person = applicationContext.getBean(Person.class);
		System.out.println(person);
	}
	
}
```

运行以上main方法，输出的结果信息如下图所示。

![image-20220517133845257](Spring注解.assets/image-20220517133845257.png)

#### 二、使用@ComponentScan自动扫描组件并指定扫描规则

##### 1. 写在前面

在实际项目中，我们更多的是使用Spring的包扫描功能对项目中的包进行扫描，凡是在指定的包或其子包中的类上标注了@Repository、@Service、@Controller、@Component注解的类都会被扫描到，并将这个类注入到Spring容器中。

Spring包扫描功能可以使用XML配置文件进行配置，也可以直接使用@ComponentScan注解进行设置，使用@ComponentScan注解进行设置比使用XML配置文件来配置要简单的多。

##### 2. 使用XML文件配置包扫描

我们可以在Spring的XML配置文件中配置包的扫描，在配置包扫描时，需要在Spring的XML配置文件中的beans节点中引入context标签，如下所示。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
                        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
                        http://www.springframework.org/schema/context 
                        http://www.springframework.org/schema/context/spring-context-4.2.xsd">

```

整个beans.xml配置文件中的内容如下所示。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
						http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
						http://www.springframework.org/schema/context 
						http://www.springframework.org/schema/context/spring-context-4.2.xsd">
	
	<!-- 包扫描：只要是标注了我们熟悉的@Controller、@Service、@Repository、@Component这四个注解中的任何一个的组件，它就会被自动扫描，并加进容器中 -->
	<context:component-scan base-package="com.meimeixia"></context:component-scan>
	
	<!-- 注册组件 -->
	<bean id="person" class="com.meimeixia.bean.Person">
		<property name="age" value="18"></property>
		<property name="name" value="liayun"></property>
	</bean>
	
</beans>

```

这样配置以后，只要在com.meimeixia包下，或者com.meimeixia的子包下标注了@Repository、@Service、@Controller、@Component注解的类都会被扫描到，并自动注入到Spring容器中。

此时，我们分别创建BookDao、BookService以及BookController这三个类，并在这三个类中分别添加@Repository、@Service、@Controller注解，如下所示。

- BookDao

```java
package com.meimeixia.dao;

import org.springframework.stereotype.Repository;

// 名字默认是类名首字母小写
@Repository
public class BookDao {
    
}
```

- BookService

```java
package com.meimeixia.service;

import org.springframework.stereotype.Service;

@Service
public class BookService {
    
}
```

- BookController

```java
package com.meimeixia.controller;

import org.springframework.stereotype.Controller;

@Controller
public class BookController {
    
}
```

##### 3. 使用注解配置包扫描

可以使用@ComponentScan注解来配置包扫描了。使用@ComponentScan注解配置包扫描非常非常easy！只须在我们的MainConfig类上添加@ComponentScan注解，并将扫描的包指定为com.meimeixia即可，如下所示。

```java
package com.meimeixia.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

import com.meimeixia.bean.Person;
/**
 * 以前配置文件的方式被替换成了配置类，即配置类==配置文件
 * @author liayun
 *
 */
// 这个配置类也是一个组件 
@ComponentScan(value="com.meimeixia") // value指定要扫描的包
@Configuration // 告诉Spring这是一个配置类
public class MainConfig {

	// @Bean注解是给IOC容器中注册一个bean，类型自然就是返回值的类型，id默认是用方法名作为id
	@Bean("person")
	public Person person01() {
		return new Person("liayun", 20);
	}
	
}

```

##### 4. 关于@ComponentScan注解

我们点开ComponentScan注解类并查看其源码，如下图所示。

```java
/*
 * Copyright 2002-2020 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.context.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.beans.factory.support.BeanNameGenerator;
import org.springframework.core.annotation.AliasFor;
import org.springframework.core.type.filter.TypeFilter;

/**
 * Configures component scanning directives for use with @{@link Configuration} classes.
 * Provides support parallel with Spring XML's {@code <context:component-scan>} element.
 *
 * <p>Either {@link #basePackageClasses} or {@link #basePackages} (or its alias
 * {@link #value}) may be specified to define specific packages to scan. If specific
 * packages are not defined, scanning will occur from the package of the
 * class that declares this annotation.
 *
 * <p>Note that the {@code <context:component-scan>} element has an
 * {@code annotation-config} attribute; however, this annotation does not. This is because
 * in almost all cases when using {@code @ComponentScan}, default annotation config
 * processing (e.g. processing {@code @Autowired} and friends) is assumed. Furthermore,
 * when using {@link AnnotationConfigApplicationContext}, annotation config processors are
 * always registered, meaning that any attempt to disable them at the
 * {@code @ComponentScan} level would be ignored.
 *
 * <p>See {@link Configuration @Configuration}'s Javadoc for usage examples.
 *
 * @author Chris Beams
 * @author Juergen Hoeller
 * @author Sam Brannen
 * @since 3.1
 * @see Configuration
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {

	/**
	 * Alias for {@link #basePackages}.
	 * <p>Allows for more concise annotation declarations if no other attributes
	 * are needed &mdash; for example, {@code @ComponentScan("org.my.pkg")}
	 * instead of {@code @ComponentScan(basePackages = "org.my.pkg")}.
	 */
	@AliasFor("basePackages")
	String[] value() default {};

	/**
	 * Base packages to scan for annotated components.
	 * <p>{@link #value} is an alias for (and mutually exclusive with) this
	 * attribute.
	 * <p>Use {@link #basePackageClasses} for a type-safe alternative to
	 * String-based package names.
	 */
	@AliasFor("value")
	String[] basePackages() default {};

	/**
	 * Type-safe alternative to {@link #basePackages} for specifying the packages
	 * to scan for annotated components. The package of each class specified will be scanned.
	 * <p>Consider creating a special no-op marker class or interface in each package
	 * that serves no purpose other than being referenced by this attribute.
	 */
	Class<?>[] basePackageClasses() default {};

	/**
	 * The {@link BeanNameGenerator} class to be used for naming detected components
	 * within the Spring container.
	 * <p>The default value of the {@link BeanNameGenerator} interface itself indicates
	 * that the scanner used to process this {@code @ComponentScan} annotation should
	 * use its inherited bean name generator, e.g. the default
	 * {@link AnnotationBeanNameGenerator} or any custom instance supplied to the
	 * application context at bootstrap time.
	 * @see AnnotationConfigApplicationContext#setBeanNameGenerator(BeanNameGenerator)
	 * @see AnnotationBeanNameGenerator
	 * @see FullyQualifiedAnnotationBeanNameGenerator
	 */
	Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

	/**
	 * The {@link ScopeMetadataResolver} to be used for resolving the scope of detected components.
	 */
	Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;

	/**
	 * Indicates whether proxies should be generated for detected components, which may be
	 * necessary when using scopes in a proxy-style fashion.
	 * <p>The default is defer to the default behavior of the component scanner used to
	 * execute the actual scan.
	 * <p>Note that setting this attribute overrides any value set for {@link #scopeResolver}.
	 * @see ClassPathBeanDefinitionScanner#setScopedProxyMode(ScopedProxyMode)
	 */
	ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

	/**
	 * Controls the class files eligible for component detection.
	 * <p>Consider use of {@link #includeFilters} and {@link #excludeFilters}
	 * for a more flexible approach.
	 */
	String resourcePattern() default ClassPathScanningCandidateComponentProvider.DEFAULT_RESOURCE_PATTERN;

	/**
	 * Indicates whether automatic detection of classes annotated with {@code @Component}
	 * {@code @Repository}, {@code @Service}, or {@code @Controller} should be enabled.
	 */
	boolean useDefaultFilters() default true;

	/**
	 * Specifies which types are eligible for component scanning.
	 * <p>Further narrows the set of candidate components from everything in {@link #basePackages}
	 * to everything in the base packages that matches the given filter or filters.
	 * <p>Note that these filters will be applied in addition to the default filters, if specified.
	 * Any type under the specified base packages which matches a given filter will be included,
	 * even if it does not match the default filters (i.e. is not annotated with {@code @Component}).
	 * @see #resourcePattern()
	 * @see #useDefaultFilters()
	 */
	Filter[] includeFilters() default {};

	/**
	 * Specifies which types are not eligible for component scanning.
	 * @see #resourcePattern
	 */
	Filter[] excludeFilters() default {};

	/**
	 * Specify whether scanned beans should be registered for lazy initialization.
	 * <p>Default is {@code false}; switch this to {@code true} when desired.
	 * @since 4.1
	 */
	boolean lazyInit() default false;


	/**
	 * Declares the type filter to be used as an {@linkplain ComponentScan#includeFilters
	 * include filter} or {@linkplain ComponentScan#excludeFilters exclude filter}.
	 */
	@Retention(RetentionPolicy.RUNTIME)
	@Target({})
	@interface Filter {

		/**
		 * The type of filter to use.
		 * <p>Default is {@link FilterType#ANNOTATION}.
		 * @see #classes
		 * @see #pattern
		 */
		FilterType type() default FilterType.ANNOTATION;

		/**
		 * Alias for {@link #classes}.
		 * @see #classes
		 */
		@AliasFor("classes")
		Class<?>[] value() default {};

		/**
		 * The class or classes to use as the filter.
		 * <p>The following table explains how the classes will be interpreted
		 * based on the configured value of the {@link #type} attribute.
		 * <table border="1">
		 * <tr><th>{@code FilterType}</th><th>Class Interpreted As</th></tr>
		 * <tr><td>{@link FilterType#ANNOTATION ANNOTATION}</td>
		 * <td>the annotation itself</td></tr>
		 * <tr><td>{@link FilterType#ASSIGNABLE_TYPE ASSIGNABLE_TYPE}</td>
		 * <td>the type that detected components should be assignable to</td></tr>
		 * <tr><td>{@link FilterType#CUSTOM CUSTOM}</td>
		 * <td>an implementation of {@link TypeFilter}</td></tr>
		 * </table>
		 * <p>When multiple classes are specified, <em>OR</em> logic is applied
		 * &mdash; for example, "include types annotated with {@code @Foo} OR {@code @Bar}".
		 * <p>Custom {@link TypeFilter TypeFilters} may optionally implement any of the
		 * following {@link org.springframework.beans.factory.Aware Aware} interfaces, and
		 * their respective methods will be called prior to {@link TypeFilter#match match}:
		 * <ul>
		 * <li>{@link org.springframework.context.EnvironmentAware EnvironmentAware}</li>
		 * <li>{@link org.springframework.beans.factory.BeanFactoryAware BeanFactoryAware}
		 * <li>{@link org.springframework.beans.factory.BeanClassLoaderAware BeanClassLoaderAware}
		 * <li>{@link org.springframework.context.ResourceLoaderAware ResourceLoaderAware}
		 * </ul>
		 * <p>Specifying zero classes is permitted but will have no effect on component
		 * scanning.
		 * @since 4.2
		 * @see #value
		 * @see #type
		 */
		@AliasFor("value")
		Class<?>[] classes() default {};

		/**
		 * The pattern (or patterns) to use for the filter, as an alternative
		 * to specifying a Class {@link #value}.
		 * <p>If {@link #type} is set to {@link FilterType#ASPECTJ ASPECTJ},
		 * this is an AspectJ type pattern expression. If {@link #type} is
		 * set to {@link FilterType#REGEX REGEX}, this is a regex pattern
		 * for the fully-qualified class names to match.
		 * @see #type
		 * @see #classes
		 */
		String[] pattern() default {};

	}

}

```

这里，我们着重来看ComponentScan类中的如下两个方法。

![image-20220517135101791](Spring注解.assets/image-20220517135101791.png)

includeFilters()方法指定Spring扫描的时候按照什么规则只需要包含哪些组件，

而excludeFilters()方法指定Spring扫描的时候按照什么规则排除哪些组件。

两个方法的返回值都是`Filter[]数组`，在ComponentScan注解类的内部存在`Filter注解类`，大家可以看下上面的代码。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {
    ...
        
    Filter[] includeFilters() default {};

	Filter[] excludeFilters() default {};
    
    ...
        
    @Retention(RetentionPolicy.RUNTIME)
        @Target({})
        @interface Filter {

            FilterType type() default FilterType.ANNOTATION;

            @AliasFor("classes")
            Class<?>[] value() default {};

            @AliasFor("value")
            Class<?>[] classes() default {};

            String[] pattern() default {};

        }
	...
}
```

##### 5. 扫描时排除注解标注的类

现在有这样一个需求，除了@Controller和@Service标注的组件之外，IOC容器中剩下的组件我都要，即相当于是我要排除@Controller和@Service这俩注解标注的组件。要想达到这样一个目的，我们可以在MainConfig类上通过`@ComponentScan`注解的`excludeFilters()方法`实现。

例如，我们在MainConfig类上添加了如下的注解。

```java
@ComponentScan(value="com.meimeixia", excludeFilters={
		/*
		 * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等
		 * classes：除了@Controller和@Service标注的组件之外，IOC容器中剩下的组件我都要，即相当于是我要排除@Controller和@Service这俩注解标注的组件。
		 */
		@Filter(type=FilterType.ANNOTATION, classes={Controller.class, Service.class})
}) // value指定要扫描的包

```

#####  6. 扫描时只包含注解标注的类

我们也可以使用ComponentScan注解类中的includeFilters()方法来指定Spring在进行包扫描时，只包含哪些注解标注的类。

这里需要注意的是，当我们使用`includeFilters()`方法来指定只包含哪些注解标注的类时，需要禁用掉默认的过滤规则。 还记得我们以前在XML配置文件中配置这个只包含的时候，应该怎么做吗？我们需要在XML配置文件中先配置好`use-default-filters="false`"，也就是禁用掉默认的过滤规则，因为默认的过滤规则就是扫描所有的，只有我们禁用掉默认的过滤规则之后，只包含才能生效。

现在有这样一个需求，我们需要Spring在扫描时，只包含@Controller注解标注的类。要想达到这样一个目的，我们该怎么做呢？可以在MainConfig类上添加@ComponentScan注解，设置只包含@Controller注解标注的类，并禁用掉默认的过滤规则，如下所示。

```java
@ComponentScan(value="com.meimeixia", includeFilters={
		/*
		 * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等
		 * classes：我们需要Spring在扫描时，只包含@Controller注解标注的类
		 */
		@Filter(type=FilterType.ANNOTATION, classes={Controller.class})
}, useDefaultFilters=false) // value指定要扫描的包

```

##### 7. 重复注解@Repeatable

不知道小伙伴们有没有注意到ComponentScan注解类上有一个如下所示的注解。

![image-20220517135926898](Spring注解.assets/image-20220517135926898.png)

先来看看@ComponentScans注解是个啥，如下图所示。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
public @interface ComponentScans {

	ComponentScan[] value();

}
```

可以看到，在ComponentScans注解类的内部只声明了一个返回ComponentScan[]数组的value()方法，说到这里，大家是不是就明白了，没错，这在Java 8中是一个重复注解。

如果你用的是Java 8，那么@ComponentScan注解就是一个重复注解，也就是说我们可以在一个类上重复使用这个注解，如下所示。

```java
@ComponentScan(value="com.meimeixia", includeFilters={
		/*
		 * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等
		 * classes：我们需要Spring在扫描时，只包含@Controller注解标注的类
		 */
		@Filter(type=FilterType.ANNOTATION, classes={Controller.class})
}, useDefaultFilters=false) // value指定要扫描的包
@ComponentScan(value="com.meimeixia", includeFilters={
		/*
		 * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等
		 * classes：我们需要Spring在扫描时，只包含@Service注解标注的类
		 */
		@Filter(type=FilterType.ANNOTATION, classes={Service.class})
}, useDefaultFilters=false) // value指定要扫描的包

```

##### 8. 小结

我们可以使用@ComponentScan注解来指定Spring扫描哪些包，可以使用excludeFilters()方法来指定扫描时排除哪些组件，也可以使用includeFilters()方法来指定扫描时只包含哪些组件。当使用includeFilters()方法指定只包含哪些组件时，需要禁用掉默认的过滤规则。

#### 三、自定义TypeFilter指定@ComponentScan注解的过滤规则

##### 1. 写在前面

spring的强大之处不仅仅是提供了[IOC](https://so.csdn.net/so/search?q=IOC&spm=1001.2101.3001.7020)容器，能够通过过滤规则指定排除和只包含哪些组件，它还能够通过自定义TypeFilter来指定过滤规则。如果Spring内置的过滤规则不能够满足我们的需求，那么我们便可以通过自定义TypeFilter来实现我们自己的过滤规则。

##### 2. FilterType中常用的规则

在使用@ComponentScan注解实现包扫描时，我们可以使用@Filter指定过滤规则，在@Filter中，通过type来指定过滤的类型。而@Filter注解中的type属性是一个FilterType枚举，其源码如下图所示。

```java
public enum FilterType {

	/**
	 * Filter candidates marked with a given annotation.
	 * @see org.springframework.core.type.filter.AnnotationTypeFilter
	 */
	ANNOTATION,

	/**
	 * Filter candidates assignable to a given type.
	 * @see org.springframework.core.type.filter.AssignableTypeFilter
	 */
	ASSIGNABLE_TYPE,

	/**
	 * Filter candidates matching a given AspectJ type pattern expression.
	 * @see org.springframework.core.type.filter.AspectJTypeFilter
	 */
	ASPECTJ,

	/**
	 * Filter candidates matching a given regex pattern.
	 * @see org.springframework.core.type.filter.RegexPatternTypeFilter
	 */
	REGEX,

	/** Filter candidates using a given custom
	 * {@link org.springframework.core.type.filter.TypeFilter} implementation.
	 */
	CUSTOM
}
```

##### 3. FilterType.ANNOTATION：按照注解进行包含或者排除

```java
@ComponentScan(value="com.meimeixia", includeFilters={
		/*
		 * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等
		 * classes：我们需要Spring在扫描时，只包含@Controller注解标注的类
		 */
		@Filter(type=FilterType.ANNOTATION, classes={Controller.class})
}, useDefaultFilters=false) // value指定要扫描的包

```

##### 4. FilterType.ASSIGNABLE_TYPE：按照给定的类型进行包含或者排除

```java
@ComponentScan(value="com.meimeixia", includeFilters={
		/*
		 * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等
		 */
		// 只要是BookService这种类型的组件都会被加载到容器中，不管是它的子类还是什么它的实现类。记住，只要是BookService这种类型的
		@Filter(type=FilterType.ASSIGNABLE_TYPE, classes={BookService.class})
}, useDefaultFilters=false) // value指定要扫描的包

```

##### 5. FilterType.ASPECTJ：按照ASPECTJ表达式进行包含或者排除

```java
@ComponentScan(value="com.meimeixia", includeFilters={
		/*
		 * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等
		 */
		@Filter(type=FilterType.ASPECTJ, classes={AspectJTypeFilter.class})
}, useDefaultFilters=false) // value指定要扫描的包

```

##### 6. FilterType.REGEX：按照正则表达式进行包含或者排除

```java
@ComponentScan(value="com.meimeixia", includeFilters={
		/*
		 * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等
		 */
		@Filter(type=FilterType.REGEX, classes={RegexPatternTypeFilter.class})
}, useDefaultFilters=false) // value指定要扫描的包

```

##### 7. CUSTOM：按照自定义规则进行包含或者排除

如果实现自定义规则进行过滤时，自定义规则的类必须是org.springframework.core.type.filter.TypeFilter接口的实现类。

要想按照自定义规则进行过滤，首先我们得创建org.springframework.core.type.filter.TypeFilter接口的一个实现类，例如MyTypeFilter，该实现类的代码一开始如下所示。

```java
package com.meimeixia.config;

import java.io.IOException;

import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.core.type.filter.TypeFilter;

public class MyTypeFilter implements TypeFilter {

	/**
	 * 参数：
	 * metadataReader：读取到的当前正在扫描的类的信息
	 * metadataReaderFactory：可以获取到其他任何类的信息的（工厂）
	 */
	@Override
	public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
	
		return false; // 这儿我们先让其返回false

	}

}

```

当我们实现TypeFilter接口时，需要实现该接口中的match()方法，match()方法的返回值为boolean类型。

当返回true时，表示符合规则，会包含在Spring容器中；

当返回false时，表示不符合规则，那就是一个都不匹配，自然就都不会被包含在Spring容器中。

然后，使用@ComponentScan注解进行如下配置。

```java
@ComponentScan(value="com.meimeixia", includeFilters={
		/*
		 * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等
		 */
		// 指定新的过滤规则，这个过滤规则是我们自个自定义的，过滤规则就是由我们这个自定义的MyTypeFilter类返回true或者false来代表匹配还是没匹配
		@Filter(type=FilterType.CUSTOM, classes={MyTypeFilter.class})
}, useDefaultFilters=false) // value指定要扫描的包

```

##### 8. 实现自定义过滤规则

从上面可以知道，我们在项目的com.meimeixia.config包下新建了一个类，即MyTypeFilter，它实现了org.springframework.core.type.filter.TypeFilter接口。

此时，我们先在MyTypeFilter类中打印出当前正在扫描的类名，如下所示。

```java
package com.meimeixia.config;

import java.io.IOException;

import org.springframework.core.io.Resource;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.ClassMetadata;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.core.type.filter.TypeFilter;

public class MyTypeFilter implements TypeFilter {

	/**
	 * 参数：
	 * metadataReader：读取到的当前正在扫描的类的信息
	 * metadataReaderFactory：可以获取到其他任何类的信息的（工厂）
	 */
	@Override
	public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
		// 获取当前类注解的信息
		AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
		// 获取当前正在扫描的类的类信息，比如说它的类型是什么啊，它实现了什么接口啊之类的
		ClassMetadata classMetadata = metadataReader.getClassMetadata();
		// 获取当前类的资源信息，比如说类的路径等信息
		Resource resource = metadataReader.getResource();
		// 获取当前正在扫描的类的类名
		String className = classMetadata.getClassName();
		System.out.println("--->" + className);
		
		return false;
	}

}

```

然后，我们在MainConfig类中配置自定义过滤规则，如下所示。

```java
package com.meimeixia.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.ComponentScans;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

import com.meimeixia.bean.Person;
/**
 * 以前配置文件的方式被替换成了配置类，即配置类==配置文件
 * @author liayun
 *
 */
// 这个配置类也是一个组件
@ComponentScans(value={
		@ComponentScan(value="com.meimeixia", includeFilters={
				/*
				 * type：指定你要排除的规则，是按照注解进行排除，还是按照给定的类型进行排除，还是按照正则表达式进行排除，等等
				 */
				// 指定新的过滤规则，这个过滤规则是我们自个自定义的，过滤规则就是由我们这个自定义的MyTypeFilter类返回true或者false来代表匹配还是没匹配
				@Filter(type=FilterType.CUSTOM, classes={MyTypeFilter.class})
		}, useDefaultFilters=false) // value指定要扫描的包
})
@Configuration // 告诉Spring这是一个配置类
public class MainConfig {

	// @Bean注解是给IOC容器中注册一个bean，类型自然就是返回值的类型，id默认是用方法名作为id
	@Bean("person")
	public Person person01() {
		return new Person("liayun", 20);
	}
	
}

```

接着，我们运行IOCTest类中的test01()方法进行测试，该方法的完整代码如下所示。

```java
@SuppressWarnings("resource")
@Test
public void test01() {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
    // 我们现在就来看一下IOC容器中有哪些bean，即容器中所有bean定义的名字
    String[] definitionNames = applicationContext.getBeanDefinitionNames();
    for (String name : definitionNames) {
        System.out.println(name);
    }
}

```

此时，输出的结果信息如下图所示。

![image-20220517141134050](Spring注解.assets/image-20220517141134050.png)

可以看到，已经输出了当前正在扫描的类的名称，同时，除了Spring内置的bean的名称之外，只输出了mainConfig和person，而没有输出使用@Repository、@Service、@Controller这些注解标注的组件的名称。这是因为当前MainConfig类上标注的@ComponentScan注解是使用的自定义规则，而在自定义规则的实现类（即MyTypeFilter类）中，直接返回了false，那么就是一个都不匹配了，自然所有的bean就都没被包含进去容器中了。

我们可以在MyTypeFilter类中简单的实现一个规则，例如，当前扫描的类名称中包含有"er"字符串的，就返回true，否则就返回false。此时，MyTypeFilter类中match()方法的实现代码如下所示。

```java
package com.meimeixia.config;

import java.io.IOException;

import org.springframework.core.io.Resource;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.ClassMetadata;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.core.type.filter.TypeFilter;

public class MyTypeFilter implements TypeFilter {

	/**
	 * 参数：
	 * metadataReader：读取到的当前正在扫描的类的信息
	 * metadataReaderFactory：可以获取到其他任何类的信息的（工厂）
	 */
	@Override
	public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
		// 获取当前类注解的信息
		AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
		// 获取当前正在扫描的类的类信息，比如说它的类型是什么啊，它实现了什么接口啊之类的
		ClassMetadata classMetadata = metadataReader.getClassMetadata();
		// 获取当前类的资源信息，比如说类的路径等信息
		Resource resource = metadataReader.getResource();
		// 获取当前正在扫描的类的类名
		String className = classMetadata.getClassName();
		System.out.println("--->" + className);
		
		// 现在来指定一个规则
		if (className.contains("er")) {
			return true; // 匹配成功，就会被包含在容器中
		}
		
		return false; // 匹配不成功，所有的bean都会被排除
	}
}
```

此时，在com.meimeixia包下的所有类都会通过MyTypeFilter类中的match()方法来验证类名中是否包含有"er"字符串，若包含则返回true，否则返回false。

最后，我们再次运行IOCTest类中的test01()方法进行测试，输出的结果信息如下图所示。

![image-20220517200836436](Spring注解.assets/image-20220517200836436.png)

此时，结果信息中输出了使用@Service和@Controller这俩注解标注的组件的名称，分别是bookController和bookService。

从以上输出的结果信息中，你还可以看到输出了一个myTypeFilter，你不禁要问了，为什么会有myTypeFilter呢？这就是因为我们现在扫描的是com.meimeixia包，该包下的每一个类都会进到这个自定义规则里面进行匹配，若匹配成功，则就会被包含在容器中。

#### 四、使用@Scope注解设置组件的作用域

##### 1. 写在前面

Spring容器中的组件默认是单例的，在Spring启动时就会实例化并初始化这些对象，并将其放到Spring容器中，之后，每次获取对象时，直接从Spring容器中获取，而不再创建对象。

如果每次从Spring容器中获取对象时，都要创建一个新的实例对象，那么该如何处理呢？此时就需要使用@Scope注解来设置组件的作用域了。

##### 2. @Scopre 注解概述

@Scope注解能够设置组件的作用域，我们先来看看@Scope注解类的源码，如下所示。

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Scope {

	/**
	 * Alias for {@link #scopeName}.
	 * @see #scopeName
	 */
	@AliasFor("scopeName")
	String value() default "";

	/**
	 * Specifies the name of the scope to use for the annotated component/bean.
	 * <p>Defaults to an empty string ({@code ""}) which implies
	 * {@link ConfigurableBeanFactory#SCOPE_SINGLETON SCOPE_SINGLETON}.
	 * @since 4.2
	 * @see ConfigurableBeanFactory#SCOPE_PROTOTYPE
	 * @see ConfigurableBeanFactory#SCOPE_SINGLETON
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_SESSION
	 * @see #value
	 */
	@AliasFor("value")
	String scopeName() default "";

	/**
	 * Specifies whether a component should be configured as a scoped proxy
	 * and if so, whether the proxy should be interface-based or subclass-based.
	 * <p>Defaults to {@link ScopedProxyMode#DEFAULT}, which typically indicates
	 * that no scoped proxy should be created unless a different default
	 * has been configured at the component-scan instruction level.
	 * <p>Analogous to {@code <aop:scoped-proxy/>} support in Spring XML.
	 * @see ScopedProxyMode
	 */
	ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;

}

```

从@Scope注解类的源码中可以看出，在@Scope注解中可以设置如下值：

@since 4.2

1. @see ConfigurableBeanFactory#SCOPE_PROTOTYPE
2. @see ConfigurableBeanFactory#SCOPE_SINGLETON
3. @see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST
4. @see org.springframework.web.context.WebApplicationContext#SCOPE_SESSION



首先，我们查看一下ConfigurableBeanFactory接口的源码，发现在该接口中存在两个常量的定义，如下所示。

```java
public interface ConfigurableBeanFactory extends HierarchicalBeanFactory, SingletonBeanRegistry {

	/**
	 * Scope identifier for the standard singleton scope: {@value}.
	 * <p>Custom scopes can be added via {@code registerScope}.
	 * @see #registerScope
	 */
	String SCOPE_SINGLETON = "singleton";

	/**
	 * Scope identifier for the standard prototype scope: {@value}.
	 * <p>Custom scopes can be added via {@code registerScope}.
	 * @see #registerScope
	 */
	String SCOPE_PROTOTYPE = "prototype";
	...
}
```

没错，SCOPE_SINGLETON就是singleton，而SCOPE_PROTOTYPE就是prototype。

当我们使用Web容器来运行Spring应用时，在@Scope注解中可以设置WebApplicationContext类中的SCOPE_REQUEST和SCOPE_SESSION这俩的值，而SCOPE_REQUEST的值就是request，SCOPE_SESSION的值就是session。

综上。在@Scope注解中的取值如下所示。

![](Spring注解.assets/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3llcmVueXVhbl9wa3U=,size_16,color_FFFFFF,t_70#pic_center.png)

其中，request和session作用域是需要Web环境来支持的，这两个值基本上使用不到。当我们使用Web容器来运行Spring应用时，如果需要将组件的实例对象的作用域设置为request和session，那么我们通常会使用

request.setAttribute("key", object);

和

session.setAttribute("key", object);

这两种形式来将对象实例设置到request和session中，而不会使用@Scope注解来进行设置。

##### 3. 单实例bean作用域

首先，创建一个配置类，例如MainConfig2，然后在该配置类中实例化一个Person对象，并将其放置在Spring容器中，如下所示。

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.meimeixia.bean.Person;

@Configuration
public class MainConfig2 {
	
	@Bean("person")
	public Person person() {
		return new Person("美美侠", 25);
	}
	
}
```

接着，在IOCTest类中创建一个test02()测试方法，在该测试方法中创建一个AnnotationConfigApplicationContext对象，创建完毕后，从Spring容器中按照id获取两个Person对象，并判断这两个对象是否是同一个对象，代码如下所示。

```java
@SuppressWarnings("resource")
@Test
public void test02() {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
    
    // 获取到的这个Person对象默认是单实例的，因为在IOC容器中给我们加的这些组件默认都是单实例的，
    // 所以说在这儿我们无论多少次获取，获取到的都是我们之前new的那个实例对象
    Person person = (Person) applicationContext.getBean("person");
    Person person2 = (Person) applicationContext.getBean("person");
    System.out.println(person == person2);
}

```

由于对象在Spring容器中默认是单实例的，所以，Spring容器在启动时就会将实例对象加载到Spring容器中，之后，每次从Spring容器中获取实例对象，都是直接将对象返回，而不必再创建新的实例对象了。很显然，此时运行test02()方法之后会输出true.

这也正好验证了我们的结论：**对象在Spring容器中默认是单实例的，Spring容器在启动时就会将实例对象加载到Spring容器中，之后，每次从Spring容器中获取实例对象，都是直接将对象返回，而不必再创建新的实例对象了**。

##### 4. 多实例bean作用域

修改Spring容器中组件的作用域，我们需要借助于@Scope注解。此时，我们将MainConfig2配置类中Person对象的作用域修改成prototype，如下所示。

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

@Configuration
public class MainConfig2 {
	
	@Scope("prototype") // 通过@Scope注解来指定该bean的作用范围，也可以说成是调整作用域
	@Bean("person")
	public Person person() {
		return new Person("美美侠", 25);
	}
	
}
```

 此时，我们再次运行IOCTest类中的test02()方法，你觉得从Spring容器中获取到的person对象和person2对象还是同一个对象吗？

![image-20220518105756758](Spring注解.assets/image-20220518105756758.png)

很显然不是，从以上输出结果中也可以看出，此时，输出的person对象和person2对象已经不是同一个对象了。

##### 5. 单实例bean作用域何时创建对象？

Spring容器在启动时，将单实例组件实例化之后，会即刻加载到Spring容器中，以后每次从容器中获取组件实例对象时，都是直接返回相应的对象，而不必再创建新的对象了。

##### 6. 多实例bean作用域何时创建对象？

在创建Spring容器时，并不会去实例化和加载多实例对象，当向Spring容器中获取实例对象时，Spring容器才会实例化对象，再将其加载到Spring容器中去。

当对象的Scope作用域为多实例时，每次向Spring容器获取对象时，它都会创建一个新的对象并返回。并且每个实例对象都不相等。

##### 7. 单实例bean注意的事项

单实例bean是整个应用所共享的，所以需要考虑到线程安全问题，之前在玩SpringMVC的时候，SpringMVC中的Controller默认是单例的，有些开发者在Controller中创建了一些变量，那么这些变量实际上就变成共享的了，Controller又可能会被很多线程同时访问，这些线程并发去修改Controller中的共享变量，此时很有可能会出现数据错乱的问题，所以使用的时候需要特别注意。

##### 8. 多实例bean注意的事项

**多实例bean每次获取的时候都会重新创建，如果这个bean比较复杂，创建时间比较长，那么就会影响系统的性能，因此这个地方需要注意点。**

##### 9. 自定义Scope

自定义Scope主要分为三个步骤，如下所示。

第一步，实现Scope接口。我们先来看下Scope接口的源码，如下所示。

```java
package org.springframework.beans.factory.config;

import org.springframework.beans.factory.ObjectFactory;

public interface Scope {

	/**
	 * 返回当前作用域中name对应的bean对象
	 * @param name 需要检索的bean对象的名称
	 * @param objectFactory 如果name对应的bean对象在当前作用域中没有找到，那么可以调用这个objectFactory来创建这个对象
	 */
	Object get(String name, ObjectFactory<?> objectFactory);

	/**
	 * 将name对应的bean对象从当前作用域中移除
	 */
	Object remove(String name);

	/**
	 * 用于注册销毁回调，若想要销毁相应的对象，则由Spring容器注册相应的销毁回调，而由自定义作用域选择是不是要销毁相应的对象
	 */
	void registerDestructionCallback(String name, Runnable callback);

	/**
	 * 用于解析相应的上下文数据，比如request作用域将返回request中的属性
	 */
	Object resolveContextualObject(String key);

	/**
	 * 作用域的会话标识，比如session作用域的会话标识是sessionId
	 */
	String getConversationId();

}

```

第二步，将自定义Scope注册到容器中。此时，需要调用org.springframework.beans.factory.config.ConfigurableBeanFactory#registerScope

第三步，使用自定义的作用域。也就是在定义bean的时候，指定bean的scope属性为自定义的作用域名称。

#### 五、懒加载

懒加载，也称延时加载，仅针对单实例bean生效。 单实例bean是在Spring容器启动的时候加载的，添加`@Lazy`注解后就会延迟加载，**在Spring容器启动的时候并不会加载，而是在第一次使用此bean的时候才会加载，但当你多次获取bean的时候并不会重复加载，只是在第一次获取的时候才会加载，这不是延迟加载的特性，而是单实例bean的特性。**

```java
@Lazy
@Bean("person")
public Person person() {
    System.out.println("给容器中添加咱们这个Person对象...");
    return new Person("美美侠", 25);
}
```

#### 六、如何按照条件向Spring容器中注册bean？

##### 1. Conditional注解概述

@Conditional注解可以按照一定的条件进行判断，满足条件向容器中注册bean，不满足条件就不向容器中注册bean。

@Conditional注解是由Spring Framework提供的一个注解，它位于 org.springframework.context.annotation包内，定义如下。

```java
package org.springframework.context.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {

	Class<? extends Condition>[] value();

}

```

从@Conditional注解的源码来看，@Conditional注解不仅可以添加到类上，也可以添加到方法上。在@Conditional注解中，还存在着一个Condition类型或者其子类型的Class对象数组，Condition是个啥呢？我们点进去看一下。

```java
package org.springframework.context.annotation;

import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.core.type.AnnotatedTypeMetadata;

@FunctionalInterface
public interface Condition {

	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);

}
```

可以看到，它是一个接口。所以，我们使用@Conditional注解时，需要写一个类来实现Spring提供的Condition接口，它会匹配@Conditional所符合的方法。

我们可以在哪些场合使用@Conditional注解呢？@Conditional注解的使用场景如下图所示。

![image-20220518111439532](Spring注解.assets/image-20220518111439532.png)

##### 2. 带条件注册bean

现在，我们就要提出一个新的需求了，比如，如果当前操作系统是Windows操作系统，那么就向Spring容器中注册名称为bill的Person对象；如果当前操作系统是Linux操作系统，那么就向Spring容器中注册名称为linus的Person对象。要想实现这个需求，我们就得要使用@Conditional注解了。

使用Spring中的AnnotationConfigApplicationContext类就能够获取到当前操作系统的类型，如下所示。

```java
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
ConfigurableEnvironment environment = applicationContext.getEnvironment(); // 拿到IOC运行环境
// 动态获取坏境变量的值，例如操作系统的名字
String property = environment.getProperty("os.name"); // 获取操作系统的名字，例如Windows 10
System.out.println(property);

```

我们将上述代码整合到IOCTest类中的test06()方法中，如下所示。

```java
@Test
public void test06() {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
    // 我们现在就来看一下IOC容器中Person这种类型的bean都有哪些
    String[] namesForType = applicationContext.getBeanNamesForType(Person.class);
    ConfigurableEnvironment environment = applicationContext.getEnvironment(); // 拿到IOC运行环境
    // 动态获取坏境变量的值，例如操作系统的名字
    String property = environment.getProperty("os.name"); // 获取操作系统的名字，例如Windows 10
    System.out.println(property);
    
    for (String name : namesForType) {
        System.out.println(name);
    }
    
    Map<String, Person> persons = applicationContext.getBeansOfType(Person.class); // 找到这个Person类型的所有bean
    System.out.println(persons);
}

```

接下来就要来实现上面那个需求了。此时，我们可以借助Spring中的@Conditional注解来实现。

要想使用@Conditional注解，我们需要实现Condition接口来为@Conditional注解设置条件，所以，这里我们创建了两个实现Condition接口的类，它们分别是LinuxCondition和WindowsCondition，如下所示。

- LinuxCondition

```java
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

/**
* 判断操作系统是否是Linux系统
* @author liayun
*
*/
public class LinuxCondition implements Condition {

    /**
    * ConditionContext：判断条件能使用的上下文（环境）
    * AnnotatedTypeMetadata：当前标注了@Conditional注解的注释信息
    */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 判断操作系统是否是Linux系统
        
        // 1. 获取到bean的创建工厂（能获取到IOC容器使用到的BeanFactory，它就是创建对象以及进行装配的工厂）
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        // 2. 获取到类加载器
        ClassLoader classLoader = context.getClassLoader();
        // 3. 获取当前环境信息，它里面就封装了我们这个当前运行时的一些信息，包括环境变量，以及包括虚拟机的一些变量
        Environment environment = context.getEnvironment();
        // 4. 获取到bean定义的注册类
        BeanDefinitionRegistry registry = context.getRegistry();
        
        String property = environment.getProperty("os.name");
        if (property.contains("linux")) {
            return true;
        }
        
        return false;
    }

}

```

- 说说BeanDefinitionRegistry

context的getRegistry()方法获取到的bean定义的注册对象，即BeanDefinitionRegistry对象了。它到底是个啥呢？我们可以点进去看一下它的源码，如下所示，可以看到它是一个接口。

![image-20220518112326087](Spring注解.assets/image-20220518112326087.png)

Spring容器中所有的bean都可以通过BeanDefinitionRegistry对象来进行注册，因此我们可以通过它来查看Spring容器中到底注册了哪些bean。而且仔细查看一下BeanDefinitionRegistry接口中声明的各个方法，你就知道我们还可以通过BeanDefinitionRegistry对象向Spring容器中注册一个bean、移除一个bean、查询某一个bean的定义信息或者判断Spring容器中是否包含有某一个bean的定义。

因此，我们可以在这儿做更多的判断，比如说我可以判断一下Spring容器中是不是包含有某一个bean，就像下面这样，如果Spring容器中果真包含有名称为person的bean，那么就做些什么事情，如果没包含，那么我们还可以利用BeanDefinitionRegistry对象向Spring容器中注册一个bean。

```java
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

/**
* 判断操作系统是否是Linux系统
* @author liayun
*
*/
public class LinuxCondition implements Condition {

    /**
    * ConditionContext：判断条件能使用的上下文（环境）
    * AnnotatedTypeMetadata：当前标注了@Conditional注解的注释信息
    */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 判断操作系统是否是Linux系统
        
        // 1. 获取到bean的创建工厂（能获取到IOC容器使用到的BeanFactory，它就是创建对象以及进行装配的工厂）
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        // 2. 获取到类加载器
        ClassLoader classLoader = context.getClassLoader();
        // 3. 获取当前环境信息，它里面就封装了我们这个当前运行时的一些信息，包括环境变量，以及包括虚拟机的一些变量
        Environment environment = context.getEnvironment();
        // 4. 获取到bean定义的注册类
        BeanDefinitionRegistry registry = context.getRegistry();
        
        // 在这儿还可以做更多的判断，比如说我判断一下Spring容器中是不是包含有某一个bean，就像下面这样，如果Spring容器中果真包含有名称为person的bean，那么就做些什么事情...
        boolean definition = registry.containsBeanDefinition("person");

        String property = environment.getProperty("os.name");
        if (property.contains("linux")) {
            return true;
        }
        
        return false;
    }

}

```

- WindowsCondition

```java
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

/**
* 判断操作系统是否是Windows系统
* @author liayun
*
*/
public class WindowsCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        if (property.contains("Windows")) {
            return true;
        }
        return false;
    }

}

```

然后，我们就需要在MainConfig2配置类中使用@Conditional注解添加条件了。添加该注解后的方法如下所示。

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Lazy;

import com.meimeixia.bean.Person;
import com.meimeixia.condition.LinuxCondition;
import com.meimeixia.condition.WindowsCondition;

@Configuration
public class MainConfig2 {
	
	@Lazy
	@Bean("person")
	public Person person() {
		System.out.println("给容器中添加咱们这个Person对象...");
		return new Person("美美侠", 25);
	}

	@Conditional({WindowsCondition.class})
	@Bean("bill")
	public Person person01() {
		return new Person("Bill Gates", 62);
	}
	
	@Conditional({LinuxCondition.class})
	@Bean("linus")
	public Person person02() {
		return new Person("linus", 48);
	}
	
}
```

##### @Conditional的扩展注解

![](readme.assets/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3llcmVueXVhbl9wa3U=,size_16,color_FFFFFF,t_70#pic_center-16528444430906.png)

#### 七、使用@Import注解给容器中快速导入一个组件

##### 1. 写在前面

我们知道，我们可以将一些bean组件交由Spring来管理，并且Spring还支持单实例bean和多实例bean。我们自己写的类，自然是可以通过包扫描+给组件标注注解（@Controller、@Servcie、@Repository、@Component）的形式将其注册到IOC容器中，但这种方式比较有局限性，局限于我们自己写的类，比方说我们自己写的类，我们当然能把以上这些注解标注上去了。

那么如果不是我们自己写的类，比如说我们在项目中会经常引入一些第三方的类库，我们需要将这些第三方类库中的类注册到Spring容器中，该怎么办呢？此时，我们就可以使用@Bean和@Import注解将这些类快速的导入Spring容器中。

接下来，我们来一起探讨下如何使用@Import注解给容器中快速导入一个组件。

##### 2. 注册Bean的方式

向Spring容器中注册bean通常有以下几种方式：

1. 包扫描+给组件标注注解（@Controller、@Servcie、@Repository、@Component），但这种方式比较有局限性，局限于我们自己写的类
2. @Bean注解，通常用于导入第三方包中的组件
3. @Import注解，快速向Spring容器中导入一个组件

##### 3. @Import注解概述

Spring 3.0之前，创建bean可以通过XML配置文件与扫描特定包下面的类来将类注入到Spring IOC容器内。而在Spring 3.0之后提供了JavaConfig的方式，也就是将IOC容器里面bean的元信息以Java代码的方式进行描述，然后我们可以通过@Configuration与@Bean这两个注解配合使用来将原来配置在XML文件里面的bean通过Java代码的方式进行描述。

@Import注解提供了@Bean注解的功能，同时还有XML配置文件里面标签组织多个分散的XML文件的功能，当然在这里是组织多个分散的@Configuration，因为一个配置类就约等于一个XML配置文件。

源码如下：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

	/**
	 * {@link Configuration @Configuration}, {@link ImportSelector},
	 * {@link ImportBeanDefinitionRegistrar}, or regular component classes to import.
	 */
	Class<?>[] value();

}
```

从源码里面可以看出@Import可以配合Configuration、ImportSelector以及ImportBeanDefinitionRegistrar来使用，下面的or表示也可以把Import当成普通的bean来使用。

注意：@Import注解只允许放到类上面，不允许放到方法上。

下面我们来看一下该注解的具体使用方式。

##### 4. Import 注解的使用方式

@Import注解的三种用法主要包括：

1. 直接填写class数组的方式
2. **ImportSelector接口的方式，即批量导入，这是重点**
3. ImportBeanDefinitionRegistrar接口方式，即手工注册bean到容器中

##### 5. 使用Import注解

首先，我们创建一个Color类，这个类是一个空类，没有成员变量和方法，如下所示。

```java
package com.meimeixia.bean;

public class Color {

}

```

然后，我们在IOCTest类中创建一个testImport()方法，在其中输出Spring容器中所有bean定义的名字，来查看是否存在Color类对应的bean实例，以此来判断Spring容器中是否注册有Color类对应的bean实例。

```java
@Test
public void testImport() {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
    String[] definitionNames = applicationContext.getBeanDefinitionNames();
    for (String name : definitionNames) {
        System.out.println(name);
    }
}
```

运行以上testImport()方法之后，输出的结果信息如下所示。

![image-20220518131008973](Spring注解.assets/image-20220518131008973.png)

可以看到Spring容器中并没有Color类对应的bean实例。

>  使用@Import注解时的效果

首先，我们在MainConfig2配置类上添加一个@Import注解，并将Color类填写到该注解中，如下所示。

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.Lazy;

import com.meimeixia.bean.Color;
import com.meimeixia.bean.Person;
import com.meimeixia.condition.LinuxCondition;
import com.meimeixia.condition.WindowsCondition;

// 对配置类中的组件进行统一设置
@Conditional({WindowsCondition.class}) // 满足当前条件，这个类中配置的所有bean注册才能生效
@Configuration
@Import(Color.class) // @Import快速地导入组件，id默认是组件的全类名
public class MainConfig2 {
	
	@Lazy
	@Bean("person")
	public Person person() {
		System.out.println("给容器中添加咱们这个Person对象...");
		return new Person("美美侠", 25);
	}
	
	@Bean("bill")
	public Person person01() {
		return new Person("Bill Gates", 62);
	}
	
	@Conditional({LinuxCondition.class})
	@Bean("linus")
	public Person person02() {
		return new Person("linus", 48);
	}
	
}

```

然后，我们运行IOCTest类中的testImport()方法，会发现输出的结果信息如下所示。

![image-20220518131149124](Spring注解.assets/image-20220518131149124.png)

可以看到，输出结果中打印了com.meimeixia.bean.Color，说明使用@Import注解快速地导入组件时，容器中就会自动注册这个组件，并且id默认是组件的全类名。

`@Import注解还支持同时导入多个类`

#### 八、在@Import注解中使用ImportSelector接口导入bean

##### 1. ImportSelector接口概述

ImportSelector接口是Spring中导入外部配置的核心接口，在Spring Boot的自动化配置和@EnableXXX（功能性注解）都有它的存在。我们先来看一下ImportSelector接口的源码，如下所示。

```java
public interface ImportSelector {

	/**
	 * Select and return the names of which class(es) should be imported based on
	 * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
	 * @return the class names, or an empty array if none
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);

	/**
	 * Return a predicate for excluding classes from the import candidates, to be
	 * transitively applied to all classes found through this selector's imports.
	 * <p>If this predicate returns {@code true} for a given fully-qualified
	 * class name, said class will not be considered as an imported configuration
	 * class, bypassing class file loading as well as metadata introspection.
	 * @return the filter predicate for fully-qualified candidate class names
	 * of transitively imported configuration classes, or {@code null} if none
	 * @since 5.2.4
	 */
	@Nullable
	default Predicate<String> getExclusionFilter() {
		return null;
	}

}
```



在ImportSelector接口的selectImports()方法中，存在一个AnnotationMetadata类型的参数，这个参数能够`获取到当前标注@Import注解的类的所有注解信息`，也就是说不仅能获取到@Import注解里面的信息，还能获取到其他注解的信息。

##### 2. ImportSelector接口实例

首先，我们创建一个MyImportSelector类实现ImportSelector接口，如下所示，先在selectImports()方法中返回null，后面我们再来改。

```java
package com.lbgao.condition;

import org.springframework.context.annotation.ImportSelector;
import org.springframework.core.type.AnnotationMetadata;

import java.util.function.Predicate;

/**
 * @Auther: lbgao
 * @Date: 2022/5/18 13:54
 */

public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return null;
    }

    @Override
    public Predicate<String> getExclusionFilter() {
        return ImportSelector.super.getExclusionFilter();
    }
}

```

然后，在MainConfig01配置类的@Import注解中，导入MyImportSelector类，如下所示。

```java
package com.lbgao.config;

import com.lbgao.condition.MyImportSelector;
import com.lbgao.pojo.Color;
import com.lbgao.pojo.Person;
import com.lbgao.pojo.Red;
import org.springframework.context.annotation.*;

/**
 * @Auther: lbgao
 * @Date: 2022/5/18 13:39
 */

// 对配置类中的组件进行统一设置
@Configuration
@Import({Color.class, Red.class, MyImportSelector.class}) // @Import快速地导入组件，id默认是组件的全类名
public class MainConfig01 {
    @Lazy
    @Bean("person")
    public Person person() {
        System.out.println("给容器中添加咱们这个Person对象...");
        return new Person("美美侠", 25);
    }
}

```

至于使用MyImportSelector类要导入哪些bean，就需要你在MyImportSelector类的selectImports()方法中进行设置了，只须在`MyImportSelector`类的`selectImports()`方法中返回要导入的`类的全类名`（包名+类名）即可。

接着，我们就要运行IOCTest类中的testImport()方法了，在运行该方法之前，咱们先在MyImportSelector类的selectImports()方法处打一个断点，debug调试一下，如下图所示。

![image-20220518140047403](Spring注解.assets/image-20220518140047403.png)

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {

        System.out.println("MyImportSelector#selectImports");
        MergedAnnotations annotations = importingClassMetadata.getAnnotations();
        for (MergedAnnotation<Annotation> annotation : annotations) {
            System.out.println(annotation.toString() + "++");
        }
        return new String[]{"com.lbgao.pojo.Yellow"};
    }

    @Override
    public Predicate<String> getExclusionFilter() {
        return ImportSelector.super.getExclusionFilter();
    }
}
```

![image-20220518140922445](Spring注解.assets/image-20220518140922445.png)

#### 九、在@Import注解中使用ImportBeanDefinitioRegistrar向容器注册bean

##### 1. ImportBeanDefinitionRegistrar接口的简要介绍

源码

```java
public interface ImportBeanDefinitionRegistrar {
	default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry,
			BeanNameGenerator importBeanNameGenerator) {

		registerBeanDefinitions(importingClassMetadata, registry);
	}

	default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
	}

}
```

ImportBeanDefinitionRegistrar需要配合@Configuration和@Import这俩注解，其中，@Configuration注解定义Java格式的Spring配置文件，@Import注解导入实现了ImportBeanDefinitionRegistrar接口的类。

##### 2. ImportBeanDefinitionRegistrar接口实例

既然ImportBeanDefinitionRegistrar是一个接口，那我们就创建一个MyImportBeanDefinitionRegistrar类，去实现ImportBeanDefinitionRegistrar接口，如下所示。

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    /**
     * AnnotationMetadata：当前类的注解信息
     * BeanDefinitionRegistry：BeanDefinition注册类
     *
     * 我们可以通过调用BeanDefinitionRegistry接口中的registerBeanDefinition方法，手动注册所有需要添加到容器中的bean
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        boolean definition = registry.containsBeanDefinition("com.lbgao.pojo.Red");
        boolean definition2 = registry.containsBeanDefinition("com.lbgao.pojo.Color");
        if (definition && definition2) {
            // 指定bean的定义信息，包括bean的类型、作用域等等
            // RootBeanDefinition是BeanDefinition接口的一个实现类
            RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class); // bean的定义信息
            // 注册一个bean，并且指定bean的名称
            registry.registerBeanDefinition("rainBow", beanDefinition);
        }
    }

}
```

以上registerBeanDefinitions()方法的实现逻辑很简单，就是判断Spring容器中是否同时存在以com.lbgao.pojo.Red命名的bean和以com.lbgao.pojo.Color命名的bean，如果真的同时存在，那么向Spring容器中注入一个以rainBow命名的bean。

最后，我们运行IOCTest类中的testImport()方法来进行测试，输出结果信息如下所示。

![image-20220520160609547](Spring注解.assets/image-20220520160609547.png)

#### 十、如何使用FactBean向Spring容器中注册Bean？

##### 1. FactoryBean概述

一般情况下，Spring是通过反射机制利用bean的class属性指定实现类来实例化bean的。在某些情况下，实例化bean过程比较复杂，如果按照传统的方式，那么则需要在标签中提供大量的配置信息，配置方式的灵活性是受限的，这时采用编码的方式可以得到一个更加简单的方案。

Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化bean的逻辑。

FactoryBean接口对于Spring框架来说占有非常重要的地位，Spring自身就提供了70多个FactoryBean接口的实现。它们隐藏了实例化一些复杂bean的细节，给上层应用带来了便利。从Spring 3.0开始，FactoryBean开始支持泛型，即接口声明改为FactoryBean<T>的形式。

```java
package org.springframework.beans.factory;

import org.springframework.lang.Nullable;

public interface FactoryBean<T> {

	/**
	 * The name of an attribute that can be
	 * {@link org.springframework.core.AttributeAccessor#setAttribute set} on a
	 * {@link org.springframework.beans.factory.config.BeanDefinition} so that
	 * factory beans can signal their object type when it can't be deduced from
	 * the factory bean class.
	 * @since 5.2
	 */
	String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

	@Nullable
	T getObject() throws Exception;

	
	@Nullable
	Class<?> getObjectType();

	default boolean isSingleton() {
		return true;
	}

}

```

这里，需要注意的是：`当配置文件中标签的class属性配置的实现类是FactoryBean时，通过 getBean()方法返回的不是FactoryBean本身，而是FactoryBean#getObject()方法所返回的对象，相当于FactoryBean#getObject()代理了getBean()方法。`

##### 2. FactroyBean案例

首先，创建一个ColorFactoryBean类，它得实现FactoryBean接口，如下所示

```java
package com.lbgao.bean;

import com.lbgao.pojo.Color;
import org.springframework.beans.factory.FactoryBean;

/**
 * @Auther: lbgao
 * @Date: 2022/5/20 16:13
 */

public class ColorFactroyBean implements FactoryBean<Color> {
    // 返回一个Color对象，这个对象会添加到容器中
    @Override
    public Color getObject() throws Exception {
        // TODO Auto-generated method stub
        System.out.println("ColorFactoryBean...getObject...");
        return new Color();
    }

    @Override
    public Class<?> getObjectType() {
        // TODO Auto-generated method stub
        return Color.class; // 返回这个对象的类型
    }

    // 是单例吗？
    // 如果返回true，那么代表这个bean是单实例，在容器中只会保存一份；
    // 如果返回false，那么代表这个bean是多实例，每次获取都会创建一个新的bean
    @Override
    public boolean isSingleton() {
        // TODO Auto-generated method stub
        return false;
    }
}

```

![image-20220520162349208](Spring注解.assets/image-20220520162349208.png)

这里需要小伙伴们注意的是：我在这里使用@Bean注解向Spring容器中注册的是ColorFactoryBean对象。

那现在我们就来看看Spring容器中到底都有哪些bean。我们所要做的事情就是，运行IOCTest类中的testImport()方法，此时，输出的结果信息如下所示。

![image-20220520162503339](Spring注解.assets/image-20220520162503339.png)

可以看到，结果信息中输出了一个colorFactoryBean，我们看下这个colorFactoryBean到底是个什么鬼！此时，我们对IOCTest类中的testImport()方法稍加改动，添加获取colorFactoryBean的代码，并输出colorFactoryBean实例的类型，如下所示。

```java
@Test
public void testImport02() {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig01.class);
    String[] definitionNames = applicationContext.getBeanDefinitionNames();
    for (String name : definitionNames) {
        System.out.println(name);
    }

    Object bean = applicationContext.getBean("colorFactroyBean");
    System.out.println("bean的类型：" + bean.getClass());

}
```

![image-20220520162642735](Spring注解.assets/image-20220520162642735.png)

可以看到，虽然我在代码中使用@Bean注解注入的是ColorFactoryBean对象，但是实际上从Spring容器中获取到的bean对象却是调用ColorFactoryBean类中的getObject()方法获取到的Color对象。

在ColorFactoryBean类中，我们将Color对象设置为单实例bean，即让isSingleton()方法返回true。接下来，我们在IOCTest类中的testImport()方法里面多次获取Color对象，并判断一下多次获取的对象是否为同一对象，如下所示。

```java
@Test
    public void testImport02() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig01.class);
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }

        Object bean = applicationContext.getBean("colorFactroyBean");
        Object bean1 = applicationContext.getBean("colorFactroyBean");
//        System.out.println("bean的类型：" + bean.getClass());
        System.out.println(bean == bean1);
    }
```

![image-20220520162955719](Spring注解.assets/image-20220520162955719.png)

可以看到，在ColorFactoryBean类中的isSingleton()方法里面返回true时，每次获取到的Color对象都是同一个对象，说明Color对象是单实例bean。

如果，在ColorFactoryBean类中的isSingleton()方法里面返回false，这样就会将Color对象设置为多实例bean。

##### 3. 如何在Spring容器中获取到FactoryBean对象本身呢？

`其实，这也很简单，只需要在获取工厂Bean本身时，在id前面加上&符号即可，例如&colorFactoryBean。`

```java
@Test
    public void testImport02() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig01.class);
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }

        Object bean = applicationContext.getBean("colorFactroyBean");
        Object bean1 = applicationContext.getBean("&colorFactroyBean");
        System.out.println("bean1的类型：" + bean1.getClass());
        System.out.println("bean的类型：" + bean.getClass());
//        System.out.println(bean == bean1);
    }
```

![image-20220520163425444](Spring注解.assets/image-20220520163425444.png)

可以看到，在获取bean时，在id前面加上&符号就会获取到ColorFactoryBean实例对象。

那问题又来了！！`为什么在id前面加上&符号就会获取到ColorFactoryBean实例对象呢？`

接下来，我们就要揭开这个神秘的面纱了，打开BeanFactory接口，查看其源码。

```java
public interface BeanFactory {

   /**
    * Used to dereference a {@link FactoryBean} instance and distinguish it from
    * beans <i>created</i> by the FactoryBean. For example, if the bean named
    * {@code myJndiObject} is a FactoryBean, getting {@code &myJndiObject}
    * will return the factory, not the instance returned by the factory.
    */
   String FACTORY_BEAN_PREFIX = "&";
   ...
   }
```

在BeanFactory接口中定义了一个&前缀，只要我们使用bean的id来从Spring容器中获取bean时，Spring就会知道我们是在获取FactoryBean本身。

#### 十一、如何使用@Bean注解指定初始化和销毁的方法？

##### 1. bean的生命周期

通常意义上讲的bean的生命周期，指的是bean从创建到初始化，经过一系列的流程，最终销毁的过程。只不过，在Spring中，bean的生命周期是由Spring容器来管理的。在Spring中，我们可以自己来指定bean的初始化和销毁的方法。我们指定了bean的初始化和销毁方法之后，当容器在bean进行到当前生命周期的阶段时，会自动调用我们自定义的初始化和销毁方法。

##### 2. 如何定义初始化和销毁方法？

我们已经知道了由Spring管理bean的生命周期时，我们可以指定bean的初始化和销毁方法，那具体该如何定义这些初始化和销毁方法呢？接下来，我们就介绍第一种定义初始化和销毁方法的方式：通过@Bean注解指定初始化和销毁方法。

如果是使用XML配置文件的方式配置bean的话，那么可以在标签中指定bean的初始化和销毁方法，如下所示。

```xml
<bean id="person" class="com.meimeixia.bean.Person" init-method="init" destroy-method="destroy">
    <property name="age" value="18"></property>
    <property name="name" value="liayun"></property>
</bean>

```

这里，需要注意的是，在我们自己写的Person类中，需要存在init()方法和destroy()方法。而且Spring中还规定，这里的init()方法和destroy()方法必须是无参方法，但可以抛出异常。

如果我们使用注解的方式，那么该如何实现指定bean的初始化和销毁方法呢？别急，我们下面就一起来搞定它！！

首先，创建一个名称为Car的类，这个类的实现比较简单，如下所示。

```java
public class Car {
    public Car() {
        System.out.println("car constructor...");
    }

    public void init() {
        System.out.println("car ... init...");
    }

    public void destroy() {
        System.out.println("car ... destroy...");
    }
}
```

然后，我们将Car类对象通过注解的方式注册到Spring容器中，具体的做法就是新建一个MainConfigOfLifeCycle类作为Spring的配置类，将Car类对象通过MainConfigOfLifeCycle类注册到Spring容器中，MainConfigOfLifeCycle类的代码如下所示。

```java
@Configuration
public class MainConfigOfLifeCycle {

    @Bean
    public Car car() {
        return new Car();
    }
}
```

接着，我们就新建一个IOCTest_LifeCycle类来测试容器中的Car对象，该测试类的代码如下所示。

```java
@Test
    public void test01() {
        // 1. 创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
    }
```

![image-20220520170304106](Spring注解.assets/image-20220520170304106.png)

可以看到，在Spring容器创建完成时，会自动调用单实例bean的构造方法，对单实例bean进行了实例化操作。

`总之，对于单实例bean来说，会在Spring容器启动的时候创建对象；对于多实例bean来说，会在每次获取bean的时候创建对象。`

现在，我们在Car类中指定了init()方法和destroy()方法，那么，如何让Spring容器知道Car类中的init()方法是用来执行对象的初始化操作，而destroy()方法是用来执行对象的销毁操作呢？如果是使用XML文件配置的话，那么我们可以使用如下配置来实现。

```xml
<bean id="car" class="com.meimeixia.bean.Car" init-method="init" destroy-method="destroy">

</bean>

```

如果我们使用的是@Bean注解，那么又该如何实现呢？其实很简单，我们来看下@Bean注解的源码，如下所示。

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {

	@AliasFor("name")
	String[] value() default {};

	@AliasFor("value")
	String[] name() default {};

	@Deprecated
	Autowire autowire() default Autowire.NO;

	boolean autowireCandidate() default true;

	String initMethod() default "";

	String destroyMethod() default AbstractBeanDefinition.INFER_METHOD;

}
```

看到@Bean注解的源码，相信小伙伴们会有种豁然开朗的感觉，没错，就是使用@Bean注解的initMethod属性和destroyMethod属性来指定bean的初始化方法和销毁方法。

所以，我们得在MainConfigOfLifeCycle配置类的@Bean注解中指定initMethod属性和destroyMethod属性，如下所示。

```java
@Configuration
public class MainConfigOfLifeCycle {

    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}
```

此时，我们再来运行IOCTest_LifeCycle类中的test01()方法，会发现输出的结果信息如下所示。

![image-20220520170825007](Spring注解.assets/image-20220520170825007.png)

从输出结果中可以看出，在Spring容器中，先是调用了`Car类的构造方法来创建Car对象`，接下来便是调用了`Car对象的init()方法`来进行初始化。

**有些小伙伴们可能会问了，运行上面的测试方法并没有打印出bean的销毁方法中的信息啊，那什么时候执行bean的销毁方法呢？这个问题问的很好，bean的销毁方法是在容器关闭的时候被调用的。**

所以，我们在IOCTest_LifeCycle类中的test01()方法中，添加关闭容器的代码，如下所示。

```java
@Test
    public void test01() {
        // 1. 创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");

        //关闭容器
        applicationContext.close();
    }
```

![image-20220520170951389](Spring注解.assets/image-20220520170951389.png)

##### 3. 指定初始化和销毁方法的使用场景

一个典型的使用场景就是对于数据源的管理。例如，在配置数据源时，在初始化的时候，会对很多的数据源的属性进行赋值操作；在销毁的时候，我们需要对数据源的连接等信息进行关闭和清理。这个时候，我们就可以在自定义的初始化和销毁方法中来做这些事情了！

##### 4. 初始化和销毁方法调用的时机

- bean对象的初始化方法调用的时机：对象创建完成，如果对象中存在一些属性，并且这些属性也都赋好值之后，那么就会调用bean的初始化方法。对于单实例bean来说，在Spring容器创建完成后，Spring容器会自动调用bean的初始化方法；对于多实例bean来说，在每次获取bean对象的时候，调用bean的初始化方法。
- bean对象的销毁方法调用的时机：对于单实例bean来说，在容器关闭的时候，会调用bean的销毁方法；`对于多实例bean来说，Spring容器不会管理这个bean，也就不会自动调用这个bean的销毁方法了。不过，小伙伴们可以手动调用多实例bean的销毁方法。`

我们已经说了单实例bean的初始化和销毁方法。接下来，我们来说下多实例bean的初始化和销毁方法。我们将Car对象变成多实例bean来验证下。

首先，我们在MainConfigOfLifeCycle配置类中的car()方法上通过@Scope注解将Car对象设置成多实例bean，如下所示。

```java
@Configuration
public class MainConfigOfLifeCycle {

    @Scope("prototype")
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}
```

然后，我们修改一下IOCTest_LifeCycle类中的test01()方法，将关闭容器的那行代码给注释掉，如下所示。

```java
 @Test
    public void test01() {
        // 1. 创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");

        //关闭容器
//        applicationContext.close();
    }
```

![image-20220520171701520](Spring注解.assets/image-20220520171701520.png)

可以看到，当我们将Car对象设置成多实例bean，并且没有获取bean实例对象时，Spring容器并没有执行bean的构造方法、初始化方法和销毁方法。

说到这，我们就在IOCTest_LifeCycle类中的test01()方法里面添加一行获取Car对象的代码，如下所示。

```java
 @Test
    public void test01() {
        // 1. 创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");

        applicationContext.getBean("car"); // 多实例bean在获取的时候才创建对象
        //关闭容器
//        applicationContext.close();
    }
```

![image-20220520171825102](Spring注解.assets/image-20220520171825102.png)

当我们在获取多实例bean对象的时候，会创建对象并进行初始化，但是销毁方法是在什么时候被调用呢？是在容器关闭的时候吗？我们可以将IOCTest_LifeCycle类中的test01()方法里面的那行关闭容器的代码放开来进行验证，就像下面这样。

```java
 @Test
    public void test01() {
        // 1. 创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");

        applicationContext.getBean("car"); // 多实例bean在获取的时候才创建对象
        //关闭容器
        applicationContext.close();
    }
```



![image-20220520171903733](Spring注解.assets/image-20220520171903733.png)

`可以看到，多实例的bean在容器关闭的时候是不进行销毁的，也就是说你每次获取时，IOC容器帮你创建出对象交还给你，至于要什么时候销毁这是你自己的事，Spring容器压根就不会再管理这些多实例的bean了。`

#### 十二、使用InitializingBean和DisposableBean来管理bean的生命周期

##### 1. InitializingBean接口

Spring中提供了一个InitializingBean接口，该接口为bean提供了属性初始化后的处理方法，它只包括afterPropertiesSet方法，凡是继承该接口的类，在bean的属性初始化后都会执行该方法。InitializingBean接口的源码如下所示。

```java
public interface InitializingBean {

	/**
	 * Invoked by the containing {@code BeanFactory} after it has set all bean properties
	 * and satisfied {@link BeanFactoryAware}, {@code ApplicationContextAware} etc.
	 * <p>This method allows the bean instance to perform validation of its overall
	 * configuration and final initialization when all bean properties have been set.
	 * @throws Exception in the event of misconfiguration (such as failure to set an
	 * essential property) or if initialization fails for any other reason
	 */
	void afterPropertiesSet() throws Exception;

}
```

根据InitializingBean接口中提供的afterPropertiesSet()方法的名字不难推断出，afterPropertiesSet()方法是在属性赋好值之后调用的。

##### 2. 何时调用InitializingBean接口？

```java
protected void invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd)
			throws Throwable {

		boolean isInitializingBean = (bean instanceof InitializingBean);
		if (isInitializingBean && (mbd == null || !mbd.hasAnyExternallyManagedInitMethod("afterPropertiesSet"))) {
			if (logger.isTraceEnabled()) {
				logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
			}
			if (System.getSecurityManager() != null) {
				try {
					AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
						((InitializingBean) bean).afterPropertiesSet();
						return null;
					}, getAccessControlContext());
				}
				catch (PrivilegedActionException pae) {
					throw pae.getException();
				}
			}
			else {
				((InitializingBean) bean).afterPropertiesSet();
			}
		}

		if (mbd != null && bean.getClass() != NullBean.class) {
			String initMethodName = mbd.getInitMethodName();
			if (StringUtils.hasLength(initMethodName) &&
					!(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
					!mbd.hasAnyExternallyManagedInitMethod(initMethodName)) {
				invokeCustomInitMethod(beanName, bean, mbd);
			}
		}
	}
```

分析上述代码后，我们可以初步得出如下信息：

- Spring为bean提供了两种初始化的方式，实现InitializingBean接口（也就是要实现该接口中的afterPropertiesSet方法），或者在配置文件或@Bean注解中通过init-method来指定，两种方式可以同时使用。
- 实现InitializingBean接口是直接调用afterPropertiesSet()方法，与通过反射调用init-method指定的方法相比，效率相对来说要高点。但是init-method方式消除了对Spring的依赖。
- 如果调用afterPropertiesSet方法时出错，那么就不会调用init-method指定的方法了。

也就是说Spring为bean提供了两种初始化的方式，第一种方式是实现InitializingBean接口（也就是要实现该接口中的afterPropertiesSet方法），第二种方式是在配置文件或@Bean注解中通过init-method来指定，这两种方式可以同时使用，同时使用先调用afterPropertiesSet方法，后执行init-method指定的方法。

##### 3. DisposableBean接口

实现org.springframework.beans.factory.DisposableBean接口的bean在销毁前，Spring将会调用DisposableBean接口的destroy()方法。也就是说我们可以实现DisposableBean这个接口来定义咱们这个销毁的逻辑。

源码如下：

```java
public interface DisposableBean {

	/**
	 * Invoked by the containing {@code BeanFactory} on destruction of a bean.
	 * @throws Exception in case of shutdown errors. Exceptions will get logged
	 * but not rethrown to allow other beans to release their resources as well.
	 */
	void destroy() throws Exception;

}
```

可以看到，在DisposableBean接口中只定义了一个destroy()方法。

在bean生命周期结束前调用destroy()方法做一些收尾工作，亦可以使用destroy-method。前者与Spring耦合高，使用类型强转.方法名()，效率高；后者耦合低，使用反射，效率相对来说较低。

##### 4. Disposable接口注意事项

多实例bean的生命周期不归Spring容器来管理，这里的DisposableBean接口中的方法是由Spring容器来调用的，所以如果一个多实例bean实现了DisposableBean接口是没有啥意义的，因为相应的方法根本不会被调用，当然了，在XML配置文件中指定了destroy方法，也是没有任何意义的。所以，在多实例bean情况下，Spring是不会自动调用bean的销毁方法的。

##### 5. 单实例bean案例

首先，创建一个Cat的类来实现InitializingBean和DisposableBean这俩接口，代码如下所示，注意该Cat类上标注了一个@Component注解。

```java
@Component
public class Cat implements InitializingBean, DisposableBean {
    public Cat() {
        System.out.println("cat constructor...");
    }

    /**
     * 会在容器关闭的时候进行调用
     */
    @Override
    public void destroy() throws Exception {
        // TODO Auto-generated method stub
        System.out.println("cat destroy...");
    }

    /**
     * 会在bean创建完成，并且属性都赋好值以后进行调用
     */
    @Override
    public void afterPropertiesSet() throws Exception {
        // TODO Auto-generated method stub
        System.out.println("cat afterPropertiesSet...");
    }
}
```

然后，在MainConfigOfLifeCycle配置类中通过包扫描的方式将以上类注入到Spring容器中。

```java
@Configuration
@ComponentScan("com.lbgao.pojo")
public class MainConfigOfLifeCycle {

    @Scope("prototype")
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}
```

接着，运行IOCTest_LifeCycle类中的test01()方法，输出的结果信息如下所示。

![image-20220520174230839](Spring注解.assets/image-20220520174230839.png)

从输出的结果信息中可以看出，单实例bean情况下，IOC容器创建完成后，会自动调用bean的初始化方法；而在容器销毁前，会自动调用bean的销毁方法。

在多实例bean情况下，Spring不会自动调用bean的销毁方法。

#### 十三、@PostConstruct注解和@PreDestroy注解

##### @PostConstruct注解

@PostConstruct注解好多人以为是Spring提供的，其实它是Java自己的注解，是JSR-250规范里面定义的一个注解。我们来看下@PostConstruct注解的源码，如下所示。

```java
package javax.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface PostConstruct {
}

```

从源码可以看出，@PostConstruct注解是Java中的注解，并不是Spring提供的注解。

@PostConstruct注解被用来修饰一个非静态的void()方法。被@PostConstruct注解修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器执行一次。被@PostConstruct注解修饰的方法通常在构造函数之后，init()方法之前执行。

通常我们是会在Spring框架中使用到@PostConstruct注解的，该注解的方法在整个bean初始化中的执行顺序如下：

> Constructor（构造方法）→@Autowired（依赖注入）→@PostConstruct（注释的方法）

##### @PreDestroy注解

@PreDestroy注解同样是Java提供的，它也是JSR-250规范里面定义的一个注解。看下它的源码，如下所示。

```java
package javax.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface PreDestroy {
}
```

被@PreDestroy注解修饰的方法会在服务器卸载Servlet的时候运行，并且只会被服务器调用一次，类似于Servlet的destroy()方法。被@PreDestroy注解修饰的方法会在destroy()方法之后，Servlet被彻底卸载之前执行。执行顺序如下所示：

> 调用destroy()方法→@PreDestroy→destroy()方法→bean销毁

##### 一个案例

首先，我们创建一个Dog类，如下所示，注意在该类上标注了一个@Component注解。

```java
package com.lbgao.pojo;

import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/**
 * @Auther: lbgao
 * @Date: 2022/5/21 16:15
 */

@Component
public class Dog {

    public Dog() {
        System.out.println("dog constructor...");
    }

    // 在对象创建完成并且属性赋值完成之后调用
    @PostConstruct
    public void init() {
        System.out.println("dog...@PostConstruct...");
    }

    // 在容器销毁（移除）对象之前调用
    @PreDestroy
    public void destory() {
        System.out.println("dog...@PreDestroy...");
    }

}

```

可以看到，在以上Dog类中，我们提供了构造方法、init()方法以及destroy()方法，并且还使用了@PostConstruct注解和@PreDestroy注解来分别标注init()方法和destroy()方法。

然后，在MainConfigOfLifeCycle配置类中通过包扫描的方式将以上类注入到Spring容器中。

接着，运行IOCTest_LifeCycle类中的test01()方法，输出的结果信息如下所示。

![image-20220521161903187](Spring注解.assets/image-20220521161903187.png)

从输出的结果信息中可以看出，被@PostConstruct注解修饰的方法是在bean创建完成并且属性赋值完成之后才执行的，而被@PreDestroy注解修饰的方法是在容器销毁bean之前执行的，通常是进行一些清理工作。

#### 十四、关于BeanPostProcessor后置处理器，你了解多少？

##### BeanPostProcessor后置处理器概述

```java
package org.springframework.beans.factory.config;

import org.springframework.beans.BeansException;
import org.springframework.lang.Nullable;

public interface BeanPostProcessor {

	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

}

```

从源码可以看出，`BeanPostProcessor`是一个接口，其中有两个方法，即`postProcessBeforeInitialization`和`postProcessAfterInitialization`这两个方法，这两个方法分别是在Spring容器中的bean初始化前后执行，所以Spring容器中的每一个bean对象初始化前后，都会执行`BeanPostProcessor`接口的实现类中的这两个方法。

也就是说，`postProcessBeforeInitialization`方法会在bean实例化和属性设置之后，自定义初始化方法之前被调用，而`postProcessAfterInitialization`方法会在自定义初始化方法之后被调用。当容器中存在多个`BeanPostProcessor`的实现类时，会按照它们在容器中注册的顺序执行。对于自定义的`BeanPostProcessor`实现类，还可以让其实现Ordered接口自定义排序。

因此我们可以在每个bean对象初始化前后，加上自己的逻辑。实现方式是自定义一个BeanPostProcessor接口的实现类，例如MyBeanPostProcessor，然后在该类的postProcessBeforeInitialization和postProcessAfterInitialization这俩方法中写上自己的逻辑。

##### BeanPostProcessor后置处理器实例

我们创建一个MyBeanPostProcessor类，实现BeanPostProcessor接口，如下所示。

```java
@Component // 将后置处理器加入到容器中，这样的话，Spring就能让它工作了
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization..." + beanName + "=>" + bean);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization..." + beanName + "=>" + bean);
        return bean;
    }
}
```

接下来，我们应该是要编写测试用例来进行测试了。不过，在这之前，咱们得做几处改动，第一处改动是将MainConfigOfLifeCycle配置类中的car()方法上的`@Scope("prototype")`注解给注释掉，因为咱们之前做测试的时候，是将Car对象设置成多实例bean了。

```java
@Configuration
@ComponentScan("com.lbgao")
public class MainConfigOfLifeCycle {

//    @Scope("prototype")
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}
```

直接运行IOCTest_LifeCycle类中的test01()方法就行，该方法的代码如下所示。

```java
@Test
    public void test01() {
        // 1. 创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");

//        applicationContext.getBean("car"); // 多实例bean在获取的时候才创建对象
        //关闭容器
        applicationContext.close();
    }
```

![image-20220521163353664](Spring注解.assets/image-20220521163353664.png)

可以看到，postProcessBeforeInitialization方法会在bean实例化和属性设置之后，自定义初始化方法之前被调用，而postProcessAfterInitialization方法会在自定义初始化方法之后被调用。

##### BeanPostProcessor后置处理器作用

后置处理器可用于bean对象初始化前后进行逻辑增强。Spring提供了BeanPostProcessor接口的很多实现类，例如AutowiredAnnotationBeanPostProcessor用于@Autowired注解的实现，AnnotationAwareAspectJAutoProxyCreator用于Spring AOP的动态代理等等。

除此之外，我们还可以自定义BeanPostProcessor接口的实现类，在其中写入咱们需要的逻辑。下面我会以AnnotationAwareAspectJAutoProxyCreator为例，简单说明一下后置处理器是怎样工作的。

我们都知道spring AOP的实现原理是动态代理，最终放入容器的是`代理类的对象`，而不是bean本身的对象，那么Spring是什么时候做到这一步的呢？

就是在`AnnotationAwareAspectJAutoProxyCreator`后置处理器的`postProcessAfterInitialization`方法中，即bean对象初始化完成之后，后置处理器会判断该bean是否注册了切面，若是，则生成代理对象注入到容器中。

这一部分的关键代码是在哪儿呢？我们定位到`AbstractAutoProxyCreator`抽象类中的`postProcessAfterInitialization`方法处便能看到了，如下所示。

![image-20220521164110841](Spring注解.assets/image-20220521164110841.png)

#### ==十五、BeanPostProcessor的执行流程==

##### 写在前面

在前面的文章中，我们讲述了`BeanPostProcessor`的`postProcessBeforeInitialization()方法`和`postProcessAfterInitialization()`方法在bean初始化的前后调用。而且我们可以自定义类来实现BeanPostProcessor接口，并在postProcessBeforeInitialization()方法和postProcessAfterInitialization()方法中编写我们自定义的逻辑。

今天，我们来一起探讨下BeanPostProcessor的底层原理。

##### bean的初始化和销毁

我们知道BeanPostProcessor的postProcessBeforeInitialization()方法是在bean的初始化之前被调用；而postProcessAfterInitialization()方法是在bean初始化的之后被调用。并且bean的初始化和销毁方法我们可以通过如下方式进行指定。

###### （一）通过@Bean指定init-method和destroy-method

```java
@Bean(initMethod="init", destroyMethod="destroy")
public Car car() {
    return new Car();
}
```

###### （二）通过让bean实现InitializingBean和DisposableBean这俩接口

```java
@Component
public class Cat implements InitializingBean, DisposableBean {
    public Cat() {
        System.out.println("cat constructor...");
    }

    /**
     * 会在容器关闭的时候进行调用
     */
    @Override
    public void destroy() throws Exception {
        // TODO Auto-generated method stub
        System.out.println("cat destroy...");
    }

    /**
     * 会在bean创建完成，并且属性都赋好值以后进行调用
     */
    @Override
    public void afterPropertiesSet() throws Exception {
        // TODO Auto-generated method stub
        System.out.println("cat afterPropertiesSet...");
    }
}
```

###### 使用JSR-250规范里面定义的@PostConstruct和@PreDestroy这俩注解

- @PostConstruct：在bean创建完成并且属性赋值完成之后，来执行初始化方法
- @PreDestroy：在容器销毁bean之前通知我们进行清理工作

```java
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/**
 * @Auther: lbgao
 * @Date: 2022/5/21 16:15
 */

@Component
public class Dog {

    public Dog() {
        System.out.println("dog constructor...");
    }

    // 在对象创建完成并且属性赋值完成之后调用
    @PostConstruct
    public void init() {
        System.out.println("dog...@PostConstruct...");
    }

    // 在容器销毁（移除）对象之前调用
    @PreDestroy
    public void destory() {
        System.out.println("dog...@PreDestroy...");
    }


```

###### （四）通过让bean实现BeanPostProcessor接口

```java
@Component // 将后置处理器加入到容器中，这样的话，Spring就能让它工作了
public class MyBeanPostProcessor implements BeanPostProcessor, Ordered {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization..." + beanName + "=>" + bean);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization..." + beanName + "=>" + bean);
        return bean;
    }

    @Override
    public int getOrder() {
        return 3;
    }
}
```

通过以上这四种方式，我们就可以对bean的整个生命周期进行控制：

- bean的实例化：调用bean的构造方法，我们可以在bean的无参构造方法中执行相应的逻辑。
- bean的初始化：在初始化时，可以通过`BeanPostProcessor`的`postProcessBeforeInitialization()`方法和`postProcessAfterInitialization()`方法进行拦截，执行自定义的逻辑；通过`@PostConstruct`注解、`InitializingBean`和`init-method`来指定bean初始化前后执行的方法，在该方法中咱们可以执行自定义的逻辑。
- bean的销毁：可以通过@PreDestroy注解、DisposableBean和destroy-method来指定bean在销毁前执行的方法，在该方法中咱们可以执行自定义的逻辑。

所以，通过上述四种方式，我们可以控制Spring中bean的整个生命周期。

##### BeanPostProcessor源码分析

如果想深刻理解BeanPostProcessor的工作原理，那么就不得不看下相关的源码，我们可以在MyBeanPostProcessor类的`postProcessBeforeInitialization()`方法和`postProcessAfterInitialization()`方法这两处打上断点来进行调试，如下所示。

```java
@Component // 将后置处理器加入到容器中，这样的话，Spring就能让它工作了
public class MyBeanPostProcessor implements BeanPostProcessor, Ordered {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization..." + beanName + "=>" + bean);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization..." + beanName + "=>" + bean);
        return bean;
    }

    @Override
    public int getOrder() {
        return 3;
    }
}
```

###### 第一步 在IOCTest_LifeCycle类的test01()方法中，首先通过new实例对象的方式创建了一个IOC容器。

![image-20220521172227255](Spring注解.assets/image-20220521172227255.png)

```java
public class IOCTest_LifeCycle {

    @Test
    public void test01() {
        // 1. 创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");

//        applicationContext.getBean("car"); // 多实例bean在获取的时候才创建对象
        //关闭容器
        applicationContext.close();
    }
}
```

###### 第二步，通过调用栈继续分析，单击IOCTest_LifeCycle类的test01()方法上面的那个方法，这时会进入AnnotationConfigApplicationContext类的构造方法中。

![image-20220521172326986](Spring注解.assets/image-20220521172326986.png)

![image-20220521172351694](Spring注解.assets/image-20220521172351694.png)

可以看到，在AnnotationConfigApplicationContext类的构造方法中会调用refresh()方法。

###### 第三步，我们继续跟进方法调用栈，如下所示，可以看到，方法的执行定位到AbstractApplicationContext类的refresh()方法中的如下那行代码处。

![image-20220521172521877](Spring注解.assets/image-20220521172521877.png)

![image-20220521172623916](Spring注解.assets/image-20220521172623916.png)

上面这行代码的作用就是初始化所有的（非懒加载的）单实例bean对象。

AbstractApplicationContext类中的refresh()方法有点长，下面是源码，可以清楚地看到refresh()方法里面调用了finishBeanFactoryInitialization()方法。

```java
@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);
				beanPostProcess.end();

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
				contextRefresh.end();
			}
		}
	}
```

###### 第四步，我们继续跟进方法调用栈，如下所示，可以看到，方法的执行定位到AbstractApplicationContext类的finishBeanFactoryInitialization()方法中的如下那行代码处。

![image-20220521172901036](Spring注解.assets/image-20220521172901036.png)

![image-20220521172940036](Spring注解.assets/image-20220521172940036.png)

这行代码的作用同样是初始化所有的（非懒加载的）单实例bean。

AbstractApplicationContext类的finishBeanFactoryInitialization()方法同样有点长，源码如下所示。

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
		// Initialize conversion service for this context.
		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
				beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
			beanFactory.setConversionService(
					beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
		}

		// Register a default embedded value resolver if no BeanFactoryPostProcessor
		// (such as a PropertySourcesPlaceholderConfigurer bean) registered any before:
		// at this point, primarily for resolution in annotation attribute values.
		if (!beanFactory.hasEmbeddedValueResolver()) {
			beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
		}

		// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
		String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
		for (String weaverAwareName : weaverAwareNames) {
			getBean(weaverAwareName);
		}

		// Stop using the temporary ClassLoader for type matching.
		beanFactory.setTempClassLoader(null);

		// Allow for caching all bean definition metadata, not expecting further changes.
		beanFactory.freezeConfiguration();

		// Instantiate all remaining (non-lazy-init) singletons.
		beanFactory.preInstantiateSingletons();
	}
```

###### 第五步，我们继续跟进方法调用栈，如下所示，可以看到，方法的执行定位到DefaultListableBeanFactory类的preInstantiateSingletons()方法的最后一个else分支调用的getBean()方法上。

![image-20220521173226696](Spring注解.assets/image-20220521173226696.png)

![image-20220521173245190](Spring注解.assets/image-20220521173245190.png)

DefaultListableBeanFactory类的preInstantiateSingletons()方法同样是有点长，源码如下所示。

```java
@Override
	public void preInstantiateSingletons() throws BeansException {
		if (logger.isTraceEnabled()) {
			logger.trace("Pre-instantiating singletons in " + this);
		}

		// Iterate over a copy to allow for init methods which in turn register new bean definitions.
		// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

		// Trigger initialization of all non-lazy singleton beans...
		for (String beanName : beanNames) {
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
				if (isFactoryBean(beanName)) {
					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
					if (bean instanceof FactoryBean) {
						FactoryBean<?> factory = (FactoryBean<?>) bean;
						boolean isEagerInit;
						if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
							isEagerInit = AccessController.doPrivileged(
									(PrivilegedAction<Boolean>) ((SmartFactoryBean<?>) factory)::isEagerInit,
									getAccessControlContext());
						}
						else {
							isEagerInit = (factory instanceof SmartFactoryBean &&
									((SmartFactoryBean<?>) factory).isEagerInit());
						}
						if (isEagerInit) {
							getBean(beanName);
						}
					}
				}
				else {
					getBean(beanName);
				}
			}
		}
```

###### 第六步，继续跟进方法调用栈，如下所示。

![image-20220521173410319](Spring注解.assets/image-20220521173410319.png)

![image-20220521173422677](Spring注解.assets/image-20220521173422677.png)

此时方法定位到AbstractBeanFactory类的getBean()方法中了，在getBean()方法中，又调用了doGetBean()方法。

###### 第七步，继续跟进方法调用栈，如下所示，此时，方法的执行定位到AbstractBeanFactory类的doGetBean()方法中的如下那行代码处。

![image-20220521173511898](Spring注解.assets/image-20220521173511898.png)

![image-20220521173546681](Spring注解.assets/image-20220521173546681.png)

可以看到，在Spring内部是通过getSingleton()方法来获取单实例bean的。

###### 第八步，继续跟进方法调用栈，如下所示，此时，方法定位到DefaultSingletonBeanRegistry类的getSingleton()方法中的如下那行代码处。

![image-20220521173653776](Spring注解.assets/image-20220521173653776.png)

![image-20220521173741411](Spring注解.assets/image-20220521173741411.png)

可以看到，在getSingleton()方法里面又调用了getObject()方法来获取单实例bean。

###### 第九步，继续跟进方法调用栈，如下所示，此时，方法定位到AbstractBeanFactory类的doGetBean()方法中的如下那行代码处。

![image-20220521173945216](Spring注解.assets/image-20220521173945216.png)

![image-20220521174121447](Spring注解.assets/image-20220521174121447.png)

也就是说，当第一次获取单实例bean时，由于单实例bean还未创建，那么Spring会调用createBean()方法来创建单实例bean。

###### 第十步，继续跟进方法调用栈，如下所示，可以看到，方法的执行定位到AbstractAutowireCapableBeanFactory类的createBean()方法中的如下那行代码处。

![image-20220521174317640](Spring注解.assets/image-20220521174317640.png)

![image-20220521174303977](Spring注解.assets/image-20220521174303977.png)

```java
@Override
	protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {

		if (logger.isTraceEnabled()) {
			logger.trace("Creating instance of bean '" + beanName + "'");
		}
		RootBeanDefinition mbdToUse = mbd;

		// Make sure bean class is actually resolved at this point, and
		// clone the bean definition in case of a dynamically resolved Class
		// which cannot be stored in the shared merged bean definition.
		Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
		if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
			mbdToUse = new RootBeanDefinition(mbd);
			mbdToUse.setBeanClass(resolvedClass);
		}

		// Prepare method overrides.
		try {
			mbdToUse.prepareMethodOverrides();
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
					beanName, "Validation of method overrides failed", ex);
		}

		try {
			// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
			if (bean != null) {
				return bean;
			}
		}
		catch (Throwable ex) {
			throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
					"BeanPostProcessor before instantiation of bean failed", ex);
		}

		try {
			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
			if (logger.isTraceEnabled()) {
				logger.trace("Finished creating instance of bean '" + beanName + "'");
			}
			return beanInstance;
		}
		catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
			// A previously detected exception with proper bean creation context already,
			// or illegal singleton state to be communicated up to DefaultSingletonBeanRegistry.
			throw ex;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
		}
	}
```

可以看到，Spring中创建单实例bean调用的是doCreateBean()方法。

###### 第十一步，继续跟进方法调用栈，如下所示，此时，方法的执行已经定位到AbstractAutowireCapableBeanFactory类的doCreateBean()方法中的如下那行代码处了。

![image-20220521174450313](Spring注解.assets/image-20220521174450313.png)

![image-20220521174502607](Spring注解.assets/image-20220521174502607.png)

在initializeBean()方法里面会调用一系列的后置处理器。

###### 第十二步，继续跟进方法调用栈，如下所示，此时，方法的执行定位到AbstractAutowireCapableBeanFactory类的initializeBean()方法中的如下那行代码处。

![image-20220521174546762](Spring注解.assets/image-20220521174546762.png)

![image-20220521174611233](Spring注解.assets/image-20220521174611233.png)

小伙伴们需要重点留意一下这个applyBeanPostProcessorsBeforeInitialization()方法。

回过头来我们再来看看AbstractAutowireCapableBeanFactory类的doCreateBean()方法中的如下这行代码。

![image-20220521174907654](Spring注解.assets/image-20220521174907654.png)

没错，在以上initializeBean()方法中调用了后置处理器的逻辑，这我上面已经说到了。小伙伴们需要特别注意一下，在AbstractAutowireCapableBeanFactory类的doCreateBean()方法中，调用initializeBean()方法之前，还调用了一个`populateBean()`方法，我也在上图中标注出来了。

我们点进去这个populateBean()方法中，看下这个方法到底执行了哪些逻辑，如下所示。

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
		if (bw == null) {
			if (mbd.hasPropertyValues()) {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
			}
			else {
				// Skip property population phase for null instance.
				return;
			}
		}

		// Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
		// state of the bean before properties are set. This can be used, for example,
		// to support styles of field injection.
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
				if (!bp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
					return;
				}
			}
		}

		PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

		int resolvedAutowireMode = mbd.getResolvedAutowireMode();
		if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
			MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
			// Add property values based on autowire by name if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
				autowireByName(beanName, mbd, bw, newPvs);
			}
			// Add property values based on autowire by type if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
				autowireByType(beanName, mbd, bw, newPvs);
			}
			pvs = newPvs;
		}

		boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
		boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

		PropertyDescriptor[] filteredPds = null;
		if (hasInstAwareBpps) {
			if (pvs == null) {
				pvs = mbd.getPropertyValues();
			}
			for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
				PropertyValues pvsToUse = bp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
				if (pvsToUse == null) {
					if (filteredPds == null) {
						filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
					}
					pvsToUse = bp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
					if (pvsToUse == null) {
						return;
					}
				}
				pvs = pvsToUse;
			}
		}
		if (needsDepCheck) {
			if (filteredPds == null) {
				filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
			}
			checkDependencies(beanName, mbd, filteredPds, pvs);
		}

		if (pvs != null) {
			applyPropertyValues(beanName, mbd, bw, pvs);
		}
	}
```

populateBean()方法同样是AbstractAutowireCapableBeanFactory类中的方法，它里面的代码比较多，但是逻辑非常简单，populateBean()方法做的工作就是为bean的属性赋值。也就是说，在Spring中会先调用populateBean()方法为bean的属性赋好值，然后再调用initializeBean()方法。

接下来，我们好好分析下initializeBean()方法，为了方便，我将Spring中AbstractAutowireCapableBeanFactory类的initializeBean()方法的代码特意提取出来了，如下所示。

```java
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

		try {
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
	}
```

在initializeBean()方法中，调用了invokeInitMethods()方法，代码行如下所示。

```java
invokeInitMethods(beanName, wrappedBean, mbd);
```

invokeInitMethods()方法的作用就是执行初始化方法，这些初始化方法包括我们之前讲的：`在XML配置文件的标签中使用init-method属性指定的初始化方法；在@Bean注解中使用initMehod属性指定的方法；使用@PostConstruct注解标注的方法；实现InitializingBean接口的方法等。`

在调用invokeInitMethods()方法之前，Spring调用了applyBeanPostProcessorsBeforeInitialization()这个方法，代码行如下所示。

```java
if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}
```

这里，我们先来看看`applyBeanPostProcessorsBeforeInitialization()`方法中具体执行了哪些逻辑，该方法位于AbstractAutowireCapableBeanFactory类中，源码如下所示。

![image-20220521175644205](Spring注解.assets/image-20220521175644205.png)

可以看到，在`applyBeanPostProcessorsBeforeInitialization()`方法中，会遍历所有`BeanPostProcessor`对象，然后依次执行所有BeanPostProcessor对象的`postProcessBeforeInitialization()`方法，一旦`BeanPostProcessor`对象的`postProcessBeforeInitialization()`方法返回null以后，则后面的BeanPostProcessor对象便不再执行了，而是直接`退出for循环`。这些都是我们看源码看到的。

看Spring源码，我们还看到了一个细节，**在Spring中调用initializeBean()方法之前，还调用了`populateBean()`方法来为bean的`属性赋值`，** 这在上面我也已经说过了。

==经过上面的一系列的跟踪源码分析，我们可以将关键代码的调用过程使用如下伪代码表述出来。==

```java
populateBean(beanName, mbd, instanceWrapper); // 给bean进行属性赋值
initializeBean(beanName, exposedObject, mbd)
{
	applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
	invokeInitMethods(beanName, wrappedBean, mbd); // 执行自定义初始化
	applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
}
```

也就是说，在Spring中，调用initializeBean()方法之前，调用了populateBean()方法为bean的属性赋值，为bean的属性赋好值之后，再调用initializeBean()方法进行初始化。

在initializeBean()中，调用自定义的初始化方法（即invokeInitMethods()）之前，调用了applyBeanPostProcessorsBeforeInitialization()方法，而在调用自定义的初始化方法之后，又调用了applyBeanPostProcessorsAfterInitialization()方法。至此，整个bean的初始化过程就这样结束了。

#### 十六、BeanPostProcessor在Spring底层是如何使用的？

BeanPostProcessor接口源码如下：

```java
public interface BeanPostProcessor {
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
}
```

可以看到，在BeanPostProcessor接口中，提供了两个方法：postProcessBeforeInitialization()方法和postProcessAfterInitialization()方法。postProcessBeforeInitialization()方法会在bean初始化之前调用，postProcessAfterInitialization()方法会在bean初始化之后调用。接下来，我们就来分析下BeanPostProcessor接口在Spring中的实现。

##### ApplicationContextAwareProcessor类

```java
/*
 * Copyright 2002-2020 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.context.support;

import java.security.AccessControlContext;
import java.security.AccessController;
import java.security.PrivilegedAction;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.factory.config.EmbeddedValueResolver;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.ApplicationEventPublisherAware;
import org.springframework.context.ApplicationStartupAware;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.EmbeddedValueResolverAware;
import org.springframework.context.EnvironmentAware;
import org.springframework.context.MessageSourceAware;
import org.springframework.context.ResourceLoaderAware;
import org.springframework.lang.Nullable;
import org.springframework.util.StringValueResolver;

/**
 * {@link BeanPostProcessor} implementation that supplies the {@code ApplicationContext},
 * {@link org.springframework.core.env.Environment Environment}, or
 * {@link StringValueResolver} for the {@code ApplicationContext} to beans that
 * implement the {@link EnvironmentAware}, {@link EmbeddedValueResolverAware},
 * {@link ResourceLoaderAware}, {@link ApplicationEventPublisherAware},
 * {@link MessageSourceAware}, and/or {@link ApplicationContextAware} interfaces.
 *
 * <p>Implemented interfaces are satisfied in the order in which they are
 * mentioned above.
 *
 * <p>Application contexts will automatically register this with their
 * underlying bean factory. Applications do not use this directly.
 *
 * @author Juergen Hoeller
 * @author Costin Leau
 * @author Chris Beams
 * @since 10.10.2003
 * @see org.springframework.context.EnvironmentAware
 * @see org.springframework.context.EmbeddedValueResolverAware
 * @see org.springframework.context.ResourceLoaderAware
 * @see org.springframework.context.ApplicationEventPublisherAware
 * @see org.springframework.context.MessageSourceAware
 * @see org.springframework.context.ApplicationContextAware
 * @see org.springframework.context.support.AbstractApplicationContext#refresh()
 */
class ApplicationContextAwareProcessor implements BeanPostProcessor {

	private final ConfigurableApplicationContext applicationContext;

	private final StringValueResolver embeddedValueResolver;


	/**
	 * Create a new ApplicationContextAwareProcessor for the given context.
	 */
	public ApplicationContextAwareProcessor(ConfigurableApplicationContext applicationContext) {
		this.applicationContext = applicationContext;
		this.embeddedValueResolver = new EmbeddedValueResolver(applicationContext.getBeanFactory());
	}


	@Override
	@Nullable
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		if (!(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
				bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
				bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware ||
				bean instanceof ApplicationStartupAware)) {
			return bean;
		}

		AccessControlContext acc = null;

		if (System.getSecurityManager() != null) {
			acc = this.applicationContext.getBeanFactory().getAccessControlContext();
		}

		if (acc != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareInterfaces(bean);
				return null;
			}, acc);
		}
		else {
			invokeAwareInterfaces(bean);
		}

		return bean;
	}

	private void invokeAwareInterfaces(Object bean) {
		if (bean instanceof EnvironmentAware) {
			((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
		}
		if (bean instanceof EmbeddedValueResolverAware) {
			((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
		}
		if (bean instanceof ResourceLoaderAware) {
			((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
		}
		if (bean instanceof ApplicationEventPublisherAware) {
			((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
		}
		if (bean instanceof MessageSourceAware) {
			((MessageSourceAware) bean).setMessageSource(this.applicationContext);
		}
		if (bean instanceof ApplicationStartupAware) {
			((ApplicationStartupAware) bean).setApplicationStartup(this.applicationContext.getApplicationStartup());
		}
		if (bean instanceof ApplicationContextAware) {
			((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
		}
	}

}

```

要想使用ApplicationContextAwareProcessor类向组件中注入IOC容器，我们就不得不提Spring中的另一个接口了，即`ApplicationContextAware`。如果需要向组件中注入IOC容器，那么可以让组件实现`ApplicationContextAware`接口。

例如，我们创建一个Dog类，使其实现`ApplicationContextAware`接口，此时，我们需要实现`ApplicationContextAware`接口中的`setApplicationContext()`方法，在`setApplicationContext()`方法中有一个`ApplicationContext`类型的参数，这个就是IOC容器对象，我们可以在Dog类中定义一个`ApplicationContext`类型的成员变量，然后在`setApplicationContext()`方法中为这个成员变量赋值，此时就可以在Dog类中的其他方法中使用`ApplicationContext`对象了，如下所示。

```java
package com.lbgao.bean;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/**
 * @Auther: lbgao
 * @Date: 2022/5/22 20:49
 */

@Component
public class Dog implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Dog() {
        System.out.println("dog constructor...");
    }

    // 在对象创建完成并且属性赋值完成之后调用
    @PostConstruct
    public void init() {
        System.out.println("dog...@PostConstruct...");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("dog...@PreDestroy...");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}

```

看到这里，相信不少小伙伴们都有一种很熟悉的感觉，没错，我之前也在项目中使用过！是的，这就是BeanPostProcessor在Spring底层的一种使用场景。至于上面的案例代码为何会在setApplicationContext()方法中获取到ApplicationContext对象，这就是`ApplicationContextAwareProcessor`类的功劳了！

接下来，我们就深入分析下ApplicationContextAwareProcessor类。

我们先来看下ApplicationContextAwareProcessor类中对于postProcessBeforeInitialization()方法的实现，如下所示。

![`image-20220522205703812](Spring注解.assets/image-20220522205703812.png)

在bean初始化之前，首先对当前bean的类型进行判断，如果当前bean的类型不是`EnvironmentAware`，不是`EmbeddedValueResolverAware`，不是`ResourceLoaderAware`，不是`ApplicationEventPublisherAware`，不是`MessageSourceAware`，也不是`ApplicationContextAware`，那么直接返回bean。如果是上面类型中的一种类型，那么最终会调用`invokeAwareInterfaces()`方法，并将bean传递给该方法。

invokeAwareInterfaces()方法又是个什么鬼呢？我们继续看invokeAwareInterfaces()方法的源码，如下所示。

```java
private void invokeAwareInterfaces(Object bean) {
		if (bean instanceof EnvironmentAware) {
			((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
		}
		if (bean instanceof EmbeddedValueResolverAware) {
			((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
		}
		if (bean instanceof ResourceLoaderAware) {
			((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
		}
		if (bean instanceof ApplicationEventPublisherAware) {
			((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
		}
		if (bean instanceof MessageSourceAware) {
			((MessageSourceAware) bean).setMessageSource(this.applicationContext);
		}
		if (bean instanceof ApplicationStartupAware) {
			((ApplicationStartupAware) bean).setApplicationStartup(this.applicationContext.getApplicationStartup());
		}
		if (bean instanceof ApplicationContextAware) {
			((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
		}
	}
```

可以看到`invokeAwareInterfaces()`方法的源码比较简单，就是`判断当前bean属于哪种接口`类型，然后将`bean强转为哪种接口类型`的对象，接着调用接口中的方法，将相应的参数传递到接口的方法中。这里，我们在创建Dog类时，实现的是ApplicationContextAware接口，所以，在invokeAwareInterfaces()方法中，会执行如下的逻辑代码。

```java
if (bean instanceof ApplicationContextAware) {
			((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
		}
```

我们可以看到，此时会将`this.applicationContext`传递到`ApplicationContextAware`接口的`setApplicationContext()`方法中。所以，我们在Dog类的`setApplicationContext()`方法中就可以直接接收到ApplicationContext对象了。

##### BeanValidationPostProcessor类

org.springframework.validation.beanvalidation.BeanValidationPostProcessor类主要是用来为bean进行校验操作的，当我们创建bean，并为bean赋值后，我们可以通过BeanValidationPostProcessor类为bean进行校验操作。BeanValidationPostProcessor类的源码如下所示。

```java
/*
 * Copyright 2002-2018 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.validation.beanvalidation;

import java.util.Iterator;
import java.util.Set;

import javax.validation.ConstraintViolation;
import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;

import org.springframework.aop.framework.AopProxyUtils;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanInitializationException;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.lang.Nullable;
import org.springframework.util.Assert;

/**
 * Simple {@link BeanPostProcessor} that checks JSR-303 constraint annotations
 * in Spring-managed beans, throwing an initialization exception in case of
 * constraint violations right before calling the bean's init method (if any).
 *
 * @author Juergen Hoeller
 * @since 3.0
 */
public class BeanValidationPostProcessor implements BeanPostProcessor, InitializingBean {

	@Nullable
	private Validator validator;

	private boolean afterInitialization = false;


	/**
	 * Set the JSR-303 Validator to delegate to for validating beans.
	 * <p>Default is the default ValidatorFactory's default Validator.
	 */
	public void setValidator(Validator validator) {
		this.validator = validator;
	}

	/**
	 * Set the JSR-303 ValidatorFactory to delegate to for validating beans,
	 * using its default Validator.
	 * <p>Default is the default ValidatorFactory's default Validator.
	 * @see javax.validation.ValidatorFactory#getValidator()
	 */
	public void setValidatorFactory(ValidatorFactory validatorFactory) {
		this.validator = validatorFactory.getValidator();
	}

	/**
	 * Choose whether to perform validation after bean initialization
	 * (i.e. after init methods) instead of before (which is the default).
	 * <p>Default is "false" (before initialization). Switch this to "true"
	 * (after initialization) if you would like to give init methods a chance
	 * to populate constrained fields before they get validated.
	 */
	public void setAfterInitialization(boolean afterInitialization) {
		this.afterInitialization = afterInitialization;
	}

	@Override
	public void afterPropertiesSet() {
		if (this.validator == null) {
			this.validator = Validation.buildDefaultValidatorFactory().getValidator();
		}
	}


	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		if (!this.afterInitialization) {
			doValidate(bean);
		}
		return bean;
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		if (this.afterInitialization) {
			doValidate(bean);
		}
		return bean;
	}


	/**
	 * Perform validation of the given bean.
	 * @param bean the bean instance to validate
	 * @see javax.validation.Validator#validate
	 */
	protected void doValidate(Object bean) {
		Assert.state(this.validator != null, "No Validator set");
		Object objectToValidate = AopProxyUtils.getSingletonTarget(bean);
		if (objectToValidate == null) {
			objectToValidate = bean;
		}
		Set<ConstraintViolation<Object>> result = this.validator.validate(objectToValidate);

		if (!result.isEmpty()) {
			StringBuilder sb = new StringBuilder("Bean state is invalid: ");
			for (Iterator<ConstraintViolation<Object>> it = result.iterator(); it.hasNext();) {
				ConstraintViolation<Object> violation = it.next();
				sb.append(violation.getPropertyPath()).append(" - ").append(violation.getMessage());
				if (it.hasNext()) {
					sb.append("; ");
				}
			}
			throw new BeanInitializationException(sb.toString());
		}
	}

}

```

这里，我们也来看看postProcessBeforeInitialization()方法和postProcessAfterInitialization()方法的实现，如下所示。

```java
@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		if (!this.afterInitialization) {
			doValidate(bean);
		}
		return bean;
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		if (this.afterInitialization) {
			doValidate(bean);
		}
		return bean;
	}
```

可以看到，在`postProcessBeforeInitialization()`方法和`postProcessAfterInitialization()`方法中的主要逻辑都是调用`doValidate()`方法对bean进行校验，只不过在这两个方法中都会对`afterInitialization`这个boolean类型的成员变量`进行判断`，若afterInitialization的值为false，则在`postProcessBeforeInitialization()`方法中`调用doValidate()`方法对bean进行校验；若afterInitialization的值为true，则在`postProcessAfterInitialization()`方法中`调用doValidate()`方法对bean进行校验。

##### InitDestroyAnnotationBeanPostProcessor类

org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor类主要用来处理`@PostConstruct`注解和`@PreDestroy`注解。

例如，我们之前创建的Dog类中就使用了@PostConstruct注解和@PreDestroy注解，如下所示。

```java
package com.lbgao.bean;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/**
 * @Auther: lbgao
 * @Date: 2022/5/22 20:49
 */

@Component
public class Dog implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Dog() {
        System.out.println("dog constructor...");
    }

    // 在对象创建完成并且属性赋值完成之后调用
    @PostConstruct
    public void init() {
        System.out.println("dog...@PostConstruct...");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("dog...@PreDestroy...");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}

```

那么，在Dog类中使用了@PostConstruct注解和@PreDestroy注解来标注方法，Spring怎么就知道什么时候执行@PostConstruct注解标注的方法，什么时候执行@PreDestroy注解标注的方法呢？这就要归功于`InitDestroyAnnotationBeanPostProcessor`类了。

接下来，我们也通过Debug的方式来跟进下代码的执行流程。首先，在Dog类的initt()方法上打上一个断点，如下所示。

![image-20220522211902231](Spring注解.assets/image-20220522211902231.png)

然后，我们以Debug的方式运行IOCTest_LifeCycle类中的test01()方法.

我们还是带着问题来分析，Spring怎么就能定位到使用`@PostConstruct`注解标注的方法呢？通过分析方法的调用栈，我们发现在进入使用`@PostConstruct`注解标注的方法之前，Spring调用了`InitDestroyAnnotationBeanPostProcessor`类的`postProcessBeforeInitialization()`方法，如下所示。

![image-20220522212101988](Spring注解.assets/image-20220522212101988.png)

```java
@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass());
		try {
			metadata.invokeInitMethods(bean, beanName);
		}
		catch (InvocationTargetException ex) {
			throw new BeanCreationException(beanName, "Invocation of init method failed", ex.getTargetException());
		}
		catch (Throwable ex) {
			throw new BeanCreationException(beanName, "Failed to invoke init method", ex);
		}
		return bean;
	}
```

在`InitDestroyAnnotationBeanPostProcessor`类的`postProcessBeforeInitialization()`方法中，首先会找到bean中有关生命周期的注解，比如@PostConstruct注解等，找到这些注解之后，则将这些信息赋值给`LifecycleMetadata`类型的变量metadata，之后调用metadata的invokeInitMethods()方法，通过反射来调用标注了`@PostConstruct`注解的方法。这就是为什么标注了@PostConstruct注解的方法会被Spring执行的原因。

##### AutowiredAnnotationBeanPostProcessor类

#### 十七、如何使用@Value注解未bean的属性赋值？

##### @Value注解

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Value {

	/**
	 * The actual value expression such as <code>#{systemProperties.myProp}</code>
	 * or property placeholder such as <code>${my.app.myProp}</code>.
	 */
	String value();

}
```

`从@Value注解的源码中我们可以看出，@Value注解可以标注在字段、方法、参数以及注解上，而且在程序运行期间生效。`

##### @Value注解的用法

###### 不通过配置文件注入属性的情况

通过@Value注解将外部的值动态注入到bean的属性中，一般有如下这几种情况：

- 注入普通字符串

```java
@Value("高洛彬")
private String name; // 注入普通字符串
```

- 注入操作系统属性

```java
@Value("#{systemProperties['os.name']}")
private String systemPropertiesName; // 注入操作系统属性
```

- 注入SpEL表达式结果

```java
@Value("#{ T(java.lang.Math).random() * 100.0 }")
private double randomNumber; //注入SpEL表达式结果
```

- 注入其他bean中属性的值

```java
@Value("#{person.name}")
private String username; // 注入其他bean中属性的值，即注入person对象的name属性中的值
```

- 注入文件资源

```java
@Value("classpath:/config.properties")
private Resource resourceFile; // 注入文件资源
```

- 注入URL资源

```java
@Value("http://www.baidu.com")
private Resource url; // 注入URL资源
```

###### 通过配置文件注入属性的情况

首先，我们可以在项目的src/main/resources目录下新建一个属性文件，例如person.properties，其内容如下：

```java
person.name=高
```

然后，我们新建一个MainConfigOfPropertyValues配置类，并在该类上使用@PropertySource注解读取外部配置文件中的key/value并保存到运行的环境变量中。

```java
package com.lbgao.config;

import com.lbgao.pojo.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

/**
 * @Auther: lbgao
 * @Date: 2022/5/22 21:36
 */

@Configuration
@PropertySource(value = {"classpath:/person.properties"})
public class MainConfigPropertyValues {
    
    @Bean
    public Person person() {
        return new Person();
    }
}

```

加载完外部的配置文件以后，接着我们就可以使用`${key}`取出配置文件中key所对应的值，并将其注入到bean的属性中了。

##### @Value中#{···}和${···}的区别

我们在这里提供一个测试属性文件，例如advance_value_inject.properties，大致的内容如下所示。

```java
server.name=server1,server2,server3
author.name=lbgao
```

然后，新建一个AdvanceValueInject类，并在该类上使用@PropertySource注解读取外部属性文件中的key/value并保存到运行的环境变量中，即加载外部的advance_value_inject.properties属性文件。

```java
package com.lbgao.bean;

import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

/**
 * @Auther: lbgao
 * @Date: 2022/5/22 21:43
 */

@Component
@PropertySource(value = {"classpath:/advance_value_inject.properties"})
public class AdvanceValueInject {
}

```

###### ${...}用法

{}里面的内容必须符合`SpEL表达式`，通过`@Value("${spelDefault.value}")`我们可以获取属性文件中对应的值，但是如果属性文件中没有这个属性，那么就会报错。不过，我们可以通过赋予默认值来解决这个问题，如下所示。

```java
@Value("${author.name:meimeixia}")
private String name;
```

上述代码的含义是表示向bean的属性中注入属性文件中的author.name属性所对应的值，如果属性文件中`没有author.name这个属性`，那么便向bean的属性中`注入默认值meimeixia`。

###### #{...}用法

{}里面的内容同样也是必须符合SpEL表达式。例如，

```java
// SpEL：调用字符串Hello World的concat方法
@Value("#{'Hello World'.concat('!')}")
private String helloWorld;

// SpEL：调用字符串的getBytes方法，然后再调用其length属性
@Value("#{'Hello World'.bytes.length}")
private String helloWorldBytes;

```

###### ${···}和#{···}的混合使用

`${···}`和`#{···}`可以混合使用，例如，

```java
// SpEL：传入一个字符串，根据","切分后插入列表中， #{}和${}配合使用时，注意不能反过来${}在外面，而#{}在里面
@Value("#{'${server.name}'.split(',')}")
private List<String> severs;
```

上面片段的代码的执行顺序：通过`${server.name}`从属性文件中获取值并进行替换，然后就变成了执行SpEL表达式`{'server1,server2,server3'.split(',')}`。

在上文中`#{}`在外面，`${}`在里面可以执行成功，那么反过来是否可以呢？也就是说能否让`${}`在外面，`#{}`在里面，就像下面这样呢？

```java
// SpEL：注意不能反过来，${}在外面，而#{}在里面，因为这样会执行失败
@Value("${#{'HelloWorld'.concat('_')}}")
private List<String> severs2;
```

答案是不能。因为Spring执行`${}`的时机要早于`#{}`，当Spring执行外层的`${}`时，内部的`#{}`为空，所以会执行失败！

##### 小结

- `#{···}`：用于执行SpEl表达式，并将内容赋值给属性
- `${···}`：主要用于加载外部属性文件中的值
- `${···}`和`#{···}`可以混合使用，但是必须`#{}`在外面，`${}`在里面