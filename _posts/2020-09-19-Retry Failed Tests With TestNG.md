---
layout: post
title:  "Retry Failed Tests With TestNG"
date:   2020-09-19T00:00:52-03:00
author: Santiago L. Schumacher
categories: framework
---

<h2>Why Retry Automated Tests with Selenium?</h2>

Retrying failed automated tests is fundamental in a good Automation Framework. It is so because things rarely go exactly as planned, there are numerous errors
that can happen during test executions which are false negatives, and shouldn't be shared with the team, because if they review the failures, and find out that 9 out of 10 times
its false negatives, it will cause distrust within your team with your Automated Tests. 
<br/>
Some reasons for these false negatives includes: <br />
<ul>
	<li>Network hiccups causing disconnection between the Client (Your Framework) and the Browser Driver - or between Selenium's Hub and Nodes if you're using Selenium Grid</li>
	<li>Temporary application slowness causing timeouts, that cannot be consistently reproduced</li>
	<li>If you're using dynamic input to the tests, such as obtaning random data from a Database, failures can be related to data issues instead of the Application</li>
	<li>Selenium or Browser Drivers bugs that would not happen to real users</li>
</ul>

Luckily for us, TestNG comes bundled with a way to deal with this. It just needs a bit of configuration to get it right.

<h2>The IRetryAnalyzer Interface</h2>
What we are interested in is an interface called <strong>IRetryAnalyzer</strong>. It is fairly straightforward, contains a single method called <code>retry</code>
which receives an <strong>ITestResult</strong> parameter. 
We must implement that method in such a way that if it returns *true* then the test will re-run, and won't otherwise.

<h2>Implementing IRetryAnalyzer - The Static (not-recommended) way</h2>
Let's look at how it will look like. I created a class called *RetryAnalyzer*, and placed it in my package *src/main/java/tetsrunner*:

{% highlight JAVA %}
package testrunner;

import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;
import org.testng.Reporter;

import utils.Constants;

/**
 * Retry class. If a test were to fail, it will try (1) more time. 
 * The number can be adjusted by changing the Constants.MAX_RETRY_COUNT value.
 * 
 * @author Santiago Schumacher
 *
 */
public class RetryAnalyzer implements IRetryAnalyzer {

	private int retryCount = 0; // Variable is intrinsic to a single test method, and should be Thread-safe if our framework is.
    private int maxRetryCount = Constants.MAX_RETRY_COUNT; // Defined in Constants class but you can write it here directly.
 
    @Override
    public boolean retry(ITestResult result) {
        if (++retryCount < maxRetryCount) {
            Reporter.log("Test failed. Retrying. Iteration: " + retryCount);
            return true;
        }
        return false;
    }
}
{% endhighlight %}

<br />

You might have noticed one thing, we receive the *ITestResult* parameter, but we aren't using it ! This is because TestNG will only get this far to analyze the Retry class, if 
the test failed. So there is no need to check if the execution failed, ie: <code> if (!result.isSuccess()) { ... } </code>.

<i>Note: Keep your max retry count to 2-3 iterations, we're merely giving the test another chance, not trying to *force* it to pass !</i> <br />

What we have now is enough to use the Retries system. You can test it out ! Create a test that looks something like this:

{% highlight JAVA %}
@Test(retryAnalyzer = RetryAnalyzer.class)
	public void testCase() {
		Assert.assertTrue(false); // Purposedly failing the Test.
	}
{% endhighlight %}

<br />

<h2>Implementing IRetryAnalyzer - The Dynamic (recommended!) way</h2>

Of course, the example above is not ideal because it forces the Test Automation Engineer to copy-paste the content between the parenthesis in every single test.
To enable Retry on your whole suite, let's start by creating another class in the same package as your RetryAnalyzer, called *AnnotationTransformer*: <br />

{% highlight JAVA %}
package testrunner;

import java.lang.reflect.Constructor;
import java.lang.reflect.Method;

import org.testng.IAnnotationTransformer;
import org.testng.annotations.ITestAnnotation;

public class AnnotationTransformer implements IAnnotationTransformer {

	@Override
	public void transform(ITestAnnotation annotation, Class testClass, Constructor testConstructor, Method testMethod) {
		annotation.setRetryAnalyzer(RetryAnalyzer.class);
		// Like the name implies, this will 'transform' your @Test method, and add the 'retryAnalyzer = RetryAnalyzer.class' inside.
	}
	
}
{% endhighlight %}

If you've been following the guide step-by-step and try to run the suite again, you will notice it no longer is retrying your Tests !! Oh no, is it broken ? <br />
No worries - it has a very simple fix - your listener (*AnnotationTransformer*) needs to be added to the testng.xml file, like this:

{% highlight XML %}
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd" >
  
<suite name="Regression" verbose="1">
	<listeners>
        <listener class-name="testrunner.AnnotationTransformer"/> 
        <!-- You can specify as many listeners as you have here, and it will be applied to the entire suite. -->
  	</listeners>
	<test name="Regression1" parallel="methods" thread-count="5">
		<parameter name="driverName" value="CHROME" /> <!-- Sending parameter values through testng.xml -->
		<packages>
			<package name="tests" /> <!-- Replace with the name of the package in which your tests reside -->
		</packages>
	</test>
</suite>
{% endhighlight %}

That's it for today's post. 
Thanks for reading, until the next time,
Santiago.