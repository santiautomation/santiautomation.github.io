---
layout: post
title:  "Parallelization with TestNG"
date:   2020-09-13T00:00:52-03:00
author: Santiago L. Schumacher
categories: miscellaneous
---

<h2>Why Parallelization?</h2>

Parallelization is the notion of executing more than one test at the same time, exponentially decreasing the amount of time needed to run your test suite/s.
This can be achieved in your local computer, in a server with Jenkins or similar, or in several nodes (computers running Selenium instances) through Selenium Grid. <br/>

We live in a time where managers and users expect some fast changes to applications functionality. This is almost impossible to achieve without a good Test Automation Regression,
as big changes can break the application in unexpected places, even when parts of the application being modified seems disconnected at first glance.
Now, if we need to wait 12 hours (for example) for your tests to complete, it's not ideal, and can almost defeat the purpose of Automation. <br/><br/>
We must decrease the test time, and the most obvious and easiest answer is adding parallelization, to execute 2, 5, or 50 and sometimes more tests at the same time, but it's not our only option.
You can also think about: <br/>

<ul>
  <li>Using explicit Waits only, with timeouts that makes sense and are tested. Do not randomly and generically use "30 seconds" for very element.</li>
  <li>Keep your tests concise and to the point, do not add extra navigation steps and validations that are not needed or already taken care of in other tests. </li>
  <li>Find workarounds and shortcuts. For example, not every single test needs to go through the login process. You can set Cookies at the start of the session and start at the home page or even further along by using the right URLs</li>
  <li>Use API calls where possible and only test in the UI what needs to be tested.</li>
  <li>Possible some more options that I can't think of right now :)</li>
</ul>

<br/>
Althought it seems exciting at first, there are some things you need to keep in mind before jumping in on parallelization, and it's that Selenium is not Thread-safe by default,
so you will need to deal with that yourself, and that your application might have cases where its state is being modified at the same time, which will lead to false negatives, 
so the tests must be organized accordingly to run sequentially on those cases. <br />
Like the post title mentions, I will be showing parallel options for TestNG, but JUnit is just as capable (if not more) to do the same.

 

<h2>TestNG Internals</h2>

In [another post]({% post_url 2020-08-14-Driver Factory %}), when discussing how to manage instances of Selenium WebDriver, we were very careful of how we manage Threads, and making sure 
that a WebDriver instance is never referenced from a different thread, because it can have catastrophic results on our tests, and at the same time can be difficult to debug and fix.
We used some helpful classes such as <code>ThreadLocal</code> and <code>ThreadGuard</code>. <br/>

Now, how do we actually create different Threads, each with its WebDriver instance ? <br/>
This is where TestNG <strong>parallel</strong> comes in.

Before naming the options we have let's start by explaining how TestNG deals with parallelization internally by looking at the following diagram:

<br/>
![TestNG Parallelization](/assets/images/TestNGParallel.PNG)
<br/>

With the picture in mind, let's see what TestNG's <a href='https://testng.org/doc/documentation-main.html#parallel-running'>official documentation<a/> has to say, and I'll try to explain in my own words so it's clearer.
<br/>

| Parallel Type | Official Documentation | Meaning | Observations |
|:-:|:-:|:-:|:-:|
| instances | <i>TestNG will run all the methods in the same instance in the same thread, but two methods on two different instances will be running in different threads. </i> | To create more than one instance of a Test, you need to invoke your Test Class more than once, for example using <code><a href='https://www.journaldev.com/21237/testng-factory-annotation'>@Factory</a></code> | Parallel instances will behave exactly as parallel classes, unless you're invoking multiple instances of the same Test .java class. Overall, I would avoid this since it's not really useful, and it's full of <a href='https://github.com/cbeust/testng/issues/751'>open defects</a> |
| classes | <i>TestNG will run all the methods in the same class in the same thread, but each class will be run in a separate thread. </i> | One thread per .java class (limited by 'thread-count'). | Works great when you are keeping one Test Method per Java class, and Classes with multiple test methods will run sequentially. You have to organize each @Test in different Classes, which can result in having too many classes. |
| tests | <i>TestNG will run all the methods in the same <test> tag in the same thread, but each <test> tag will be in a separate thread. This allows you to group all your classes that are not thread safe in the same <test> and guarantee they will all run in the same thread while taking advantage of TestNG using as many threads as possible to run your tests. </i> | TestNG 'tests' refer only to <test> from your testng.xml file. That means if you want more than one Thread you will need to write one <test> tag for each one. | *Only* use this if you have no self-respect and are willing to deal with xml with thousand of lines. In other words, do not use this. |
| methods | <i>TestNG will run all your test methods in separate threads. Dependent methods will also run in separate threads but they will respect the order that you specified.</i> | Will spawn Threads by detecting the @Test annotations. This way you can point to packages in your testng.xml file leaving it with just a few lines and simple maintenance. | The best option TestNG offers to run parallel tests. |

<br/>
In summary, *my recommendation* is you can either:

<ol>
	<li>Create only one @Test method in each Java Class (only have more @Test if they run sequentially), and use <strong>parallel='classes'</strong></li>
	<li>Group the @Test methods in classes according to functionality (ie: LoginTests with many @Test methods), and use <strong>parallel='methods'</strong></li>
</ol>

You can find the sample testng.xml that I have on my <a href='https://github.com/santiautomation/santi-automation-framework'>framework</a> below:

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

Thanks for reading, until the next time,
Santiago.