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

<!--The session will be captured on video a placed here after the workshop for asynchronous study.-->
The session was captured for your asynchronous review.

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/QyKM8lVR85U" title="DUNE Computing Tutorial May 2022 Code-makeover - Submit with POMS" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

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

Suppose you want to run something with a minor modification to a fcl file. Why bother with downloading and building a custom release and shipping that to your jobs when you don't have any new code or libraries? A much simpler way is to make a local copy of the .fcl file in job and then edit the local copy as needed.

[https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=14110](https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=14110)

## Run a workflow with multiple executable sections (and chained inputs/outputs)

In this example we run a reco2 step followed by a michelremoving step in the same job. The output of reco2 serves as input to michelremoving. Here we only do one input file per job to preserve the 1:1 correspondence between reco1 and reco2 files (nice in this case but not necessary all the time). We see there are multiple executable stages and multiple job_output sections since we save both outputs and actually write them to different directories. REMEMBER: never put more than a few thousand files in any one directory in dCache. If you have a large campaign with many jobs, you may need to split your outputs appropriately with mutliple directories. The only exception to that is if you're copying to a dropbox location for transfer to a tape-backed area, since those will be automatically cleaned up after transfer.

You'll see we also retained the fcl editing bit and a metadata extractor for the reco2 file.

[https://pomsgpvm01.fnal.gov/poms/show_campaign_stages/dune/analysis?campaign_name=kherner-May2022Tutorial-multi-exec](https://pomsgpvm01.fnal.gov/poms/show_campaign_stages/dune/analysis?campaign_name=kherner-May2022Tutorial-multi-exec)

## Create a multi-stage workflow

Sometimes you may want to break separate components into multiple stages. We have the same components as the mutiple executable example but we split the reco2 and michelremoving since the michelremoving files are very small and we want to merge many to one (not we set n_files_per_job much higher in the michel stage). We could of course do things the same way as before but would have to manually merge later. In particular note the [stage_X] sections of the POMS config file. For each stage section you have have separate overrides and still use the same config file. We also use the add_to_dataset option for job outputs and give it the `_poms_task` special keyword; POMS will take care of automatically generating a SMA dataset consisting of the reco2 outputs, which will serve as the input dataset to the next stage.

[https://pomsgpvm01.fnal.gov/poms/show_campaign_stages/dune/analysis?campaign_name=kherner-May2022Tutorial-multi-stage](https://pomsgpvm01.fnal.gov/poms/show_campaign_stages/dune/analysis?campaign_name=kherner-May2022Tutorial-multi-stage)

## Iterate over multiple datasets or list items

Datasets do not have to be actual SAM dataset; they can really be any value or even a list of values. Here we set up a list and cycle through it. Each time you submit, POMS automatically increments the list position until you reach the end of the list.

[https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=14113](https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=14113)

## Draining dataset

If you have very large datasets, or datasets that change over time (such as in keepup processing), you may want to submit only a part of it at once. Of course then you need to keep trakc of what has already been submitted from the main dataset each time. Fortunately POMS provides such functionality with the draining and drainingn dataset types. Our example uses the latter, which will carve out a dataset of at most n files at a time from the larger dataset, keeping track of which files have already been submitted to ensure no duplication between different submissions. We will use the same MC example dataset as we have been using but will now submit it in several pieces.

[https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=14108](https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=14108)

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


