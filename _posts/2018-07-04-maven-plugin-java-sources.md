---
layout: page
title:  "Creating a Maven Plugin for Java Sources Processing"
date:   2018-07-04 00:00:00 -0300
categories: java maven plugin source
---

> This article shows the basics about creating a custom Maven plugin. And how to get java sources in it.

## Before getting started

Just brushing up maven plugins basics:  

We can configure one or more plugins to use in our projects using Maven's pom.xml. 
We can either customize something from a default plugin or add extra plugins. For example:  

```xml
        <plugins>
			<plugin>
				<artifactId>maven-war-plugin</artifactId>
				<configuration>
					<warSourceDirectory>src/main/webapp</warSourceDirectory>
					<packagingExcludes>WEB-INF/lib/*.jar</packagingExcludes>
					<dependentWarExcludes>WEB-INF/**/*,index.*</dependentWarExcludes>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.appfuse.plugins</groupId>
				<artifactId>maven-warpath-plugin</artifactId>
				<version>2.2.1</version>
				<extensions>true</extensions>
				<executions>
					<execution>
						<goals>
							<goal>add-classes</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
            ....
        </plugins>
```  

Each plugin has one or more goals. Goal represent an action itself.  These goals may be attached to a phase, this is not required though. 

> Before creating a custom plugin, you are should have a good understanding of how to use Maven plugins (goals and phases).  

## Custom Maven plugin

Creating a simple Maven plugin may sound like a complex task, but actually it's pretty easy.  

At first, we start as the same as any other maven based project: *create a from an archetype*.

Maven provides an archetype for plugins: [maven-archetype-plugin](https://maven.apache.org/archetypes/maven-archetype-plugin/).  
So, create your project using it:

> mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-plugin -DarchetypeVersion=1.3

This generates a project with a *pom.xml* which contains the following:  

```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-plugin-plugin</artifactId>
        <version>3.2</version>
        <configuration>
            <goalPrefix>rss</goalPrefix>
            <skipErrorNoDescriptorsFound>true</skipErrorNoDescriptorsFound>
        </configuration>
        <executions>
            <execution>
                <id>help-goal</id>
                <goals>
                    <goal>helpmojo</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
```

### Mojo

The created project contains a class MyMojo, which you may rename as you prefer.  
**Mojo** stands for *Maven plain Old Java Object*. It's the responsable for the action itself, that is, it the implementation of a goal in this plugin.  

A *Mojo* is a java class that extends `AbstractMojo` and has the `@Mojo` annotation. In this class, you need to implement the method *execute*.  

```java
        @Mojo(name = "check")
        public class JavaProcessor extends AbstractMojo {
            public void execute() throws MojoExecutionException, MojoFailureException {
                ...
            }
        }
```        

In the `@Mojo` the *name* binds a goal to this plugin. In this example, the goal to be called is "check".  Another attribute (optional) of this annotation is *defaultPhase* (self explanatory). 

> *Calling this plugin & goal:* **mvn rss:check**

### Working with the java sources

This is the one part I ended up spending some time in my first plugin. I was trying to figure out how to get java compiler units from Maven build.  
But the point is what Maven provides is the project's properties, like the source directory. From this information, you'll get simple plain files.  
Then, you need to process these files using any way you prefer.

In the Mojo class: 

```java
        // ...
        @Parameter(defaultValue = "${project.build.sourceDirectory}", property = "sourceRoot")
	    private String sourceRoot;

        // ...
        File directory = new File(sourceRoot);
```

In this example, it's possible to specify the *sourceRoot* parameter within the plugin configuration. By default, it gets the builtin *project.build.sourceDirectory*.  

To process java files, there are some options:

* [Roaster](https://github.com/forge/roaster)
    * The one I generally use (I like its syntax)
* [Java Parser](https://github.com/javaparser/javaparser)

That's it!

*!EOF*

---
