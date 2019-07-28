---
title: spring-aop-analyze
date: 2019-07-27 19:26:27
tags:
---


》//开启Aspect后，注册AnnotationAwareAspectJAutoProxyCreator，他根据@Pointcut注解定义的切点来自动代理相匹配的bean。
》`org.springframework.aop.config.AopConfigUtils#registerAspectJAnnotationAutoProxyCreatorIfNecessary(org.springframework.beans.factory.support.BeanDefinitionRegistry, java.lang.Object)`
    》》//spring加载bean时，为bean创建代理的入口
    》》`org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#postProcessAfterInitialization` 
        》》》//判断是否需要增强
        》》》`org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#wrapIfNecessary`
            》》》》//1，获取增强器
            》》》》`org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator#getAdvicesAndAdvisorsForBean`
                》》》》》//遍历所有的bean，找到所有的切面（@Aspect）
                》》》》》`org.springframework.aop.aspectj.annotation.BeanFactoryAspectJAdvisorsBuilder#buildAspectJAdvisors`
                    》》》》》》//获取切面中的通知（@Before、@After等）信息
                    》》》》》》`org.springframework.aop.aspectj.annotation.ReflectiveAspectJAdvisorFactory#getAdvisor`
                        》》》》》》》//根据通知信息生成增强
                        》》》》》》》`org.springframework.aop.aspectj.annotation.InstantiationModelAwarePointcutAdvisorImpl#InstantiationModelAwarePointcutAdvisorImpl`
                            》》》》》》》》//根据不同通知类型生成增强（Advice）
                            》》》》》》》》//生成的增强方法会由对应的Interceptor执行
                            》》》》》》》》//例如AspectJMethodBeforeAdvice中的方法会由org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor#invoke执行
                            》》》》》》》》//Interceptor会在？
                            》》》》》》》》`org.springframework.aop.aspectj.annotation.ReflectiveAspectJAdvisorFactory#getAdvice`
                》》》》》//2，匹配增强器，寻找增强器中适用于当前class的增强器
                》》》》》`org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator#findAdvisorsThatCanApply`
                    》》》》》》//引介增强和普通的增强处理不一样，所以会分开处理
                    》》》》》》`org.springframework.aop.support.AopUtils#canApply(org.springframework.aop.Advisor, java.lang.Class<?》, boolean)`
            》》》》//3，创建代理
            》》》》`org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#createProxy`

》//配置两个参数
》//1，是否强制使用cglib：proxy-target-class
》//2，是否暴露代理对象：expose-proxy；用于被jdk代理的目标对象的自调用方法。
》`org.springframework.context.annotation.AspectJAutoProxyRegistrar#registerBeanDefinitions`
