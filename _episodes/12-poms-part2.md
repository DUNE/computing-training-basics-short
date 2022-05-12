---
title: Code-makeover - Submit with POMS
teaching: 60
exercises: 0
questions:
- How to submit realistic grid jobs with POMS?
objectives:  
- Demonstrate use of POMS for job submission with more complicated setups.
keypoints:
- Always, always, always prestage input datasets. No exceptions.
---

## Video Session

The session will be captured on video a placed here after the workshop for asynchronous study.

<!--<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/x6AkusPlrqE" title="DUNE Computing Tutorial May 2021 Day 3 Code-makover - Submit with POMS" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/NTuJa-2TuuQ" title="DUNE Computing Tutorial May 2021 Day 3 Code-makover - Submit with POMS" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

</center>-->

# Submit with POMS Part II: More complicated workflows

This lesson extends from earlier work: [Grid Job Submission and Common Errors]({{ site.baseurl }}/07-grid-job-submission/index.html) 

POMS is the recommended way of submitting large workflows. It offers several advantages over other systems, such as 

* Fully configurable. Any executables can be run, not necessarily only lar or art
* Automatic monitoring and campaign management options
* Multi-stage workflow dependencies, automatic dataset creation between stages
* Automated recovery options

At its core, in POMS one makes a "campaign", which has one or more "stages". In our first example there was only a single stage.  

For analysis use: [main POMS page][poms-page-ana]  
An [example campaign](https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=14101).

Of course in reality you may need to run multiple steps of a workflow in a single job, or run multiple stages in separate jobs, or modify things from a default release, or iterate over multiple config sets, or any number of other arbitrarily complicated ideas. You also have to consider things like SAM metadata for your files. In this lesson we'll look at some of these more realistic cases and show how they can be set up with POMS campaigns.


## Submit work with a modified .fcl file

Suppose you want to run something with a minor modification to a fcl file. Why bother with downloading and building a custom release and shipping that to your jobs when you don't have any new code or libraries? A much simpler way 

## Run a workflow with multiple executable sections (and chained inputs/outputs)


## Create a multi-stage workflow

## Iterate over multiple datasets or list items



{%include links.md%} 

[job-autorelease]: https://cdcvs.fnal.gov/redmine/projects/fife/wiki/Job_autorelease
[dune-openscience-grid-org]: dune.opensciencegrid.org
[larsoft-openscience-grid-org]: larsoft.opensciencegrid.org
[redmine-wiki-jobsub]: https://cdcvs.fnal.gov/redmine/projects/jobsub/wiki
[redmine-wiki-using-the-client]: https://cdcvs.fnal.gov/redmine/projects/jobsub/wiki/Using_the_Client

[fifemon-dune]: https://fifemon.fnal.gov/monitor/d/000000053/experiment-batch-details?orgId=1&var-experiment=dune
[fifemon-myjobs]: https://fifemon.fnal.gov/monitor/d/000000116/user-batch-details?orgId=1&var-cluster=fifebatch&var-user=kherner
[fifemon-whyheld]: https://fifemon.fnal.gov/monitor/d/000000146/why-are-my-jobs-held?orgId=1
[kibana]: https://fifemon.fnal.gov/kibana/goto/8f432d2e4a40cbf81d3072d9c9d688a6
[poms-page-ana]: https://pomsgpvm01.fnal.gov/poms/index/dune/analysis/
[poms-user-doc]: https://cdcvs.fnal.gov/redmine/projects/prod_mgmt_db/wiki/POMS_User_Documentation
[fife-launch-ref]: https://cdcvs.fnal.gov/redmine/projects/fife_utils/wiki/Fife_launch_Reference 
[poms-campaign-stage-info]: https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=9023
[project-py-guide]: https://cdcvs.fnal.gov/redmine/projects/project-py/wiki/Project-py_guide
[DUNE_computing_tutorial_advanced_topics_20210129]: https://indico.fnal.gov/event/20144/contributions/55932/attachments/34945/42690/DUNE_computing_tutorial_advanced_topics_and_best_practices_20200129.pdf


