---
layout: post
title: Automated accessibility testing
categories: [test automation, accessibility]
tags: [test automation, accessibility]
bigimg: /img/smyrna.jpg
---
#### Ok, so you want to test your website for compliance with Section 508c.  Where do you start?

Thankfully, you don’t have to shell out big bucks to get a report that tells you about your website’s 508C compliance. Open Source FTW! Enter [pa11y](http://pa11y.org/)


Lets do the (short) setup and get started with a quick example.

* Clone the pa11y repo.

 > git clone https://github.com/pa11y/pa11y.git

* Run the pa11y command line tool against your website, eg www.bbc.com

 > pa11y http://www.bbc.com/

* And you’ll see a bunch of results! The output looks like this

 > • Error: Iframe element requires a non-empty title attribute that identifies the frame.
  ├── WCAG2AA.Principle2.Guideline2_4.2_4_1.H64.1
  ├── #wwhp > iframe:nth-child(24)
  └── <iframe
src=“http://tpc.googlesyndication.com/safeframe/1-0-6/html/container.html”
style=“visibility: hidden; display: none;”></iframe>
 
 > • Error: Iframe element requires a non-empty title attribute that identifies the frame.
  ├── WCAG2AA.Principle2.Guideline2_4.2_4_1.H64.1
  ├── #kx-proxy-JA8mItOH
  └── <iframe id=“kx-proxy-JA8mItOH”
src=“http://cdn.krxd.net/partnerjs/xdi/proxy.3d2100fd7107262ecb55ce6847f01fa5.html#!kxcid=JA8mItOH&amp;kxt=http%3A%2F%2Fwww.bbc.com&amp;kxcl=cdn&amp;kxp=”
style=“display: none; visibility: hidden; height: 0; width: 0;”>…

 > 22 Errors
159 Warnings
415 Notices

* If you want a more readable format (of course you do), you’d use the
html report option or the csv/json options. Here’s a sample with the
html report.

 > pa11y http://www.bbc.com/ –reporter html > /tmp/report.html

* Then when you open up report.html, you see content that looks like this.
Way better right? 
![placeholder](/assets/pa11y_html_report.png)

* What next? Fix those issues :) And after that? Setup a dashboard that
runs these reports for you, against websites of your choosing and shows
you those results. Wow, once again, Open Source FTW!!!

* How to get the dashboard going? Start by cloning this repo,

 > git clone https://github.com/pa11y/dashboard

* Install the dependencies, notably MongoDB and node. Then run the below
command,

 > PORT=8080 node index.js

* Now you should be able to hit localhost:8080 and see the dashboard. In
the dashboard, you can easily add the URL you want to hit and you’re
done! Here’s what it looks like on my dashboard.
![placeholder](/assets/pa11y_dashboard_setup.png)
![placeholder](/assets/pa11y_dashboard.png)

* and finally,
![placeholder](/assets/pa11y_sample_dashboard.png)

Is that the easiest accessibility testing you’ve ever done?
