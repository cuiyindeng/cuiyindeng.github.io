---
title: 分析spring ioc的实现
date: 2019-07-29 11:33:36
tags:
---


1：bean的生命周期

<!-- more -->

2：IOC容器是如何解决循环依赖的

3：spring的哪中循环依赖是不能不解决的？ 为啥不能解决？

1）org.springframework.context.support.AbstractApplicationContext#getbean 

2）org.springframework.beans.factory.support.AbstractBeanfactorydogetbean

    2.1）transformedbeanname解析我们的别名

    2.2）getsingleton首先尝试去我们的缓存池中去获取对象，（由于是第一次创建Bean所以缓存池中没有对象）

    2.3）若出现循环依赖，判断bean是不是单例的不是就抛出异常

    2.4）当前容器是否有父工厂有由父工厂加教

    2.5）合并我们的Bean定义

    2.6）检查我们当前的bean定义是不是抽象的？

    2.7）做dependson的依赖检查

    2.8）调用 getsingleton的方法

        2.8.1）beforesingletoncreation标志当前的Bean正在被创建日
3）createbean创建Bean的流程
    
    resolvebeforeinstantiation aop过程中找切面（ aspectj）
    
    3.1）docreatebean：调用我们真正的创建bean的流程

4）org. springframework beans. factory support. Abstractautowirecapablebeanfactory#docreatebean

    4.1）createbeaninstance（）调用我们的构造器来进行创建对象（属性还没有被赋值，）（早期对象）
    
    4.2）提前暴露我们的早期对象..加入到我们的缓存中

    创建A----去给我们的A中的属性B赋值的过程，发现依赖B对象然后 getbean（ Instb）

    创建B的流程去给我们B中的属性赋值的时候，发现依赖A对象

》//解析bean
》`org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors`
    》》`org.springframework.context.support.PostProcessorRegistrationDelegate#invokeBeanFactoryPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, java.util.List<org.springframework.beans.factory.config.BeanFactoryPostProcessor>)`
        》》》`org.springframework.context.support.PostProcessorRegistrationDelegate#invokeBeanDefinitionRegistryPostProcessors`
            》》》》`org.springframework.context.annotation.ConfigurationClassPostProcessor#postProcessBeanDefinitionRegistry`
                》》》》》//找到Config类
                》》》》》`org.springframework.context.annotation.ConfigurationClassPostProcessor#processConfigBeanDefinitions`
                    》》》》》》//解析Config类下的bean
                    》》》》》》`org.springframework.context.annotation.ConfigurationClassParser#parse(java.util.Set<org.springframework.beans.factory.config.BeanDefinitionHolder>)`
                        》》》》》》》//生成Config类所在的package
                        》》》》》》》`org.springframework.context.annotation.ConfigurationClassParser#processConfigurationClass`
                            》》》》》》》》//解析Config类所在的package
                            》》》》》》》》`org.springframework.context.annotation.ComponentScanAnnotationParser#parse`
                                》》》》》》》》》//扫描package下的class
                                》》》》》》》》》`org.springframework.context.annotation.ClassPathBeanDefinitionScanner#doScan`
                                    》》》》》》》》》》//找到package下spring托管的bean，并初始化bean定义
                                    》》》》》》》》》》`org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#findCandidateComponents`


》//初始化bean
》`org.springframework.context.support.AbstractApplicationContext#finishBeanFactoryInitialization`
    》》spring 容器初始化时调用“预实例化单例”方法
    》》`org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons`
        》》》根据bean名称获取容器中的bean
        》》》`org.springframework.beans.factory.support.AbstractBeanFactory#getBean(java.lang.String)`
            》》》》`org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean`