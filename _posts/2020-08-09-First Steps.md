---
layout: post
title:  "Getting started building your own Framework"
date:   2020-08-08T14:25:52-05:00
author: Santiago L. Schumacher
categories: framework
---

<h1>Your first step towards a robust Test Automation Framework</h1>

If you've been reading my first post on the framework category, you know we gave a rough overview of some of the capabilities the Client has asked of our framework.
Feel free to navigate to my previous post and write them down if it'll help you follow along my postings.

This time around we'll go ahead and get started with a little coding. 

<h3>Requirements</h3>
<ol>
  <li>Java: Make sure <a href='https://java.com/en/download/help/download_options.xml'>Java is correctly installed in your System</a>. We need to install the development kit (JDK). Remember to
  confirm it was correctly installed by opening a Command line console and typying <code>java -version</code></li>
  <li>Your favorite IDE. Personally, I use <a href='https://www.eclipse.org/downloads/'>Eclipse</a> out of habit, but most developers lately have been inclining towards <a href='https://www.jetbrains.com/idea/download/'>IntelliJ IDEA</a></li>
  <li>Start a Repository. Being able to keep track of changes and work efficiently in teams is fundamental. GIT is the go-to choice here</li>
</ol>

<br/>
Create a new <strong>Maven</strong> project. When prompted to select an Archetype, for now we will stick with <code>maven-archetype-quickstart</code> as it will give you the structure we need.
<br/>
At this point your Project looks something like this:

<br/>
![Project structure after first step](/assets/images/MavenQuickstartArchetypeProject.PNG)
<br/>

We will add all the code relevant to the Framework, such as the Loggers, a Base Test for the pre-post test configurations, listeners, utils, retry analyzers etc. in the <code>src/main/java</code> package.
The  <code>src/main/test</code> package will only contain the Tests and Page Objects.

Don't sweat it if at this point you read a bunch of words you're not familiar with as we will be explaining each one.

Open the maven's pom.xml file.
Let's copy the following content there for now (we will circle back to the file to add more things are needed)


{% highlight XML %}
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>io.github.santiautomation</groupId> <!-- Replace with your company/project names! -->
	<artifactId>santi-test-framework</artifactId>
	<version>1.0.0</version>
	<packaging>jar</packaging>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<log4j.version>2.13.0</log4j.version> <!-- Putting repeated versions as a property makes for easier changes  -->
	</properties>

	<dependencies>
		<dependency> <!-- Contains many awesome Utilities classes and Object definitions -->
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>28.1-jre</version>
		</dependency>
		
		<dependency> <!-- Needed for the WebDriverManager -->
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
			<version>4.5.11</version>
		</dependency>
		
		<dependency>
			<groupId>org.testng</groupId>
			<artifactId>testng</artifactId>
			<version>7.1.0</version>
		</dependency>

		<dependency> <!-- Logging Facade that we can use with Log4J -->
		    <groupId>org.slf4j</groupId>
		    <artifactId>slf4j-log4j12</artifactId>
		    <version>1.7.30</version>
		    <scope>test</scope>
		</dependency>
		
		<dependency> <!-- Loggers ! -->
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-api</artifactId>
			<version>${log4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-core</artifactId>
			<version>${log4j.version}</version>
		</dependency>
		
		<dependency> <!-- Needed for API testing -->
			<groupId>io.rest-assured</groupId>
			<artifactId>rest-assured</artifactId>
			<version>4.2.0</version>
			<exclusions>
				<exclusion>
					<groupId>org.apache.httpcomponents</groupId>
					<artifactId>httpclient</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		
		<dependency>
			<groupId>org.seleniumhq.selenium</groupId>
			<artifactId>selenium-java</artifactId>
			<version>4.0.0-alpha-4</version> <!-- Use this one or replace with the latest stable version -->
		</dependency>
		
		<dependency> <!-- Mobile Testing -->
			<groupId>io.appium</groupId>
			<artifactId>java-client</artifactId>
			<version>7.3.0</version>
			<exclusions> <!-- To make sure we are getting the dependencies from only one place. Saves you a lot of headaches later -->
				<exclusion>
					<groupId>org.seleniumhq.selenium</groupId>
					<artifactId>selenium-support</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.apache.httpcomponents</groupId>
					<artifactId>httpclient</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.seleniumhq.selenium</groupId>
					<artifactId>selenium-api</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.seleniumhq.selenium</groupId>
					<artifactId>selenium-java</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		
		<dependency> 
			<groupId>io.github.bonigarcia</groupId>
			<artifactId>webdrivermanager</artifactId>
			<version>3.8.1</version>
			<exclusions>
				<exclusion>
					<groupId>org.apache.httpcomponents</groupId>
					<artifactId>httpclient</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
	</dependencies>
	
	<build>
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<filtering>true</filtering>
			</resource>
		</resources>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-resources-plugin</artifactId>
				<version>3.1.0</version>
				<configuration>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<plugin>
				<artifactId>maven-surefire-plugin</artifactId>
				<version>2.22.2</version>
				<configuration>
					<suiteXmlFiles>
						<suiteXmlFile>src/test/resources/testng.xml</suiteXmlFile>
					</suiteXmlFiles>
				</configuration>
			</plugin>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.8.1</version>
				<configuration>
					<source>11</source> <!-- Replace with Java version -->
					<target>11</target>
				</configuration>
			</plugin>
		</plugins>
	</build>
	
	<!-- profiles -->
	<profiles>
		<profile>
			<id>qa-blaze</id>
			<properties>
				<url>http://demoblaze.com/</url> <!-- This value is dynamically replaced in your properties file depending on your selected Profile. -->
			</properties>
			<activation>
				<activeByDefault>true</activeByDefault>
			</activation>
		</profile>
		<profile>
			<id>prod-blaze</id>
			<properties>
				<url>http://productionurl.com/</url> <!-- Replace with website you're testing -->
			</properties>
		</profile>
	</profiles>
</project>

{% endhighlight %}

Head over to my next post so we can finally run a Selenium WebDriver instance and get things more interesting !



 

