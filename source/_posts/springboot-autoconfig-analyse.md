---
title: spring boot自动配置的使用及原理分析
date: 2018-11-05 11:06:51
tags:
---

spring boot提供的自动配置，可以简化项目的配置，封装成starter后可以降低项目之间的依赖。

#### spring boot使用自动配置的方式有两种：
##### 1. 直接使用@Configuration + @ConditionalOnClass/@ConditionalOnMissingBean + @ConfigurationProperties + @EnableConfigurationProperties。这种方法可以在存在或者不存在一些类时，直接加载配置。

##### 2. 定义Enable*注解/在spring.factories文件中增加配置类 + @Import + @Configuration + @ConditionalOnClass/@ConditionalOnMissingBean + @ConfigurationProperties + @EnableConfigurationProperties。。这种方法可以有选择性的加载特定的配置。

<!-- more -->

