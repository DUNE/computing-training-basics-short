---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
venue: "Fermi National Accelerator Laboratory"
address: "online"
country: "us"
language: "en"
latitude: "45"
longitude: "-1"
humandate: "January 26, 2023"
humantime: "8:00 am - 11:30 am"
startdate: "2023-01-26"
enddate: "2023-01-26"
instructor: ["Michael Kirby","Steve Timm","Tom Junk","Ken Herner"]
helper: ["mentor1", "mentor2"]
email: ["mkirby@fnal.gov","timm@fnal.gov","junk@fnal.gov","herner@fnal.gov"]
collaborative_notes: "2023-01-26-dune"
eventbrite:
---

This tutorial will teach you the basics of DUNE Computing. It is split into four parts that you can attend independently (we advise newcomers to follow the whole event though).

Each part will have a little introduction followed by hands-on sessions in breakout rooms. Here mentors will answer your questions and provide technical support.

<!-- this is an html comment -->

{% comment %} This is a comment in Liquid {% endcomment %}

> ## Prerequisites
>
> Command line experience is necessary for this training. We recommend the
> participants to go through
> [The Unix Shell](https://swcarpentry.github.io/shell-novice/), if new to the
> command line (also known as terminal or shell).  
{: .prereq}

By the end of this workshop, participants will know how to:

* Utilize data volumes at FNAL.
* Understand good data management practices.
* Provide a basic overview of art and LArSoft to a new researcher.
* Develop configuration files to control batch jobs.
* Monitor jobs submitted to the grid.

> ## Getting Started
>
> First step: follow the directions in the "[Setup](
> {{ page.root }}/setup.html)" to arrived prepared for this event. Follow the instructions; we give you an easy exercise 
> to make sure you are good to go.
{: .callout}

You will need a valid FNAL or CERN account to be able to do the tutorial and be on the DUNE Collaboration member list. If you do not, contact your team leader.


<h2 id="schedule">Schedule by Day</h2>


{% comment %}
{% include sc/schedule.html %}
<!--<center><img  alt="" src="fig/Schedule_computing_training_202105.png"/></center>-->
<!-- An [asynchronous session]({{site.baseurl}}/asynchronous/) is designed as later day acivities for the first two days of the workshop.-->
{% endcomment %}

{% include links.md %}
