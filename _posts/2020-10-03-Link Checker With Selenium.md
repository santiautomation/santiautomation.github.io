---
layout: post
title:  "Link Checker With Selenium"
date:   2020-10-03T00:00:52-03:00
author: Santiago L. Schumacher
categories: framework
---

As a Test Automation Engineer, if you're solely dedicated to writing test cases in Selenium you will not be performing to your true potential.
There are many things you could be doing to: <br />

* Help the team achieve excellence in your applications
* Reduce testing time as much as possible
* Help developers gain *confidence* in their changes knowing in a short time if their commits produced unintentional/unexpected alterations  

One of those things will be explained and shared with working code on this Post... *Validating all the links on your Website*.  There are a number of other helpful
Utils we could add to our Test Framework, and I will be discussing them in following posts, so make sure to **Subscribe** scrolling all the way down to the bottom of the page,
so you don't miss them!.

The advantage of using your solution, instead of the many available online, such as the recommended <a href='https://validator.w3.org/checklink'>W3C Link Checker</a>,
is that you will be having everything in one place, which makes for a single entry point to run your tests, analyze links and other tools you can add, and have a single central
Report.  
It will also allow you to navigate to and load a link in your browser with Selenium, which is more than what normal Link Checkers you find online will do (performing only HTTP calls).    
<br />


<h2>What Are Broken Links</h2>  

Broken links happen when your Web Browser can't find the URL you're looking for. In this context, we will one be looking at links *within* a valid website.  
It can happen because:  
* A webpage was moved without updating its references
* The URL structure of a website was changed
* Missing/misconfigured properties files in the deployment

And attempting to navigate to them will send you to ugly 404 sites, and reduce the Website quality in <a href='https://searchengineland.com/guide/what-is-seo'>Search Engine Optimization</a>, which is
why it's important to discover broken links *before* they're deployed in Production.<br /><br />


<h2>Implementing a Simple Link Checker in Your Test Framework</h2>
First, let's be clear on a few things we will need:  
* We want to be able to run it as part of our Automated tests, and we will use our Framework for that
* The report will follow the format: <i>URL that has the broken link -> Broken Link</i>
* We want to go through all the possible URLs within the same Host, for example, if your website contains links to other sites, we shouldn't check their links
* Check the links by doing a HEAD HTTP request instead of a GET to improve performance
* Make sure we only check an URL once, so we need a List of previously checked URLs

I've created a package *linkchecker* inside the *utils* package.  
Let's take a look at a class that will be doing most of the heavy work. Remember you can access the entire code at my <a href='https://github.com/santiautomation/santi-automation-framework'>Github</a>  


{% highlight JAVA %}
package utils.linkchecker;

import java.net.MalformedURLException;
import java.net.URL;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

import javax.swing.text.html.HTML;

import org.apache.commons.validator.routines.UrlValidator;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;

import com.google.common.collect.ArrayListMultimap;
import com.google.common.collect.Multimap;

import seleniumutils.Utilities;

/**
 * Link Validator class, with navigation and recursion to go through all the URLs available from the
 * starting point passed in the constructor. 
 * If the failedLinks variable is empty after finishing then you should pass your test, otherwise print/report all the broken links.
 * 
 * 
 * @author Santiago Schumacher
 *
 */
public class LinkChecker {

	private final WebDriver driver;
	private List<String> linksPass = new ArrayList<>();
	// Multimap is neat class from Google Commons that will allow us to have duplicated keys, which we will need to store the failed links
	// The Keys in our case is the URL which has broken link/s, and the Value is the link that's broken.
	private Multimap<String, String> linksFail = ArrayListMultimap.create();
	private Set<String> scannedLinks = new HashSet<>(); // Set to avoid duplicates.
	private String url;
	private UrlValidator urlValidator = new UrlValidator(); // From Apache Commons. More info at https://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/routines/UrlValidator.html
	private String host;
	
	/**
	 * 
	 * @param driver
	 * @param url - Starting point of the Link Checker
	 * @throws MalformedURLException
	 */
	public LinkChecker(final WebDriver driver, String url) throws MalformedURLException {
		this.driver = driver;
		this.url = url; // Our starting point, the website to be checked.
		this.host = extractHost(url);
	}
	
	/**
	 * Returns the host (String between http/https/www and .com/.net/io etc)
	 * Will throw an Exception if the URL is not valid.43333
	 * @param url
	 * @return
	 * @throws MalformedURLException
	 */
	private String extractHost(String url) throws MalformedURLException {
		URL u = new URL(url);
		return u.getHost();
	}

	// Getters and setters of our variables omitted here for brevity
	
	/**
	 * Entry point of the Link Checker.
	 * 
	 * @throws MalformedURLException
	 */
	public void linkCheckerWithNavigation() throws MalformedURLException {
		this.getScannedLinks().add(driver.getCurrentUrl()); // Start by adding the URL where we are at now to the scanned links.
		System.out.println("Current URL checking: " + driver.getCurrentUrl());
		List<String> allHrefAtr = getLinks(); // Get all the links from the current URL and store them
		
		for (String urlToScan: allHrefAtr) {
	
			boolean status = Utilities.getUrlResponse(urlToScan); // Actual check if the given URL String can be accessed.
			
			if (!status) {
				getLinksFail().put(driver.getCurrentUrl(), urlToScan); // Store data in format fromUrl - failedLink
			} else {
				getLinksPass().add(urlToScan);
			}
		}

		for (String newUrlToScan : getLinksPass()) { // For every valid link inside currently scanned URL
			// Checking if the URL hasn't already been processed, and if it's from the same Host.
			if (!getScannedLinks().contains(newUrlToScan) && newUrlToScan.contains(this.getHost())) { 
				driver.get(newUrlToScan); // Navigate to the next link before starting the process again.
				linkCheckerWithNavigation(); // Recursion. Start validating links again from a different URL of the same host.
			}
		}
	}

	/**
	 * Returns a List of links inside current website. 
	 */
	private List<String> getLinks() {
		List<WebElement> allLinks = driver.findElements(By.tagName(HTML.Tag.A.toString()));
		List<String> stringLinks = new ArrayList<>();
		
		for (WebElement link : allLinks) {
			
			String theLink = link.getAttribute(HTML.Attribute.HREF.toString());
			// Skip repeated URL, URLs already checked, or if its equals to the current URL.
			if (isLinkValid(theLink) && !this.getLinksPass().contains(theLink)) {
				stringLinks.add(theLink);
			}
		}
				
		return stringLinks;
	}

	private boolean isLinkValid(String theLink) {
		return urlValidator.isValid(theLink) && 
				!scannedLinks.contains(theLink)	&& // Make sure URL wasn't already scanned
				!driver.getCurrentUrl().equalsIgnoreCase(theLink) && //This and below links remains on the same URL so we don't navigate to them
				!theLink.contains("collapse") && 
				!theLink.contains("#");
	}

	
}

{% endhighlight %}

<br />
From the above we have almost everything we need in place. The only thing missing is the implementation of <code>Utilities.getUrlResponse(urlToScan)</code>. <br />

Like we mentioned above, it will perform better if the use HEAD HTTP calls to the link instead of calling GET or attempting to navigate, so a nice implementation
of that method looks something like this: <br />

{% highlight JAVA %}
	/**
	 * Given an URL (In String representation) returns true if it's accessible (response codes 200-300), false otherwise.
	 * 
	 * @param url
	 * @return True is the link is valid and returns a 200-300 response code, false otherwise.
	 * @throws MalformedURLException
	 */
	public static boolean getUrlResponse(String url) throws MalformedURLException {
		URL myURL = new URL(url);

		HttpURLConnection conn = null;
		try {
			conn = (HttpURLConnection) myURL.openConnection();
			conn.setRequestMethod(HttpMethod.HEAD.name());

			return conn.getResponseCode() < 400;

		} catch (IOException e) {
			return false;
		} finally {
			if (null != conn) {
				conn.disconnect(); // Always remember to disconnect, we don't want leaked resources !
			}
		}
	}
{% endhighlight %}

<br />

With all that in place, we are ready to start using them in a Test, you could instantiate the <code>LinkChecker</code> class directly in your test, but it's 
probably a better decision to add that in the <code>BasePage</code> instead, where it's available for others Test Automation Engineers if you're working with a team.

Instantiation is quite simple:

{% highlight JAVA %}	
	public Multimap<String, String> checkLinks() throws MalformedURLException {
		LinkChecker lc = new LinkChecker(this.getDriver(), driver.getCurrentUrl());
		lc.linkCheckerWithNavigation();
		
		return lc.getLinksFail();
	}
{% endhighlight %}

And now you have a working link checker available as part of your regression tests ! Keep in mind this is one of the simplest versions available, if you're
feeling adventurous I have a couple improvements in my head that you can implement:
* Add a DEPTH variable so you can alter the max level of recursion
* Include other HTML tags that can be processed with HTTP requests, for example img



That's it for today's post.<br /> 
Thanks for reading, and happy testing,
Santiago.