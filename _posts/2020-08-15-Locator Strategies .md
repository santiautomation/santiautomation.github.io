---
layout: post
title:  "Effective Locators in Selenium"
date:   2020-08-15T00:00:52-03:00
author: Santiago L. Schumacher
categories: miscellaneous
---

<h2>Successful Locator Strategies for WebElements</h2>

Let's discuss now the best strategies, patterns, tips, etc to come up with good, robust Locators that are the least likely to break down when there are 
changes in your application.

Finding the WebElements in your application is rarely difficult (with some exceptions, of course), but, knowing how to write <strong>great</strong> locators, thinking ahead,
and keeping things organized can be troublesome.

I will try to detail here some of the tips I always think of before writing a locator.

If you are not new to the Selenium world, you've likely already read about the following available <strong>Locator Strategies</strong>:

<ul> 
<li><strong>ID</strong></li> 
<li><strong>NAME</strong></li> 
<li><strong>LINKTEXT</strong></li> 
<li><strong>PARTIAL LINKTEXT</strong></li> 
<li><strong>TAG NAME</strong></li> 
<li><strong>CLASS NAME</strong></li> 
<li><strong>CSS</strong></li> 
<li><strong>XPATH</strong></li> 
</ul>

You probably also read about the Order (in terms of relevance) in which to choose them, if all are available:
<blockquote>
ID > NAME > CLASS NAME > LINKTEXT > CSS > XPATH
</blockquote>

I do not intend to go against the grain here, however, keep the following in mind:

<ul>
<li>ID's are always your safest bet, BUT: <br/>
	- They are not always unique: Althought a bad practice in Web Development, it is possible to have multiple elements sharing the same ID in one single page <br/>
	- Some Websites, specially the ones that create content for you, such as ERP's (Enterprise Resource Planning) and CRM (Customer Relationship Management), or are created dynamically, will generate ID's for the Web Developers. That means, for us, that the ID will change every time you reload your page. Lucky for us this easy to tell, as they will always look something like <strong>"SomeId_123456"</strong>. If your id is concatenated with some numbers or a large set of characters, <strong>do not use ID strategy</strong></li>
<li>Name and Class Name are often repetead in other elements. Keep this in mind before using them.</li>
<li>Choosing when to use CSS or XPATH comes out to personal preference. The difference in performance is negligible, in fact, XPATH in many circumstances comes out on top of CSS, so, you are safe to ignore the haters here</li>
<li>XPATH is the <strong>only</strong> locator able to find any web element, on any page. Making it the most exhaustive one, and useful for the trickiest elements</li>
</ul>


<section class="tips-container">
    <div class="tip"> <strong> Tip: </strong><br/>{% raw %}When using Selenium's Page Factory, replace your wordy <code>@FindBy(how = How.ID, using = "recipient-email")</code>, 
	with the shorter, more readable form <code>@FindBy(id = "recipient-email")</code> {% endraw %}
	</div>
</section>
<br/>
<h3>A Case For XPATH's</h3>

I won't discuss much about ID's or Class Names here since there really isn't much to talk about, instead, let me play devil's advocate. Personally, I've mostly used
<strong>XPATH</strong>. I've already stated some of my reasonings, but to sum up:

<div>
<ol>
	<li>Performance-wise, in general, CSS does better than XPATH. The difference between the two of them is, however, negligible. No one, in the history on Test Automation, has ever said that a test ran too slow because of the Locators.
	If you're a numbers kind of guy, you can find the Math in posts such as <a href ="http://elementalselenium.com/tips/33-xpath-vs-css-revisited">here</a>. So, the next time you hear someone complain about how slow 
	XPATH is, you can remind him that the bottleneck will <strong>always</strong> be loading the page itself, never the locators. This should never be on the table.</li>
	<li>XPATH's are able to find Elements by text, which is extremely useful. Consider the case where you got data from a Database or API, and need to compare it against your UI. Add this to the
	fact that you can also add other filters to the text, such as <code>contains</code>.  You will find explanations of each of them on my <a href='https://docs.google.com/spreadsheets/d/13a-wUwPx-9BIpa4odchN-mmnEZd2j8KHH9uO_RNBblg/edit?usp=sharing'>XPATH CHEATSHEET</a></li>
	<li>With XPATH you can find parent elements, or ancestors, which is not possible with CSS. </li>
	<li>Most people say that CSS is more readable and easiear to understand than XPATH, and this is, of course, debatable. It comes down to one of my earlier points: It depends on your preference. Stick to what you feel more comfortable with, and try not to mix them too much</li>
</ol>
</div>

I am a strong believer that most of the hate you will find towards XPATH's online comes from bad use of it, because there is such a thing as horrible xpaths, which are not common in other locators.<br/><br/>
For example, the locator:<br/>
 <code>//div/ol//*[2]</code> is a prime example of <strong>bad</strong> xpath. 
 <br/><br/>
Why ? Huh... <br/>
1 - It tells us nothing of what we're trying to find from looking at it (Other than it's an element inside a list). If you re-write to something in the lines of:
<code>//ol//a[contains(@class, 'post-link')]</code> you will know in a simple glimpse, that you're looking for a list of Posts links.
This will, together with smart variable naming convention, save you hours and hours of maintenance work.<br/>
2 - That locator is extremely fragile. It'll stop working at the slightest change in the UI of your Application. You want to find 1 or 2 main points(or anchors) that are unlikely to change
and use them.<br/><br/>
Consider for example:<br/>
<i>Bad Locator:</i><code>//body//section[@class='main-montent']//form//div/button[1]</code><br/>
<i>Good Locator:</i><code>//form//button[contains(@class, 'save-form')]</code><br/><br/>
In the example above, if the UI changes, such as moving the 'Save' button from the right side to the left, or adding more content to the form, will cause your tests to break.
The second example, on the other hand, will simply find a button that contains the <code>save-form</code> class, and at the same time belongs to the form. It is not depedentant to <strong>where</strong> the button is placed inside the form. That is how an xpath should look like.

Another very important tip from me is, if you're using xpath, never use the "<i>Copy Xpath</i>" option found in all popular browsers. They will, most of the time, provide you with awful locators that are maintenance hell and difficult to read and understand.
Take a little time to learn XPATH by yourself and write a meaningful locator !<br/> It might take some time getting used to it, but it will save you countless hours in the long run.

<section class="tips-container">
    <div class="tip"> <strong> Tip: </strong><br/>{% raw %}Always use Page Factory (ie: Selenium's @FindBy annotation), with the only 2 expections being:<br/>
	1 - You want dynamic locators, for example, creating a method which receives an Username as a paramter, and the locator being dependant, such as <code>driver.findElement(By.xpath(String.format("//p[contains(text(), '%s')]", user)))</code>{% endraw %}<br/>
	2 - You have a WebElement and need to find its childs --> <code>myWebElement.findElement(By.xpath(".//a[@class='test']"))</code> - another common use case where Page Factory can't help you.<br/>
	</div>
</section>
<br/>

<h3><i>Warning: </i>XPATH Versions</h3>
XPATH (XML Path Language) allows you to use expressions to search inside XML (in our case, HTML). 
It is not inherent or maintained by Selenium, instead, most WebDrivers (ChromeDriver, FirefoxDriver, etc) come bundled with a native XPATH of their own. <br/>
From the looks of it, most, if not all, the WebDrivers today are using XPATH 1.0, so, be careful when searching on the web and finding references to things such as <code>lower-case, upper-case, starts-with</code> 
because they are from newer versions of Xpath and might not work on your scripts. 
You can find a complete reference to XPATH 1.0 from the <strong>World Wide Web Consortium</strong> at <a href='https://www.w3.org/TR/1999/REC-xpath-19991116/'>XPATH 1.0</a>.



Please take a look at my <a href='https://docs.google.com/spreadsheets/d/13a-wUwPx-9BIpa4odchN-mmnEZd2j8KHH9uO_RNBblg/edit?usp=sharing'>XPATH CHEATSHEET</a> to learn more. 

<i>Having a difficult time with a rough element? <strong>Contact me</strong> and I'll help you find it.</i>



Thanks for reading, until the next time,
Santiago.