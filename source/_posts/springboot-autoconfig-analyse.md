---
title: spring boot自动配置的原理分析
date: 2018-11-05 11:06:51
tags:
---

 spring boot使用自动配置的方式有两种：
##### 1. 直接使用@Configuration + @ConditionalOnClass/@ConditionalOnMissingBean

##### 2. 