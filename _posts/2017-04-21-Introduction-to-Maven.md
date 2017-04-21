---
layout: post
title: Introduction to Maven
---

### What is Maven

Maven is a build, complier, tester tool for Java projects, it can also be used with Scala.


### What is a build tool?

It is a tool that automates everything that is needed to building software projects whether it is in Java or Scala. Building software projects can include one or more of the following activities:

+ generating source code
+ generating documentation from the source code
+ Compiling the source code
+ Package the complied source code into JAR files (download any external JARs that are required for the stated dependencies)


### Maven Overview

Maven uses a POM file (Project Object Model) which contains all the information that Maven needs to build your software package. A POM file is in an XML format and contains information about the source code, test code and any dependencies. The POM file should be located in the root of the project folder. 


### How does it work?

1. Maven first reads the POM file which is called pom.xml as this pom file defines all the information that Maven needs to build your software project.

2. It then sees if any dependencies (need for an external JAR) are defined in the pom file and it then checks to see if they exist in your local Maven repository and if it doesn't Maven will download them to the local repository which is a folder stored on your computer.

3. Next depending on what you pass in as the parameter when you run the Maven command that is the type of build process is ran. Maven is split  up into build lifecycles, phases and goals. Where lifecycles are made up of phases and phases are made up of goals. If a lifecycle is passed in as a parameter when calling the Maven command all the phases that are in the lifecycle are executed and if a phase is passed in then all the phases that are before it in a pre-defined sequence are also executed.

4. Maven then executes any plugins which are used to insert extra goals into a build phase. This is useful if an action that you want to execute is non-standard. Then Maven will build phases or goals, just add a plugin to the POM file.

5. Maven then builds the profile based on the values that you specify in the profile part of the POM file, this means that you can have multiple build profiles and using an id tag to set different profile types. So when you build you project you specify which profile to use to build your project of.  


### POM 

As mentioned above POM is an xml file format and in this file it states the directories where the source code exists, where the test code exists, what dependencies your project uses and so on. POM file describes what to build and not how to build it. When your run the Maven command you specify the build and that tells Maven how to build it. You can write custom code for create your own goals when can then be inserted into the build phase. 

If in your project you have subprojects, you can have a POM file in each subproject and project, this will allow you build a subproject separately  and using the parent POM file to build the whole project. 


### Example of a POM file


In the modelVerison tag you need to specify what version of the POM model you are using and if you use Maven versions 2 and 3 then the POM modelVerison should be 4.0.0

```xml
<modelVerison>4.0.0</modelVerison>

```

When Maven is finished building the project it will use the value specified in the groupId tag to store the successful JAR file in.
Once Maven finishes building the project it will store the final JAR file under the ${MAVEN_HOME}/com/vishal/example/ where ${MAVEN_HOME} is a directory path of Maven's repository folder.


```xml
<groupId>com.vishal.example</groupId>
```


The value stored in the artifactId should be the name of the project and it is used as a subdirectory below the groupId directory. The artifactId is also used as a part of the name for the JAR file produced when Maven successfully builds your project and the output of a successful build is called an artifact.

```xml
<artifactId>maven-demo</artifactId>
```

You use the version tag to specify the version of this project, this will come in handy when you need to release a new version of the project and this tag will help to identify the latest version. The version tag is also used to create a subdirectory under the artifactId and used as part of the rest of the name for the JAR file along with the artifact name.

```xml
<version>1.0.0</version>
```

If we were to use the values from the above groupId, artifactId and version tags to build our project then the resulting JAR would be stored in the following location and saved as the following file name: ${MAVEN_HOME}/com/vishal/example/maven-demo/maven-demo-1.0.0-SNAPSHOT.jar

```xml

```


### Maven settings file

The file is called settings.xml and it can exist in the following areas:

+ In the Maven installation directory = ${Maven_Home}/conf/settings.xml
+ Under the user's home directory cd ~/.m2/settings.xml

In these files you can specify the location of the local repository (this is where the external JARs will be downloaded to) and more. If the settings.xml file exists under the user's home directory then the configuration values will overwrite the same configurations value in the settings file under the conf folder.


### Running Maven

You use the mvn command to run Maven but you also need to provide the build, phase or goal that Maven needs to execute. Note that you can pass more than one argument to the mvn command.

To execute a goal it is slightly different as you need to pass the build phase that is part of the goal that you execute and this needs to be separated by a ':'.         

e.g.    mvn dependency:copy-dependencies


### Dependencies

Dependencies are external JARs which may be needed by your code and these JARs need to be present in the classpath when you compile your project code. Either you can manually download each of these JARs and these JARs may require other JARs so you would have to download them as well. This can be a right pain as even new versions of the JARs can be released and it can be difficult to track but Maven can manage this, all you need to do is to add a dependency which states the groupId,artifactId and version. Maven will then find and download this JAR and any other JARs that this JAR that you stated needs, to the Maven repository.

example
```xml

```

You may need external JARs which maybe external meaning that it can't be found in a Maven repository and it may be found on the local HDD. So you need to state that Maven will not be able to find them in its repository and instead it can find the JAR on the local HDD. You need to use the tag scope and the value to be system, and then using the tag systemPath to specify the local of the JAR in reference to the location of the POM. Using ${basedir} as the location of the POM file and you then need to state the rest of the path from the "basedir".

example
```xml

```

### Maven Build Lifecycles, Phases and goals


Maven has 3 main build lifecycles default, clean and site. Each one of these build lifecycles looks after different parts of building the software project and each build lifecycle is executed independently of each other but they can be executed together but they will be executed in sequence. 

+ Default - handles everything related to compiling and packaging you project
+ Clean - handles everything related to removing temporary files souch as previous JARs, compiled classe etc
+ Site -handles everything related to documentation for the project

You can execute a whole build lifecycle, a build phase or a build goal but note that you can't execute the default build lifecycle directly and instead you need to specify a build phase or goal within the default lifecycle. When you execute a build phase all the build phases before the one you are trying to execute, are ran prior to the one you requested. The default build lifecycle is what builds the code but as mentioned above a build phase or goal from the default lifecycle needs to be executed.  

Below are some build phases of the default lifecycle:

+ Validate - validates the project and downloads any dependencies nessecary 
+ Compile - compiles the source code
+ Test - runs the tests against the compiled source code using a unit testing framework must common is Junit
+ Package - packages the compiled source code into a JAR file
+ Install - installs the package into the local repository which then could be used as a dependency in other local projects.


If you run the following command not only will it be executing the package build phase but it will be executing all the previous phases in the list above.

```
mvn package
```

### What is a Maven Archetype?

Maven Archetype is a project template which can be generated by Maven. There are many different types but below are the most common versions:

+ mvn eclipse:eclipse - this will generate a new Java project including files for the Eclipse IDE but before you can generate the eclipse archetype the POM file needs to be in the project root directory where you want to create the archetype.
+ mvn idea:idea - this will create a Intelli J IDEA archetype.


