---
layout: post
title:  "Step 1: Choosing what to Automate"
date:   2020-07-25T14:25:52-05:00
author: Santiago L. Schumacher
categories: Miscellaneous
---

During my career, I've performed multiple interviews to aspiring Test Automation Engineers. They've ranged from IT people such as programmers or manual testers, looking 
to search for a new vocation, to people with many years on the field.

One question that I always enjoy to ask is:

<i>{{ "Which criteria would you choose to prioritize a large list of Test Cases for Automation ?" | smartify }} </i>

The first answers I get are almost always the same one:

 <ul>
  <li>Business-critical test cases</li>
  <li>Highly repetitive tests, such as smokes or sanity tests that needs to be executed after every build</li>
 </ul>

Well.... yes, of course ! Those 2 options should always be your think thought when thinking about Automation candidates.

However, there are other things to be considered.

I want you to keep one thing in mind: One of the main reasons for the role is <i>efficiency</i>. 
<br>
<ensp>Cut down on testing time<br>
<ensp>Saving money<br>
<ensp>Avoiding costly manual oversights or errors<br><br>

In other words we're aiming to maximize the company's <strong> Return of Investment </strong>

The following list provides guidance on how to increase your RoI.
<br>

<ul>
  <li>Tests that are simple, but needs to be executed in multiple configurations, ie: Tested against multiple browsers, Operating Systems, Mobile devices.</li>
  <li>Easely parameterized tests. These includes test cases that must be repeated several times using different inputs, which are fast to develop through Data Providers.</li>
  <li>Test cases that are error prone: This includes high-complexity mathematical formulas or other deterministic operations, that once coded, you'll guarantee they will always work the way you programmed it</li>
  <li>Data setup, environment configuration and other general tasks that are not necessarily Test Cases, but can be leveraged with the use of Programming and Selenium. <i> PS: We will go back to this subject on a different Post later, as there are many important things to talk about here</i></li>
  <li>Other relevant options might show up here that are intrinsic to your particular organization, so, keep your eyes open and be sure to talk to QA and business owners and walk on their shoes for a few days</li>
</ul>

Thanks for reading, until the next time,
Santiago.