---
title: Code-makeover - Submit with POMS
teaching: 30
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
The session was video captured for your asynchronous review. The video from the full two day training in May 2022 is provided here as a reference.

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/QyKM8lVR85U" title="DUNE Computing Tutorial May 2022 Code-makeover - Submit with POMS" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

# Submitting with POMS, Part I

POMS is the recommended (i.e., supported) way of submitting large workflows. It offers several advantages over other systems, such as 

* Fully configurable. Any executables can be run, not necessarily only lar or art
* Automatic monitoring and campaign management options
* Multi-stage workflow dependencies, automatic dataset creation between stages
* Automated recovery options

At its core, in POMS one makes a "campaign", which has one or more "stages". In our example there is only a single stage.  

For analysis use: [main POMS page][poms-page-ana]  
An [example campaign](https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=14102).

Typical POMS use centers around a configuration file (often more like a template which can be reused for many campaigns) and various campaign-specific settings for overriding the defaults in the config file.
An example config file designed to do more or less what we did in the previous submission is here: `/dune/app/users/kherner/jan2023tutorial/work/pomsdemo.cfg`

You can find more about POMS here: [POMS User Documentation][poms-user-doc]  
Helpful ideas for structuring your config files are here: [Fife launch Reference][fife-launch-ref]  

When you start using POMS you must upload an x509 proxy to the sever before submitting and you need to periodically repeat this as the proxy gets close to expoiration (typically once every few days). If uploading it manually, it must be named x509up_voms_dune_Analysis_yourusername when you upload it.
To upload, look for the User Data item in the left-hand menu on the POMS site, choose Uploaded Files, and follow the instructions. 

By far the easiest way to upload the proxy, however, is to use the `upload_file` command from the *fife_utils* package. To do that, simply first set up fife_utils if not already done:

```bash
setup -g analysis fife_utils
```
And then run the upload file command:
```bash
upload_file --experiment=dune --proxy
```

Typical output will look something like
~~~
Fetching options from https://fifebatch.fnal.gov/cigetcertopts.txt
Checking if /tmp/x509up_uNNNNN has at least 1.0 hours left
117.19 hours remaining, enough to reuse
Checking if myproxy.fnal.gov has at least 503.0 hours left
663.62 hours remaining, enough to reuse
uploaded: /tmp/x509up_voms_dune_Analysis_kherner to POMS server
~~~

where NNNNN will be your UID. As you see, the utility will automatically upload a proxy and give it the proper name for you.

Here is an example of a campaign that does the same thing as the previous one, using some MC reco files from ProtoDUNE SP Prod4a, but does it via making a SAM dataset using that as the input: [POMS campaign stage information](https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=14101). 
Of course, before running **any** SAM project, we should prestage our input definition(s). The way most people do that is to do

```bash
samweb prestage-dataset kherner-jan2023tutorial-mc
```
{: .source}

replacing the above definition with your own definition as appropriate. However, this does NOT reset the clock on the LRU algorithm, because if prestage-dataset sees a file is already chached, it goes on to the next one; it does no lifetime or last access checking, nor does it read the file. A better way to prestage is to instead do

```bash
unsetup curl # necessary as of Jan 2023 because there's an odd interaction with the UPS version of curl, so we need to turn it off
samweb run-project --defname=kherner-jan2023tutorial-mc --schema https 'echo %fileurl && curl -L --cert $X509_USER_PROXY --key $X509_USER_PROXY --cacert $X509_USER_PROXY --capath /etc/grid-security/certificates -H "Range: bytes=0-3" %fileurl && echo'
```

This reads the first four bytes of each file, which will reset the LRU clock. Note you will need to have the X509_USER_PROXY environment variable set. Most of the time that will simply be set as

```bash
export X509_USER_PROXY=/tmp/x509up_u$(id -u)
```

if you ever find yourself doing work under a shared account (dunepro for example) you should *NOT* manually set X509_USER_PROXY in this way.

### Side note: other systems such as the workflow requests system are also coming online; initially they'll be used for managing larger productions. If you have a large production coming up, we encourage you to make a request to the Production group to have it done centrally.


# Submit with POMS Part II: More complicated workflows

In our first example workflow there was only a single stage. Of course in reality you may need to run multiple steps of a workflow in a single job, or run multiple stages in separate jobs, or modify things from a default release, or iterate over multiple config sets, or any number of other arbitrarily complicated ideas. You also have to consider things like creating metadata for your output files and making sure they get to their destination. In this lesson we'll look at some of these more realistic cases and show how they can be set up with POMS campaigns.

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


