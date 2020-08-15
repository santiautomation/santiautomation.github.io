---
layout: post
title:  "Driver Factory"
date:   2020-08-14T14:25:52-05:00
author: Santiago L. Schumacher
categories: framework
---

<h1>Factory Pattern for Selenium WebDriver</h1>

On my previous posts we initialized a new maven Project, and added the dependencies we will need for our Framework.
Now we can get started with the main functionality of any Test Automation Framework, being able to create WebDriver instances
to interact with your Applications.

Let's get started with the basics, first, we need a way to easily choose between our favourite Browser (Chrome, Firefox, Edge ...) and change the URL of our
application depending on the environment.
 
<h3>We'll use</h3>
<ol>
  <li>Factory design pattern: A popular OOP pattern that allows for easier extension and decoupled code, where only the concrete classes need to know exactly <strong>how</strong> (logic) to create a new Instance.
  This helps create a really simple one-line-of-code way to create your WebDriver instance. This is our main focus on this post.</li>
  <li>Maven Profiles that will help us run test suites against different URL's in an easy way. We will cover this on subsequent posts</li>
</ol>

<h3> Implementing Factory Pattern </h3>
Start by creating new packages in our project, right click on <code>src/main/java</code>, go to New -> Package and create:
<ol>
  <li>drivers </li>
  <li>testrunner</li>
</ol>

You can name them as you wish of course, but always keep in mind that the idea of packages are to group similar classes/functionality inside specific folders to keep everything neatly organized and easy to find.

We want the following classes inside the <code>drivers</code> package:

<br/>
![Factory Pattern with Selenium](/assets/images/Factory.PNG)
<br/>

Now, let's go one by one.

<strong>DriverManager (Abstract Class)</strong>
{% highlight JAVA %}
package driver;

import java.util.concurrent.TimeUnit;

import org.openqa.selenium.WebDriver;

import logging.Logging;
import utils.Architecture;

public abstract class DriverManager {

	protected ThreadLocal<WebDriver> drivers = new ThreadLocal<>();
	protected abstract WebDriver createDriver();
	
	public void quitDriver() {
		if (null != drivers.get()) {
			try {
				drivers.get().quit();
			} catch (Exception e) {
				System.err("Unable to gracefully quit WebDriver.", e); // We'll replace this with actual Loggers later - don't worry !
			}
			
		}
	}
	
	public WebDriver getDriver() {
		if (null == drivers.get()) {
			drivers.set(this.createDriver());
		}
		drivers.get().manage().timeouts().implicitlyWait(1L, TimeUnit.SECONDS);
		
		return drivers.get();
	}
}
{% endhighlight %}

<br />

And the <strong>DriverFactory</strong>, which is an enum, containing all the concrete implementations for every Browser.
We can use methods inside the enum. These will make calling the Factory extremely simple !
If we want to create an instance of Chrome Driver, we can just type <code> DriverFactory.CHROME </code> and we're good to go, sounds simple, right ?
The only caveat is that implementation (adding new Browsers) can get a little tricky. 

{% highlight JAVA%}
package driver;

public enum DriverFactory {

	CHROME { 
		@Override
		public DriverManager getDriverManager() {
			return new ChromeDriverManager();
		}
	},
	FIREFOX {
		@Override
		public DriverManager getDriverManager() {
			return new FirefoxDriverManager();
		}
	};
	
	public abstract DriverManager getDriverManager();
}
{% endhighlight%}

<br/>
Finally, the implementations. I'll show ChromeDriver for now, since the rest of them are very similar:

{% highlight JAVA%}
package driver;

import static io.github.bonigarcia.wdm.DriverManagerType.CHROME;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

import io.github.bonigarcia.wdm.WebDriverManager;

public class ChromeDriverManager extends DriverManager {
	
	@Override
	protected WebDriver createDriver() {
		System.out.println("Initializing Chrome Driver"); // Change to Loggers
		WebDriverManager.getInstance(CHROME).setup();
	
		return new ChromeDriver(getChromeOptions());
	}

	private ChromeOptions getChromeOptions() {
		// A few valid Options for Chrome, showcase purpose.
		ChromeOptions options = new ChromeOptions();
		options.addArguments("--disable-notifications");
		options.addArguments("--start-maximized");
		options.addArguments("--disable-features=EnableEphemeralFlashPermission");
		options.addArguments("--disable-infobars");

		return options;
	}

}

{% endhighlight%}

Ok, looks like we have everything in place to create Selenium WebDriver instances, and now you might be wondering:
<ol>
	<li> My Client needs me to test with with Opera !! Your enum only mentions Chrome or Firefox ! To solve this you: 
		<ul>
			<li> Create a new concrete class that extends DriverManager. Call it OperaDriverManager. Replace <code>WebDriverManager.getInstance(CHROME)</code>
			with <code>WebDriverManager.getInstance(OPERA)</code> -- remember to change the static import as well -- and the return line to <code>return new OperaDriver()</code></li>
			<li> On the <strong>DriverFactory</strong> enum, add the following:
		{% highlight JAVA%}
	OPERA {
		@Override
		public DriverManager getDriverManager() {
			return new OperaDriverManager();
		}
	}
		{% endhighlight %}
			</li>
		</ul>
	</li>
	<li> "This looks cool but, how do I use it ?" - Well this one is easy. Before starting our tests we call <code>DriverFactory.valueOf(driverName).getDriverManager()</code>.
	Replace the "driverName" with "CHROME", or "FIREFOX", or any other implementation you might have coded yourself. </li>
</ol>

<br/>
Thank you for sticking with me, and see you in the next post.
		



 

