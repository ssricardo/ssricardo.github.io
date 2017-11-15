---
layout: page
title:  "From Java EE to Spring Boot"
date:   2017-11-08 20:00:00 -0200
categories: java spring boot
---

> This article is based on notes took in a a migration of a Java EE (EJB + JSF) application to Spring. It approaches an overview of Spring Boot aspects and established a small comparison with Java EE. 

## Overview Spring Boot

* Spring World

**Spring** is a well-known framework for most part of Java developers. It has several components, which includes **Spring Boot**.  
As a framework, Spring Boot itself is rather small, it doesn't have a big API, but it dependents on spring-core.  
The main part of the Boot is a tool to generate a preconfigured application, according to your needs. Thus, creating a new application becomes a really easy and simple task. Basically, this is achieved with existing *archetypes* from Apache Maven.

Some of the archetypes from Spring Boot are the following:

* spring-boot-sample-simple-archetype
* spring-boot-sample-web-ui-archetype
* spring-boot-sample-batch-archetype

Besides these *archetypes*, Spring Boot has artefact conjuncts of type POM (from maven) which aggregates rather dependencies. Thus, instead of having 10 dependencies in your project to provide a type of functionality (Web service, for instance), a single one is enough: *spring-boot-starter-\**.

Example of the dependency:  

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>


In previous versions, Spring was strongly based on XML configurations. In current versions and on Spring Boot, this was modified. In general, configurations are specified by annotations in Java code directly.

## Migrating from JavaEE to Spring Boot

### Scenario

The application which was migrated was a simple system in the company I work. It has only a few registries to control and manipulate *jobs* based on Spring Batch.

Its first version used to run on Jboss AS, using **EJB 3.0** and **JSF 1.2**. 

Then the application was migrated to Spring Boot with embedded Tomcat. Consequently, EJB is not supported. The view continued to use JSF but it was updated to **2.2**.

### Steps

**Application Class**

The starting point is to create the **main** class and to annotate it with `@SpringBootApplication`. 
Starting from this class, Spring will scan the classpath and load the found settings.

**web.xml -> Java**

The Web application configurations, which were previously concentrated at web descriptor file (web.xml), had to be exchanged for Java coded configurations obeying a defined structure with Spring predefined signatures. 

*Some examples of configurations:*

* Servlet do JSF

		@Bean
	    public ServletRegistrationBean facesServletRegistration() {
	        FacesServlet servlet = new FacesServlet();
	        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(servlet, "*.xhtml");
	        servletRegistrationBean.setLoadOnStartup(1);
	        return servletRegistrationBean;
	    }  

* Context-param

		@Override
	    public void onStartup(ServletContext servletContext)
	            throws ServletException {
	
	        servletContext.setInitParameter("BEAN_INTERCEPTORS_LIST",
	                "br.com.rss.listeners.ExceptionHandler");
	   		...
	   }

> Repeat similar configuration para each part of web.xml: *listeners*, *filters*, *welcome-file-list*, *error-pages*...

**@ManagedBean**

The JSF managed beans (that is, annotated with *javax.faces.bean.ManagedBean*) didn't work correctly. Its dependency injection and instantiation didn't occur, thus they remain with *null* value. 
Horewer, if you use *javax.annotation.ManagedBean* instead, the Spring container gets to do the DI correctly. This way, it is possible to keep using a JEE annotation, @ManagedBean, instead of putting some Spring annotation (@Controller ou @Component). The advantage of using a specification annotation is keeping it independent and less coupled to any specific framework being used at the moment. 

*Scopes*

Following with *managed beans* modifications, it's necessary to configurate its scope yet.  
In this part, no annotation from the specification worked. Either *javax.faces.bean.RequestScoped* or *javax.annotation.RequestScoped* were ignored.  
Thus, it has to use the Spring proper annotation: *@Scope*. In this one, the chosen scope is pointed in the *value* field on the annotation: `@Scope("request")`.

**DAOs**

In DAO classes, the injection of EntityManager (*@PersistenceContext*) worked without any changes.  
The related modifications needed are: exchange EJB annotations by *@Repository*, and in the methods that make changes to the database, it's needed to add *@Transactional* annotation.

## Evaluation

### Strengths

The development of a Spring Boot application from the beginning to a running version is really **fast**, mainly due to automatic configurations and archetypes already existing.  
Its use is **simple**, moreover Spring has been already a **rather known** framework and also very used in Java community; therefore there is rather content for reference. 

**Application class: **   

		@SpringBootApplication
		public class Application extends SpringBootServletInitializer {
		
		    public static void main(String[] args) {
		        SpringApplication.run(Application.class, args);
		    }
		    
		    @Override
		    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
		        return application.sources(Application.class);
		    }
		
		}

### Negative points

In some cases, using Spring API instead of Java EE is like *six of one and half a dozen of the other*, that is, almost the same.  
Examples:

**Restful**

		// Usando Spring
		@RequestMapping("/account")
		public class AccountRestService {		
	
			@RequestMapping(method = RequestMethod.GET, value = "{accid}")
			@ResponseBody
			List<String> getSomething(@PathVariable final String accid) {...}


		// Java EE
		@Path("/account")
		public class AccountRestService {
		
			@GET
			@Path("/{accid}")
			public Account getAccount(@PathParam("accid") final String accid) {
				...
			}

**JSF**

		// Spring
		@Controller
		@Scope("request")
		public class JobBean implements Serializable {}

		// Java EE
		@ManagedBean
		@RequestScoped
		public class JobBean implements Serializable {}


> These are just a few examples, but there are other similar cases

There are also some cases when the configuration in Spring Boot is completely different from the one in Java EE. This is a negative point in places when Java EE is already mature and well known. For instance, **web.xml** (Deployment descriptor in web apps) was ignored in the refactored version with Spring Boot.  All configurations which previously were in that file, now they are made through a few Java APIs.

There are also some **integration issues**; it wasn't possible to use **CDI** combined with Spring duo to conflicts between them. 
In fact, this is predictable, since both CDI and Spring Core work on top annotation processing (injection).  
Another point that is worth highlighting is the use of **JSF** in this architecture. It is possible to use these technologies altogether, but there is that  limitation whereof Spring can't manage and inject *javax.faces.bean.ManagedBean*. Furthermore, Spring has its own MVC framework, which naturally is fully integrated with other Spring parts. Thus, when looking for "Spring Boot" online, the major part of content, examples and questions refer to Spring MVC instead of JSF. 

## Conclusions

> IMO

Spring started to get very strong when Java EE platform was in version 5, in which a lot of configurations were much complex and a lot of functionalities weren't available or mature. 
In this context, Spring was a complement to Java EE with simplified configurations. In that scenario, it was better than Java EE indeed. However Java EE in versions 6 and 7, has evolved a lot. Thus, major part of its configuration is already simple. In a similar way, Spring has continued evolving and improving itself. In this context, the core of Spring is an alternative to Java EE (not a complement anymore).  

Currently, for the major part of cases, there isn't a reason to use Spring core over Java EE (except by preference). For Dependency Injection, for instance, CDI can execute the same job very well. 
Following this idea of Spring Boot (unifying multiple technologies and providing a fast bootstrap for applications), some proposals have arisen based on Java EE (that is, no dependency on Spring). An example of this is [TomEE + Shrinkwrap](https://github.com/cchacin/tomee-boot), also described in [this article](http://www.beyondjava.net/blog/javaee-counterpart-spring-boot/). Another example of tools for fast bootstrapping is [Dropwizard](http://www.dropwizard.io/).

On the other side, there are still many parts of Spring that Java EE hasn't covered yet. 
Even with these alternatives, Spring Boot certainly has its space; mainly for new applications (where you don't need to adapt it from Java EE), or those focused on fast development. Or even for teams without much knowledge in Java EE stack, because it requires deeper knowledge of configurations in order to get it running. 
In advance, Spring has faster updates compared to Java EE.  

> *Summing up*: I prefer Java EE where it's possible. But Spring continues to offer some extra features.

*!EOF*

-----
