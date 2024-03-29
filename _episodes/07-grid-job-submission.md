---
title: Grid Job Submission and Common Errors
teaching: 30
exercises: 0
questions:
- How to submit grid jobs?
objectives:
- Submit a job and understand what's happening behind the scenes
- Monitor the job and look at its outputs
- Review best practices for submitting jobs (including what NOT to do)
- Extension; submit a small job with POMS
keypoints:
- When in doubt, ask! Understand that policies and procedures that seem annoying, overly complicated, or unnecessary (especially when compared to running an interactive test) are there to ensure efficient operation and scalability. They are also often the result of someone breaking something in the past, or of simpler approaches not scaling well.
- Send test jobs after creating new workflows or making changes to existing ones. If things don't work, don't blindly resubmit and expect things to magically work the next time.
- Only copy what you need in input tar files. In particular, avoid copying log files, .git directories, temporary files, etc. from interactive areas.
- Take care to follow best practices when setting up input and output file locations.
- Always, always, always prestage input datasets. No exceptions.
---

## Video Session 

<!--The session will be captured on video a placed here after the workshop for asynchronous study.-->
The session video from the training in January 2023 is provided here as a reference.

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/3vySzd0dypk" title="DUNE Computing Tutorial January 2023 Grid Job Submission and Common Errors" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

## Submit a job

**Note that job submission requires FNAL account but can be done from a CERN machine, or any other with CVMFS access.**

First, log in to a `dunegpvm` machine (should work from `lxplus` too with a minor extra step of getting a Fermilab Kerberos ticket on `lxplus` via `kinit`). Then you will need to set up the job submission tools (`jobsub`). If you set up `dunetpc` it will be included, but if not, you need to do

```bash
source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
setup jobsub_client
mkdir -p /pnfs/dune/scratch/users/${USER}/DUNE_tutorial_Jan2023 # if you have not done this before
mkdir -p /pnfs/dune/scratch/users/${USER}/jan2023tutorial
```
Having done that, let us submit a prepared script:

~~~
jobsub_submit -G dune -M -N 1 --memory=1000MB --disk=1GB --cpu=1 --expected-lifetime=1h --resource-provides=usage_model=DEDICATED,OPPORTUNISTIC,OFFSITE -l '+SingularityImage=\"/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-wn-sl7:latest\"' --append_condor_requirements='(TARGET.HAS_Singularity==true&&TARGET.HAS_CVMFS_dune_opensciencegrid_org==true&&TARGET.HAS_CVMFS_larsoft_opensciencegrid_org==true&&TARGET.CVMFS_dune_opensciencegrid_org_REVISION>=1105)' -e GFAL_PLUGIN_DIR=/usr/lib64/gfal2-plugins -e GFAL_CONFIG_DIR=/etc/gfal2.d file:///dune/app/users/kherner/submission_test_singularity.sh
~~~
{: .source}

If all goes well you should see something like this:

~~~
/fife/local/scratch/uploads/dune/kherner/2022-05-11_151253.446116_5339
/fife/local/scratch/uploads/dune/kherner/2022-05-11_151253.446116_5339/submission_test_singularity.sh_20220511_151254_1116939_0_1_.cmd
submitting....
Submitting job(s).
1 job(s) submitted to cluster 32496605.
JobsubJobId of first job: 32496605.0@jobsub03.fnal.gov
Use job id 32496605.0@jobsub03.fnal.gov to retrieve output
~~~
{: .output}

> ## Quiz 
>
> 1. What is your job ID?
>
{: .solution}

Now, let's look at some of these options in more detail.

* `-M` sends mail after the job completes whether it was successful for not. The default is email only on error. To disable all emails, use `--mail_never`.  
* `-N` controls the number of identical jobs submitted with each cluster. Also called the process ID, the number ranges from 0 to N-1 and forms the part of the job ID number after the period, e.g. 12345678.N.  
* `--memory, --disk, --cpu, --expected-lifetime` request this much memory, disk, number of cpus, and max run time.  Jobs that exceed the requested amounts will go into a held state. Defaults are 2000 MB, 10 GB, 1, and 8h, respectively. Note that jobs are charged against the DUNE FermiGrid quota according to the greater of memory/2000 MB and number of CPUs, with fractional values possible. For example, a 3000 MB request is charged 1.5 "slots", and 4000 MB would be charged 2. You are charged for the amount **requested**, not what is actually used, so you should not request any more than you actually need (your jobs will also take longer to start the more resources you request). Note also that jobs that run offsite do NOT count against the FermiGrid quota. **In general, aim for memory and run time requests that will cover 90-95% of your jobs and use the [autorelease feature][job-autorelease] to deal with the remainder**.  
* `--resource-provides=usage_model` This controls where jobs are allowed to run. DEDICATED means use the DUNE FermiGrid quota, OPPORTUNISTIC means use idle FermiGrid resources beyond the DUNE quota if they are available, and OFFSITE means use non-Fermilab resources. You can combine them in a comma-separated list. <span style="color:red"> In nearly all cases you should be setting this to DEDICATED,OPPORTUNISTIC,OFFSITE.</span> This ensures maximum resource availability and will get your jobs started the fastest. Note that because of Singularity, **there is absolutely no difference** between the environment on Fermilab worker nodes and any other place. Depending on where your input data are (if any), you might see slight differences in network latency, but that's it.  
* `-l` (or `--lines=`) allows you to pass additional arbitrary HTCondor-style `classad` variables into the job. In this case, we're specifying exactly what `Singularity` image we want to use in the job. It will be automatically set up for us when the job starts. Any other valid HTCondor `classad` is possible. In practice you don't have to do much beyond the `Singularity` image. Here, pay particular attention to the quotes and backslashes.  
* `--append_condor_requirements` allows you to pass additional `HTCondor-style` requirements to your job. This helps ensure that your jobs don't start on a worker node that might be missing something you need (a corrupt or out of date `CVMFS` repository, for example). Some checks run at startup for a variety of `CVMFS` repositories. Here, we check that Singularity invocation is working and that the `CVMFS` repos we need ( [dune.opensciencegrid.org][dune-openscience-grid-org] and [larsoft.opensciencegrid.org][larsoft-openscience-grid-org] ) are in working order. Optionally you can also place version requirements on CVMFS repos (as we did here as an example), useful in case you want to use software that was published very recently and may not have rolled out everywhere yet.
*  `-e VAR=VAL` will set the environment variable VAR to the value VAL inside the job. You can pass this option multiple times for each variable you want to set. You can also just do `-e VAR` and that will set VAR inside the job to be whatever value it's set to in your current environment (make sure it's actually set though!) **One thing to note here as of January 2022 is that these two gfal variables may need to be set as shown to prevent problems with output copyback at a few sites.** It is safe to set these variable to the values shown here in all jobs at all sites, since the locations exist in the default container (assuming you're using that).

## Job Output

This particular test writes a file to `/pnfs/dune/scratch/users/<username>/job_output_<id number>.log`.
Verify that the file exists and is non-zero size after the job completes.
You can delete it after that; it just prints out some information about the environment.

More information about `jobsub` is available [here][redmine-wiki-jobsub] and [here][redmine-wiki-using-the-client].

## Manipulating submitted jobs

If you want to remove existing jobs, you can do

```bash
jobsub_rm -G dune --jobid=12345678.9@jobsub0N.fnal.gov
```

to remove all jobs in a given submission (i.e. if you used -N <some number greater than 1>) you can do

```bash
jobsub_rm -G dune --jobid=12345678@jobsub0N.fnal.gov
```
To remove all of your jobs, you can do
```bash
jobsub_rm -G dune --user=username
```
If you want to manipulate only a certian subset of jobs, you can use a HTCondor-style constraint. For example, if I want to remove only held jobs asking for more than say 8 GB of memory that went held because they went over their request, I could do something like
```bash
jobsub_rm -G dune --constraint='Owner=="username"&&JobStatus==5&&RequestMemory>=8000&&(HoldReasonCode==34||(HoldReasonCode==26&&HoldReasonSubCode==1))'
```
To hold jobs, it's the same procedure as `jobsub_rm`; just replace that with `jobsub_hold`. To release a held job (which will restart from the beginning), it's the same commands as above, only use `jobsub_release` in place of rm or hold.

if you get tired of typing `-G dune` all the time, you can set the JOBSUB_GROUP environment variable to dune, and then omit the -G option.

## Submit a job using the tarball containing custom code

First off, a very important point: for running analysis jobs, **you may not actually need to pass an input tarball**, especially if you are just using code from the base release and you don't actually modify any of it.
All you need to do is set up any required software from CVMFS (e.g. dunetpc and/or protoduneana), and you are ready to go.
If you're just modifying a fcl file, for example, but no code, it's actually more efficient to copy just the fcl(s) your changing to the scratch directory within the job, and edit them as part of your job script (copies of a fcl file in the current working directory have priority over others by default).

Sometimes, though, we need to run some custom code that isn't in a release. 
We need a way to efficiently get code into jobs without overwhelming our data transfer systems.
We have to make a few minor changes to the scripts you made in the previous tutorial section, generate a tarball, and invoke the proper jobsub options to get that into your job.
There are many ways of doing this but by far the best is to use the Rapid Code Distribution Service (RCDS), as shown in our example.  

If you have finished up the LArSoft follow-up and want to use your own code for this next attempt, feel free to tar it up (you don't need anything besides the localProducts* and work directories) and use your own tar ball in lieu of the one in this example.
You will have to change the last line with your own submit file instead of the pre-made one.

First, we should make a tarball. Here is what we can do (assuming you are starting from /dune/app/users/username/):

```bash
cp /dune/app/users/kherner/setupjan2023tutorial-grid.sh /dune/app/users/${USER}/
cp /dune/app/users/kherner/jan2023tutorial/localProducts_larsoft_v09_63_00_e20_prof/setup-grid /dune/app/users/${USER}/jan2023tutorial/localProducts_larsoft_v09_63_00_e20_prof/setup-grid
```

Before we continue, let's examine these files a bit. We will source the first one in our job script, and it will set up the environment for us.

~~~
#!/bin/bash                                                                                                                                                                                                      

DIRECTORY=jan2023tutorial
# we cannot rely on "whoami" in a grid job. We have no idea what the local username will be.
# Use the GRID_USER environment variable instead (set automatically by jobsub). 
USERNAME=${GRID_USER}

source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
export WORKDIR=${_CONDOR_JOB_IWD} # if we use the RCDS the our tarball will be placed in $INPUT_TAR_DIR_LOCAL.
if [ ! -d "$WORKDIR" ]; then
  export WORKDIR=`echo .`
fi

source ${INPUT_TAR_DIR_LOCAL}/${DIRECTORY}/localProducts*/setup-grid 
mrbslp
~~~
{: .source}


Now let's look at the difference between the setup-grid script and the plain setup script.
Assuming you are currently in the /dune/app/users/username directory:

```bash
diff jan2023tutorial/localProducts_larsoft_v09_63_00_e20_prof/setup jan2023tutorial/localProducts_larsoft_v09_63_00_e20_prof/setup-grid
```

~~~
< setenv MRB_TOP "/dune/app/users/<username>/jan2023tutorial"
< setenv MRB_TOP_BUILD "/dune/app/users/<username>/jan2023tutorial"
< setenv MRB_SOURCE "/dune/app/users/<username>/jan2023tutorial/srcs"
< setenv MRB_INSTALL "/dune/app/users/<username>/jan2023tutorial/localProducts_larsoft_v09_63_00_e20_prof"
---
> setenv MRB_TOP "${INPUT_TAR_DIR_LOCAL}/jan2023tutorial"
> setenv MRB_TOP_BUILD "${INPUT_TAR_DIR_LOCAL}/jan2023tutorial"
> setenv MRB_SOURCE "${INPUT_TAR_DIR_LOCAL}/jan2023tutorial/srcs"
> setenv MRB_INSTALL "${INPUT_TAR_DIR_LOCAL}/jan2023tutorial/localProducts_larsoft_v09_63_00_e20_prof"
~~~
{: .output}

As you can see, we have switched from the hard-coded directories to directories defined by environment variables; the `INPUT_TAR_DIR_LOCAL` variable will be set for us (see below).
Now, let's actually create our tar file. Again assuming you are in `/dune/app/users/kherner/jan2023tutorial/`:
```bash
tar --exclude '.git' -czf jan2023tutorial.tar.gz jan2023tutorial/localProducts_larsoft_v09_63_00_e20_prof jan2023tutorial/work setupjan2023tutorial-grid.sh
```
Then submit another job (in the following we keep the same submit file as above):

```bash
jobsub_submit -G dune -M -N 1 --memory=2500MB --disk=2GB --expected-lifetime=3h --cpu=1 --resource-provides=usage_model=DEDICATED,OPPORTUNISTIC,OFFSITE --tar_file_name=dropbox:///dune/app/users/<username>/jan2023tutorial.tar.gz -l '+SingularityImage=\"/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-wn-sl7:latest\"' --append_condor_requirements='(TARGET.HAS_Singularity==true&&TARGET.HAS_CVMFS_dune_opensciencegrid_org==true&&TARGET.HAS_CVMFS_larsoft_opensciencegrid_org==true&&TARGET.CVMFS_dune_opensciencegrid_org_REVISION>=1105&&TARGET.HAS_CVMFS_fifeuser1_opensciencegrid_org==true&&TARGET.HAS_CVMFS_fifeuser2_opensciencegrid_org==true&&TARGET.HAS_CVMFS_fifeuser3_opensciencegrid_org==true&&TARGET.HAS_CVMFS_fifeuser4_opensciencegrid_org==true)' -e GFAL_PLUGIN_DIR=/usr/lib64/gfal2-plugins -e GFAL_CONFIG_DIR=/etc/gfal2.d file:///dune/app/users/kherner/run_jan2023tutorial.sh
```

You'll see this is very similar to the previous case, but there are some new options: 

* `--tar_file_name=dropbox://` automatically copies and untars the given tarball into a directory on the worker node, accessed via the INPUT_TAR_DIR_LOCAL environment variable in the job. As of now, only one such tarball can be specified. If you need to copy additional files into your job that are not in the main tarball you can use the -f option (see the jobsub manual for details). The value of INPUT_TAR_DIR_LOCAL is by default $CONDOR_DIR_INPUT/name_of_tar_file, so if you have a tar file named e.g. jan2023tutorial.tar.gz, it would be $CONDOR_DIR_INPUT/jan2023tutorial.
* Notice that the `--append_condor_requirements` line is longer now, because we also check for the fifeuser[1-4]. opensciencegrid.org CVMFS repositories.  

Now, there's a very small gotcha when using the RCDS, and that is when your job runs, the files in the unzipped tarball are actually placed in your work area as symlinks from the CVMFS version of the file (which is what you want since the whole point is not to have N different copies of everything).
The catch is that if your job script expected to be able to edit one or more of those files within the job, it won't work because the link is to a read-only area.
Fortunately there's a very simple trick you can do in your script before trying to edit any such files: 

~~~
cp ${INPUT_TAR_DIR_LOCAL}/file_I_want_to_edit mytmpfile  # do a cp, not mv
rm ${INPUT_TAR_DIR_LOCAL}file_I_want_to_edit # This really just removes the link
mv mytmpfile file_I_want_to_edit # now it's available as an editable regular file.
~~~
{: .source}

You certainly don't want to do this for every file, but for a handful of small text files this is perfectly acceptable and the overall benefits of copying in code via the RCDS far outweigh this small cost. 
This can get a little complicated when trying to do it for things several directories down, so it's easiest to have such files in the top level of your tar file. 

<!--
## Video Session Part 2 of 2

The session will be captured on video a placed here after the workshop for asynchronous study.
-->

<!--<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/L3dx-NQJvp8" title="DUNE Computing Tutorial May 2021 Day 2 Grid Job Submission and Common Errors Part 2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>-->

## Monitor your jobs
For all links below, log in with your FNAL Services credentials (FNAL email, not Kerberos password).

* What DUNE is doing overall:  
[https://fifemon.fnal.gov/monitor/d/000000053/experiment-batch-details?orgId=1&var-experiment=dune](https://fifemon.fnal.gov/monitor/d/000000053/experiment-batch-details?orgId=1&var-experiment=dune)


* What's going on with only your jobs:   
Remember to change the url with your own username and adjust the time range to cover the region of interest.
[https://fifemon.fnal.gov/monitor/d/000000116/user-batch-details?orgId=1&var-cluster=fifebatch&var-user=kherner](https://fifemon.fnal.gov/monitor/d/000000116/user-batch-details?orgId=1&var-cluster=fifebatch&var-user=kherner)

* Why your jobs are held:  
Remember to choose your username in the upper left.  
[https://fifemon.fnal.gov/monitor/d/000000146/why-are-my-jobs-held?orgId=1](https://fifemon.fnal.gov/monitor/d/000000146/why-are-my-jobs-held?orgId=1)

## View the stdout/stderr of our jobs
Here's the link for the history page of the example job: [link](https://fifemon.fnal.gov/monitor/d/000000115/job-cluster-summary?orgId=1&var-cluster=40351757&var-schedd=jobsub01.fnal.gov&from=1611098894726&to=1611271694726). 

Feel free to sub in the link for your own jobs.

Once there, click "View Sandbox files (job logs)".
In general you want the .out and .err files for stdout and stderr.
The .cmd file can sometimes be useful to see exactly what got passed in to your job.

[Kibana][kibana] can also provide a lot of information.

You can also download the job logs from the command line with jobsub_fetchlog: 

```bash
jobsub_fetchlog --jobid=12345678.0@jobsub0N.fnal.gov --unzipdir=some_appropriately_named_directory
```

That will download them as a tarball and unzip it into the directory specified by the --unzipdir option.
Of course replace 12345678.0@jobsub0N.fnal.gov with your own job ID. 

> ## Quiz 
>
> Download the log of your last submission via jobsub_fetchlog or look it up on the monitoring pages. Then answer the following questions (all should be available in the .out or .err files):
> 1. On what site did your job run?
> 2. How much memory did it use?
> 3. Did it exit abnormally? If so, what was the exit code?
>
{: .solution}

## Brief review of best practices in grid jobs (and a bit on the interactive machines)

* When creating a new workflow or making changes to an existing one, <span style="color:red">**ALWAYS test with a single job first**</span>. Then go up to 10, etc. Don't submit thousands of jobs immediately and expect things to work.  
* **ALWAYS** be sure to prestage your input datasets before launching large sets of jobs. This may become less necesaary in the future 
* **Use RCDS**; do not copy tarballs from places like scratch dCache. There's a finite amount of transfer bandwidth available from each dCache pool. If you absolutely cannot use RCDS for a given file, it's better to put it in resilient (but be sure to remove it when you're done!). The same goes for copying files from within your own job script: if you have a large number of jobs looking for a same file, get it from resilient. Remove the copy when no longer needed. Files in resilient dCache that go unaccessed for 45 days are automatically removed.  
* Be careful about placing your output files. **NEVER place more than a few thousand files into any one directory inside dCache. That goes for all type of dCache (scratch, persistent, resilient, etc).**
* **Avoid** commands like `ifdh ls /path/with/wildcards/*/` inside grid jobs. That is a VERY expensive operation and can cause a lot of pain for many users.  
* Use xrootd when opening files interactively; this is much more stable than simply doing `root /pnfs/dune/...`
* **NEVER** copy job outputs to a directory in resilient dCache. Remember that they are replicated by a factor of 20! **Any such files are subject to deletion without warning**.  
* **NEVER** do hadd on files in `/pnfs` areas unless you're using `xrootd`. I.e. do NOT do hadd out.root `/pnfs/dune/file1 /pnfs/dune/file2 ...` This can cause severe performance degradations.  


**Side note:** Some people will pass file lists to their jobs instead of using a SAM dataset. We do not recommend that for two reasons: 1) Lists do not protect you from cases where files fall out of cache at the location(s) in your list. When that happens your jobs sit idle waiting for the files to be fetched from tape, which kills your efficiency and blocks resources for others. 2) You miss out on cases where there might be a local copy of the file at the site you're running on, or at least at closer one to your list. So you may end up unecessarily streaming across oceans, whereas using SAM (or later Rucio) will find you closer, local copies when they exist.

**Another important side note:** If you are used to using other programs for your work such as project.py (which is **NOT** officially supported by DUNE or the Fermilab Scientific Computing Division), there is a helpful tool called [Project-py][project-py-guide] that you can use to convert existing xml into POMS configs, so you don't need to start from scratch! Then you can just switch to using POMS from that point forward. As a reminder, if you use unsupported tools, you are own your own and will receive NO SUPPORT WHATSOEVER. You are still responsible for making sure that your jobs satisfy Fermilab's policy for job efficiency: https://cd-docdb.fnal.gov/cgi-bin/sso/RetrieveFile?docid=7045&filename=FIFE_User_activity_mitigation_policy_20200625.pdf&version=1


## The Future is Now(-ish): jobsub_lite

The existing jobsub consist of both a server and a client product (what users see). In order to simplify development and maintenance a new product call *jobsub_lite* is now available. There is no need for a separate server with jobsub_lite as the client talks directly to HTCondor schedulers. The client has nearly a 100% command name and feature overlap with the existing jobsub_client (sometimes hereafter called "legacy jobsub") and generally requires little to no modification of existing submission scripts. It is currently available for testing on dunegpvm14 and dunegpvm15 and will be on the rest of the gpvm machines by early February, but is already available on all machines via CVMFS.

The official jobsub_lite documentation page is here: [https://fifewiki.fnal.gov/wiki/Jobsub_Lite](https://fifewiki.fnal.gov/wiki/Jobsub_Lite)

**Note: jobsub_lite will be the default product, and legacy jobsub will no longer work, by the next collaboration meeting in May.** 

The biggest change behind the scenes other than getting rid of the server side of things is the switch to using tokens instead of x509 proxies for authenitcation. This is nearly 100% transparent in reality; the jobs all work the same way, and fresh tokens are automatically pushed to jobs much like proxies are now. In reality, there's nothing to worry about with tokens after you do the first submission (see below).

Since the changeover will be happening gradually over the next few months, now is a good time to gain familiarity with it and test your workflows. Let's try the same things we did before. If you're logged in to dunegpvm14 or dunegpvm15 with no DUNE software set up, things will already work. If you're on another machine and/or have done some UPS setup steps, then you just need to do

```bash
setup jobsub_client v_lite
```

Let's try a submission very similar to the first one we did:

```bash
jobsub_submit -G dune --mail_always -N 1 --memory=2500MB --disk=2GB --expected-lifetime=3h --cpu=1 -l '+SingularityImage="/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-wn-sl7:latest"' --append_condor_requirements='(TARGET.HAS_Singularity==true&&TARGET.HAS_CVMFS_dune_opensciencegrid_org==true&&TARGET.HAS_CVMFS_larsoft_opensciencegrid_org==true&&TARGET.CVMFS_dune_opensciencegrid_org_REVISION>=1105&&TARGET.HAS_CVMFS_fifeuser1_opensciencegrid_org==true&&TARGET.HAS_CVMFS_fifeuser2_opensciencegrid_org==true&&TARGET.HAS_CVMFS_fifeuser3_opensciencegrid_org==true&&TARGET.HAS_CVMFS_fifeuser4_opensciencegrid_org==true)' -e GFAL_PLUGIN_DIR=/usr/lib64/gfal2-plugins -e GFAL_CONFIG_DIR=/etc/gfal2.d file:///nashome/k/kherner/submission_test_singularity.sh
```
The first time you try to do a submission you will probably get some output like this:
~~~
ttempting OIDC authentication with https://htvaultprod.fnal.gov:8200

Complete the authentication at:
    https://cilogon.org/device/?user_code=ABC-D1E-FGH
No web open command defined, please copy/paste the above to any web browser
Waiting for response in web browser
~~~

In this particular case, you do want to follow the instructions and copy and paste the link into your browser (can be any browser). There is a time limit on it so its best to do it right away. Always choose Fermilab as the identity provider in the menu, even if your home institution is listed. After you hit log on, you'll get a message saying you approved the access request, and then after a short delay (may be several seconds) in the terminal you will see

~~~
Saving credkey to /nashome/u/username/.config/htgettoken/credkey-dune-default
Saving refresh token ... done
Attempting to get token from https://htvaultprod.fnal.gov:8200 ... succeeded
Storing bearer token in /tmp/bt_token_dune_Analysis_number.othernumber
Storing condor credentials for dune
Submitting job(s)
.
1 job(s) submitted to cluster 57110235.
~~~

And that's it! Then you can use *jobsub_rm*, *jobsub_fetchlog*, etc., as you normally would. **Note you can't see or manipulate jobs submitted with jobsub_lite with legacy jobsub, and vice versa.** The refresh tokens are good for 30 days, so as long as you do something that uses a token at least once every 30 days, you won't have to go through the browser request approval again. If you don't, then you'll just get the same message as earlier and you paste the link into a browser as we did. One other thing to note there is that if you often submit via scripts or other automation, make sure you have a valid refresh token before you kick the scripts off. You could just submit a single dummy job by hand and then immediately remove it, for example.

Let's consider a few differences betwene this submission at the first one:

* `-M` is replaced with `--mail_always`, though it works the same way.
* `--resource-provides=...` is no longer needed, though you can still give it without breaking anything. There are simplified options available for controlling where jobs can run; see the [documentation](https://fifewiki.fnal.gov/wiki/Differences_between_jobsub_lite_and_legacy_jobsub_client/server#--site.2F--onsite.2F--offsite) for details. The default is still to run on all available resources, which is almost always what you want anyway.
* It wasn't necessary to escape the double quotes in `-l '+SingularityImage="/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-wn-sl7:latest"'`. You can keep the escapes in any existing submission scripts though, and it will still work properly.

Now let's submit our job with an input tarball:

```bash
jobsub_submit -G dune --mail_always -N 1 --memory=2500MB --disk=2GB --expected-lifetime=3h --cpu=1 --resource-provides=usage_model=DEDICATED,OPPORTUNISTIC,OFFSITE --tar_file_name=dropbox:///dune/app/users/kherner/jan2023tutorial.tar.gz -l '+SingularityImage="/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-wn-sl7:latest"' --append_condor_requirements='(TARGET.HAS_Singularity==true&&TARGET.HAS_CVMFS_dune_opensciencegrid_org==true&&TARGET.HAS_CVMFS_larsoft_opensciencegrid_org==true&&TARGET.CVMFS_dune_opensciencegrid_org_REVISION>=1105&&TARGET.HAS_CVMFS_fifeuser1_opensciencegrid_org==true&&TARGET.HAS_CVMFS_fifeuser2_opensciencegrid_org==true&&TARGET.HAS_CVMFS_fifeuser3_opensciencegrid_org==true&&TARGET.HAS_CVMFS_fifeuser4_opensciencegrid_org==true)' -e GFAL_PLUGIN_DIR=/usr/lib64/gfal2-plugins -e GFAL_CONFIG_DIR=/etc/gfal2.d file:///dune/app/users/kherner/run_jan2023tutorial.sh
```

~~~
Attempting to get token from https://htvaultprod.fnal.gov:8200 ... succeeded
Storing bearer token in /tmp/bt_token_dune_Analysis_11469
Using bearer token located at /tmp/bt_token_dune_Analysis_11469 to authenticate to RCDS
Checking to see if uploaded file is published on RCDS.
Could not locate uploaded file on RCDS.  Will retry in 30 seconds.
Found uploaded file on RCDS.
Submitting job(s).
1 job(s) submitted to cluster 57110231.
Use job id 57110231.0@jobsub01.fnal.gov to retrieve output
~~~

Note that it did not prompt for another token since it already had one. If you're uploading a new (or modified) tar file, there may be a short delay before the submission finishes because it (now correctly) waits until the tar file is on the RCDS publishing machine.

### Some known issues and things to be aware of, January 2023

Since jobsub_lite isn't officially in production yet, there are still a few things that may slightly change, don't quite work as advertised yet, or are slightly different from how legacy jobsub worked. Here is a non-exhaustive list:

* Emails on job completion aren't working yet.
* jobsub_lite jobs (and logs) aren't visible in FIFEMON yet, but they will be.
* Output directories must be group-writable for copyback to work. Just doing `chmod g+w mydirectory` is enough. That should be fixed in February 2023 during a scheduled dCache downtime.
* Multiple `--tar_file_name` options are now supported (and will be unpacked) if you need things in multiple tarballs.
* The `-f` behavior with and without dropbox:// in front is slightly different from legacy jobsub; see the [documentation](https://fifewiki.fnal.gov/wiki/Differences_between_jobsub_lite_and_legacy_jobsub_client/server#Bug_with_-f_dropbox:.2F.2F.2Fa.2Fb.2Fc.tar) for details.
* Older versions of IFDH will not support tokens, so be careful if you're intentionally setting up old versions. Everything now current is fine though.
* For normal analysis use, you will only be able to copy back directly to your user directory in scratch dCache and possibly some other SEs as remote sites decide to allow.
* Jobsub_lite submission does not work from lxplus. Ways to submit from outside Fermilab are still under development.

Quite a bit of extra information is included in the "Futher Reading" section. See the top for jobsub_lite information.

## Further Reading
Some more background material on these topics (including some examples of why certain things are bad) is in these links:


[December 2022 jobsub_lite demo and information session](https://indico.fnal.gov/event/57514/)

[January 2023 additional experiment feedback session on jobsub_lite]( )

[Wiki page listing differences between jobsub_lite and legacy jobsub](https://fifewiki.fnal.gov/wiki/Differences_between_jobsub_lite_and_legacy_jobsub_client/server)

[DUNE Computing Tutorial:Advanced topics and best practices](DUNE_computing_tutorial_advanced_topics_20210129)

[2021 Intensity Frontier Summer School](https://indico.fnal.gov/event/49414)

[The Glidein-based Workflow Management System]( https://glideinwms.fnal.gov/doc.prd/index.html )

[Introduction to Docker](https://hsf-training.github.io/hsf-training-docker/index.html)

[job-autorelease]: https://cdcvs.fnal.gov/redmine/projects/fife/wiki/Job_autorelease

[redmine-wiki-jobsub]: https://cdcvs.fnal.gov/redmine/projects/jobsub/wiki

[redmine-wiki-using-the-client]: https://cdcvs.fnal.gov/redmine/projects/jobsub/wiki/Using_the_Client

[fifemon-dune]: https://fifemon.fnal.gov/monitor/d/000000053/experiment-batch-details?orgId=1&var-experiment=dune

[fifemon-userjobs]: https://fifemon.fnal.gov/monitor/d/000000116/user-batch-details?orgId=1&var-cluster=fifebatch&var-user=kherner

[fifemon-whyheld]: https://fifemon.fnal.gov/monitor/d/000000146/why-are-my-jobs-held?orgId=1

[kibana]: https://fifemon.fnal.gov/kibana/goto/8f432d2e4a40cbf81d3072d9c9d688a6

[poms-page-ana]: https://pomsgpvm01.fnal.gov/poms/index/dune/analysis/

[poms-user-doc]: https://cdcvs.fnal.gov/redmine/projects/prod_mgmt_db/wiki/POMS_User_Documentation

[fife-launch-ref]: https://cdcvs.fnal.gov/redmine/projects/fife_utils/wiki/Fife_launch_Reference 

[poms-campaign-stage-info]: https://pomsgpvm01.fnal.gov/poms/campaign_stage_info/dune/analysis?campaign_stage_id=9023

[project-py-guide]: https://cdcvs.fnal.gov/redmine/projects/project-py/wiki/Project-py_guide

[DUNE_computing_tutorial_advanced_topics_20200129]: https://indico.fnal.gov/event/20144/contributions/55932/attachments/34945/42690/DUNE_computing_tutorial_advanced_topics_and_best_practices_20200129.pdf  


{%include links.md%}
