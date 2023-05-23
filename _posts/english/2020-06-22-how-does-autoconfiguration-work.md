---
layout: post
title:  "How does autoconfiguration in Springboot work?"
subtitle: "brief mechanism underneath"
description:
date:   2020-06-22 00:00:00 +0900
tags: Springboot
comments: True
image: /assets/img/spring.jpeg
optimized_image: /assets/img/spring.jpeg
author: tlonist
category: Spring
---


### Prelude

(Almost)Every Springboot user knows that the convinience of the framework comes largely from the **autoconfiguration**. But exactly how is it lessening the burden, and how does this auto-process work? Today's topic is to briefly cover these two questions.

### What does it do? (How does it help?)

- First off, straight from the source.
>Enable auto-configuration of the Spring Application Context, attempting to guess and
configure beans that you are likely to need. Auto-configuration classes are usually
applied based on your classpath and what beans you have defined. For example, if you
have {@code tomcat-embedded.jar} on your classpath you are likely to want a
{@link TomcatServletWebServerFactory} (unless you have defined your own
>{@link ServletWebServerFactory} bean).

- As the explanation clearly suggests, all it does is creating beans that the user is likely to use without actually defining the beans himself. In short, it is a bunch of pre-figured @Configuration classes with slightly different semantics. For example,

@ConditionalOnClass : when beans meeting all the specified requirements are already contained in the BeanFactory. 
@ConditionalOnMissingBean : when a given bean is yet to be craeted
@Conditional : when a given condition is met

The above annotations (and others) act upon depending classes **conditionally** according to what choices the user has made. 
[![img]({{ "/assets/img/conditional_1.png"|absolute_url}})]({{ "/assets/img/conditional_1.png"|absolute_url}})

The example is the H2ConsoleAutoConfiguration class that makes use of many @Conditional annotataions. Especially, **@ConditionalOnClass(WebServlet.class)** suggests that WebServlet.class must be present in order for the class to be registered as a bean.


### How does it work?

- If you want to register a bean, normally you may use constructor injection or @autowired annotation, so that Spring beanfactory can find and register the beans for you. For beans not declared using annotations like @Configuration, @Component, @Services and etc, Spring will look into autoConfiguration classes (@Configuration + @Conditional) to scan and instantiate when necessary.

[![img]({{ "/assets/img/factory.png"|absolute_url}})]({{ "/assets/img/factory.png"|absolute_url}})

- Looking into the Spring-AutoConfigure library, inside spring.factories you can find lots of autoconfiguring classes like below.
[![img]({{ "/assets/img/autoconfig1.png"|absolute_url}})]({{ "/assets/img/autoconfig1.png"|absolute_url}})

All these classes will be tested on conditions for bean instantiating. This autoconfiguring process is activated upon **@EnableAutoConfiguration** annotataion included with **@SpringbootApplication**. This autoconfiguration has some degrees of freedom. For example, you can
- exclude some classes from autoconfiguration process using **exclude**, like @EnableAutoConfiguration(exclude{BatchAutoConfiguration.class})
- include your own autoconfiguration class.
- specify a default bean which allows being overridden in the case that a more specific bean of the same type is present in the context. [link](https://stackoverflow.com/questions/50796810/what-does-the-spring-annotation-conditionalonmissingbean-do)


