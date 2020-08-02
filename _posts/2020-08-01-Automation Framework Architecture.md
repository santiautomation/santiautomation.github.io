---
layout: post
title:  "Test Automation Framework - The main Components"
date:   2020-07-27T14:25:52-05:00
author: Santiago L. Schumacher
categories: framework
---

<h1>Setting the right building blocks for your Framework</h1>

Hello everybody ! 
In this first post about building a good, robust test automation framework, we'll start talking about some of the most necessary components every Framework should include, much like
building a house, we have to start with the plans and foundations first.

If you're feeling anxious and want to get started coding right away - feel free to jump to my next 'Framework' post, as this one won't have actual code.

<h2> Framework Scope </h2>
The first thing we're doing is deciding on <strong>what</strong> our framework is going to be able to do. Note that I haven't even thought about which technology to use yet.

Play along with my here. 
In this fictional scenario, we've just started working with a new Client, exciting ! They're eager to get started, and a bit apprehensive, as they haven't worked 
with a Test Automation Engineer before !
After some discussion you convince the Client to give you a couple weeks to work on a Framework, as it will save them hundreds of headaches and more importantly maintenance costs down the road.

You sit at your desk after a long meeting, finally ready to get started !
You open your Notebook and find the following notes:
<br>

<ul>
  <li>Most, if not all, their applications backends are coded with Java 
  From here we will pick the same programming language for our Framework as the devs. We could argue that any other language would do, however using the same
  technology as the developers will make it easier to integrate part of their code into yours, reuse components, and share other valuable things.
  In our case we'll be using Java 13, but using any version between 8 and 13 should work with minor tweaking.</li>
  <li>They use React for the UI, with HTML and Bootstrap for the CSS. It's a standard web application. Our answer here is clear, let's use <strong>Selenium</strong>. 
  You google 'Selenium Java latest release' and find in <a>https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java</a> that the stable version is 3.141.59 - let's stick to that one until Selenium 4 is stable enough to use on Production projects.
  </li>
  <li>Following the Global trend, their application is also available for iOS and Android mobile devices.
  This is our next clue, we will need <strong>Appium</strong> integration as well. Let's add that option too.</li>
  <li>Communication is really important for your Client. They mentioned that they spend many hours a day on <strong>Slack</strong>, and no major decisions are
  taken without Emailing and approvals for memo.
  That means our framework will most definitely need to send test results to both: Email and Slack.</li>
  <li>They use BrowserStack so the manual testers are able to test different browsers and mobile deviced on the cloud. We will add integration to that as well.</li>  <-- agregar link -->
  <li>API and Integration tests is a must, as the Test Automation Pyramid tells us. The clear is option is here is adding the <strong>RestAssured</strong> dependency to our list.</li>
  <li>They use Jenkins, so it's important for us to leverage everything that tool provides.</li>
  <li>Reports: The standard reports offered by runners such as TestNG, JUnit, or Maven's Surefire plugin are simply not good enough. They can be difficult to understand, not too 
  easy on the eyes, and most importantly: you can't immediatly tell what went wrong in failing tests! Hmm... we will figure our how to solve this later</li>
</ul>

And just like that ! The day isn't over yet and we already know most of the components, scope and functionality the framework must provide to clear the Client's expectations.. <br>
We're ready to start doing some coding.

Until the next time !