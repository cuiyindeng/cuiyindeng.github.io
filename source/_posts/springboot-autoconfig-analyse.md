---
title: spring boot自动配置的使用及原理分析
date: 2018-11-05 11:06:51
tags:
---

spring boot提供的自动配置，可以简化项目的配置，封装成starter后可以降低项目之间的依赖。

<!-- more -->

#### spring boot使用自动配置的方式有两种：
* 1.在spring.factories文件中增加配置类 + @Configuration + 
@ConditionalOnClass/@ConditionalOnMissingBean + @ConfigurationProperties + @EnableConfigurationProperties + @EnableAutoConfiguration。这种方法可以在存在或者不存在一些类时，直接加载配置。

##### 原理分析：

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan //默认扫描所在包下面的class。
@EnableHelloStarter //启用自定义的starter模块。相当于导入Auto Configuration类；其中的package name尽量不要和调用者的package name相同。
@RestController
public class App {

    @Autowired
    HelloServiceExtendProperties helloServiceExtendProperties;
    @Autowired
    private HelloService helloService;

    @RequestMapping("/")
    public String index() {
        helloService.setMsg(helloServiceExtendProperties.getMsg());
        return helloService.sayHello();
    }

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

上面的**@EnableAutoConfiguration**是自动配置的关键，
它会调用AutoConfigurationImportSelector的selectImports方法，该方法会获取所有的配置类。
最终spring会调用配置类的后置处理器-ConfigurationClassPostProcessor，将配置类注册到spring容器中。

```java
public class AutoConfigurationImportSelector
		implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware,
		BeanFactoryAware, EnvironmentAware, Ordered {

	private static final String[] NO_IMPORTS = {};

	private static final Log logger = LogFactory
			.getLog(AutoConfigurationImportSelector.class);

	private ConfigurableListableBeanFactory beanFactory;

	private Environment environment;

	private ClassLoader beanClassLoader;

	private ResourceLoader resourceLoader;

	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		try {
			AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
					.loadMetadata(this.beanClassLoader);
			AnnotationAttributes attributes = getAttributes(annotationMetadata);
			List<String> configurations = getCandidateConfigurations(annotationMetadata,
					attributes);
			
            ......

			return configurations.toArray(new String[configurations.size()]);
		}
		catch (IOException ex) {
			throw new IllegalStateException(ex);
		}
	}

    ......

	/**
	 * 获取@EnableAutoConfiguration注解的属性。
	 */
	protected AnnotationAttributes getAttributes(AnnotationMetadata metadata) {
		String name = getAnnotationClass().getName();
		AnnotationAttributes attributes = AnnotationAttributes
				.fromMap(metadata.getAnnotationAttributes(name, true));
		Assert.notNull(attributes,
				"No auto-configuration attributes found. Is " + metadata.getClassName()
						+ " annotated with " + ClassUtils.getShortName(name) + "?");
		return attributes;
	}

    ......

	/**
	 * 获取META-INF/spring.factories中的所有配置类
	 */
	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
			AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
				getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
		Assert.notEmpty(configurations,
				"No auto configuration classes found in META-INF/spring.factories. If you "
						+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
    ......
}
```

* 2.定义Enable*注解 + @Import + @Configuration + 
@ConditionalOnClass/@ConditionalOnMissingBean + @ConfigurationProperties + @EnableConfigurationProperties + @EnableAutoConfiguration。这种方法可以有选择性的加载配置。

##### 原理分析：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(HelloServiceAutoconfiguration.class)
public @interface EnableHelloStarter {

}
```

Enable*注解的@Import会用三种方式将配置类注册到spring容器中，方式如下

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

	/**
     *支持下面三种注册Bean的方式：
	 * {@link Configuration}, {@link ImportSelector}, {@link ImportBeanDefinitionRegistrar}
	 * or regular component classes to import.
	 */
	Class<?>[] value();

}
```

>参考：
>https://blog.csdn.net/xichenguan/article/details/73478873