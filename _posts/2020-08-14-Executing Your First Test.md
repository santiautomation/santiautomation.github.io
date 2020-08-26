---
layout: post
title:  "Executing Your First Test"
date:   2020-08-22T14:25:52-05:00
author: Santiago L. Schumacher
categories: framework
---

In our [previous post]({% post_url 2020-08-14-Driver Factory %}), we staged a way to create Drivers, but we haven't used it yet.
This time we will create some sample tests, a <code>BaseTest.java</code> class, and use TestNG.xml to run them, that way we'll be able to test if
the DriverFactory is working exactly as we expect, and we can execute concurrent or parallel tests, in different browsers, without anything breaking.
 
<h3>Why have a Base Test ?</h3>

You should never be initializing / quitting WebDriver instances in your Test Classes, with the main reason being <strong>Maintenance</strong> and <strong>Reusability</strong>.
If very single test class you write contains something in the lines of : <br/>

{% highlight JAVA %}
@BeforeTest
public void initWebDriver() {
	System.setProperty("webdriver.chrome.driver", "path of the exe file\\chromedriver.exe");
	WebDriver driver = new ChromeDriver();
	driver.get("http://www.facebook.com");
}
{% endhighlight %}

It is easy to tell this code is unmaintenable, and strongly hardcoded. How do I switch browsers ? Change the application URL ? The path to your driver file? It would require you
to navigate through each java class, make the neccesary modifications, and maybe possibly commiting your changes leading to conflicts. <br />

To fix this we'll create a <strong>BaseTest</strong>. <br />
The purpose of this class (you can rename if you prefer) is to contain the logic regarding all the steps taken before/after running tests. <br />

To be clear: The only reason to use things like <code>@BeforeTest</code>,<code>@BeforeClass</code>,<code>@AfterMethod</code>, etc. in your Test Class is to satisfy test-specific pre or post-conditions, such as:

<ul>
  <li>Obtaining Data to execute the test from the Database/CSV/API etc.</li>
  <li>Navigating to a specific part of the WebApplication before starting the actual test</li>
  <li>Restoring the WebApplication original status before starting the test (ie: If your test disables an user, it might be a good idea to enable it again in an @AfterMethod)</li>
</ul>
<br />
<h3> Creating your BaseTest </h3>
Start by creating new package in your project if you don't already have it - right click on <code>src/main/java</code>, go to New -> Package and create:
<code>testrunner</code>

Inside the package create a new <code>abstract</code> class (we don't want anyone to create instances of the BaseTest!). <br />
Copy the code below inside: <br />

{% highlight JAVA %}
package testrunner;

import org.openqa.selenium.WebDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Listeners;
import org.testng.annotations.Optional;
import org.testng.annotations.Parameters;

import asserts.Validate;
import driver.DriverFactory;
import driver.DriverManager;
import listeners.TestMethodListener;
import logging.Logging;
import utils.Constants;

@Listeners(TestMethodListener.class)
public abstract class BaseTest implements Logging {
	
	protected static DriverManager driverManager;
	
	@BeforeMethod
	@Parameters({"driverName"})
	protected void setup(@Optional("CHROME") String driverName) {		
		initializeDriverManager(driverName);	
		driverManager.getDriver().navigate().to(Constants.getContextUrl());
	}
	
	/** Separating initialization of DriverManager because it's a static class and can have strange behaviors while running tests in parallel.
	 *  The synchronized Keyword is to prevent 2 Threads from calling a static class at the same time.
	 *  Read more about synchronization in <a>https://docs.oracle.com/javase/tutorial/essential/concurrency/syncmeth.html</a>
	 */
	private synchronized void initializeDriverManager(String driverName) {
		if (null == driverManager) {
			driverManager = DriverFactory.valueOf(driverName).getDriverManager();
		} else {
			driverManager.getDriver();
		}
	}
	
	@AfterMethod
	protected void cleanUp() {
		driverManager.quitDriver();
	}
	
	protected WebDriver getDriver() {
		return driverManager.getDriver();
	}
}
{% endhighlight %}

 The Method <code>Constants.getContextUrl()</code> contains the URL of your WebApplication, which will change depending on the Environment you're executing against.
 To use that Method effectively you will need 2 more classes, and a property file.
 
 Create a package <code>utils</code> in the root with the following Java code:
 <h4>PropertyReader</h4>
{% highlight JAVA %}
package utils;

import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

import logging.Logging;

public class PropertyReader implements Logging {

    private Properties prop = new Properties();

    public PropertyReader() {   	
        try (InputStream in = getClass().getResourceAsStream("/" + Constants.PROPERTIES_NAME)){
            try {
            	getLogger().debug("Attempting .properties file load.");
                prop.load(in);
            } catch (FileNotFoundException e) {
            	getLogger().error(Constants.PROPERTIES_NAME + " Property file not found", e.getLocalizedMessage());
            } 
        
        } catch (IOException e) {
        	getLogger().error("Error reading file " + Constants.PROPERTIES_NAME, e.getLocalizedMessage());
        }
    }

    public String getString(String propertyName) {
        return prop.getProperty(propertyName);
    }
    
    public Integer getInt(String propertyName) {
    	int temp = -1;
    	
    	try {
		temp = Integer.parseInt(prop.getProperty(propertyName));
    	} catch (NumberFormatException e) {
    		getLogger().error("The property named: " + propertyName + " cannot be parsed to an Int.");
    	}
        return temp;
    }
}
{% endhighlight %}
<br />
 
<h4>Constants</h4>
{% highlight JAVA %}
package utils;

public class Constants {

	public static final String PROPERTIES_NAME = "blaze.properties"; // TODO Replace 'blaze' with your application name
	private static PropertyReader props = new PropertyReader();
	
	public static String getContextUrl() {
		return props.getString("url");
	}
}
{% endhighlight %}

Your .properties file, has to be located in the <code>src/main/resources</code> folder, and for now will contains a single line: <br />
<code>url=${url}</code>
<br />
The ${url} will be replaced dynamically when your test starts, thanks to Maven Profiles ! 

<h3>Maven Profiles: How to run tests against different Environments</h3>

All you need is a small section in your <code>pom.xml</code>:

{% highlight XML %}
<profiles>
	<profile>
		<id>qa-blaze</id> <!-- Set to any name you want your profile to have -->
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
			<url>http://productionurl.com/</url> <!-- Set to the URL of your application. -->
		</properties>
	</profile>
</profiles>
{% endhighlight %}	

This is very powerful, as you can run <code>mvn clean install -P YOUR_PROFILE_NAME</code> and use it to execute your tests against dev environment, QA, Staging, Production....

Read more about Maven Profiles:
<ul>
  <li><a href="https://maven.apache.org/guides/introduction/introduction-to-profiles.html">Official Maven Profile Documentation</a></li>
  <li><a href="https://www.eclipse.org/lists/m2e-users/msg04586.html#:~:text=Here's%20how%20it%20works%20%3A,new%20Maven%20Profile%20selection%20interface.">Setting Maven Profile on Eclipse IDE</a></li>
  <li><a href="https://www.jetbrains.com/help/idea/work-with-maven-profiles.html#activate_maven_profiles">Setting Maven Profile with IntelliJ IDE</a></li>
</ul>

<h3>Final Step: A Sample Test</h3>

Finally we are able to open a Browser. 
For every single Test and Page Object Classes, we will work in a different folder:
<code>src/test/java</code>
Because this is the default folder for your tests picked up by the maven-surefire-plugin. We can have tests in other folders/packages but it requires additional configuration 
and is not, generally speaking, a good practice.

Create a package inside that folder called <strong>tests</strong>, and in there a Java Class called SampleTest, something along these lines:
{% highlight JAVA %}
public class SampleTest extends BaseTest {

	@Test(description = "Sample test !")
	public void thisIsAnEmptyTest() throws Exception {
		Reporter.log("Starting Sample test");
	}
}
{% endhighlight %}

Noticed anything strange ?
Even having an empty test, with 0 lines of code, I am able to:

<ol>
  <li>Execute the Test, which instantiates a WebDriver instance (Using Chrome by default - we can change that later)</li>
  <li>Navigate to the URL of the application, depending on the selected Environment.</li>
  <li>Once the test completes, BaseTest will cleanly and quietly close your WebDriver instance.</li>
</ol>

What about you, are you seeing the same results ? Feel free to drop me an e-mail if you want to debug unforseen problems.

<br/>
Thank you for sticking with me, and see you in the next post.
		



 

