---
title: Storage Spaces
teaching: 45
exercises: 0
questions:
- What are the types and roles of DUNE's data volumes? 
- What are the commands and tools to handle data?  
objectives:  
- Understanding the data volumes and their properties
- Displaying volume information (total size, available size, mount point, device location)
- Differentiating the commands to handle data between grid accessible and interactive volumes
keypoints:
- Home directories are centrally managed by Computing Division and meant to store setup scripts, do NOT store certificates here.
- Network attached storage (NAS) /dune/app is primarily for code development.
- The NAS /dune/data is for store ntuples and small datasets.
- dCache volumes (tape, resilient, scratch, persistent) offer large storage with various retention lifetime.
- The tool suites idfh and XRootD allow for accessing data with appropriate transfer method and in a scalable way.
---

## Session Video

<!--The session will be captured on video a placed here after the workshop for asynchronous study.-->
The session was captured for your asynchronous review.

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/lhw4dZv8Yeo" title="DUNE Computing Tutorial May 2022 Storage Systems" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>


## Live Notes

[live Notes](https://docs.google.com/document/d/1dEn_JrGc9bZmzWzXcWzU1taTy41fQxZD19xGEmz_rrs/edit#heading=h.f0jxtlpw51py)

## Introduction
There are three types of storage volumes that you will encounter at Fermilab: local hard drives, network attached storage, and large-scale, distributed storage. Each has it's own advantages and limitations, and knowing which one to use when isn't all straightforward or obvious. But with some amount of foresight, you can avoid some of the common pitfalls that have caught out other users.


## Vocabulary

**What is POSIX?** A volume with POSIX access (Portable Operating System Interface [Wikipedia](https://en.wikipedia.org/wiki/POSIX)) allow users to directly read, write and modify using standard commands, e.g. using bash scripts, fopen(). In general, volumes mounted directly into the operating system.

**What is meant by 'grid accessible'?** Volumes that are grid accessible require specific tool suites to handle data stored there. Grid access to a volume is NOT POSIX access. This will be explained in the following sections.

**What is immutable?** A file that is immutable means that once it is written to the volume it cannot be modified. It can only be read, moved, or deleted. This property is in general a restriction imposed by the storage volume on which the file is stored. Not a good choice for code or other files you want to change.


## Interactive storage volumes

**Home area** is similar to the user's local hard drive but network mounted
* access speed to the volume very high, on top of full POSIX access
* network volumes are NOT safe to store certificates and tickets
* important: users have a single home area at FNAL used for all experiments 
* not accessible from grid worker nodes
* not for code developement (size of less than 2 GB)
* at Fermilab, need a valid Kerberos ticket in order to access files in your Home area
* periodic snapshots are taken so you can recover deleted files. (/nashome/.snapshot)

**Locally mounted volumes** are physical disks, mounted directly on the computer
* physically inside the computer node you are remotely accessing
* mounted on the machine through the motherboard (not over network)
* used as temporary storage for infrastructure services (e.g. /var, /tmp,)
* can be used to store certificates and tickets. (These are saved there automatically with owner-read on and other permissions disabled.)
* usually very small and should not be used to store data files or for code development
* files on these volumes are not backed up

**Network Attached Storage (NAS)** element behaves similar to a locally mounted volume.
* functions similar to services such as Dropbox or OneDrive
* fast and stable POSIX access to these volumes
* volumes available only on a limited number of computers or servers
* not available on larger grid computing (FermiGrid, Open Science Grid, etc.)
* /dune/app has periodic snapshots in /dune/app/.snapshot, but /dune/data and /dune/data2 do NOT

## Grid-accessible storage volumes

At Fermilab, an instance of dCache+Enstore is used for large-scale, distributed storage with capacity for more than 100 PB of storage and O(10000) connections. Whenever possible, these storage elements should be accessed over xrootd (see next section) as the mount points on interactive nodes are slow and unstable. Here are the different dCache volumes:

**Persistent dCache**: the data in the file is actively available for reads at any time and will not be removed until manually deleted by user

**Scratch dCache**: large volume shared across all experiments. When a new file is written to scratch space, old files are removed in order to make room for the newer file. removal is based on LRU policy

**Resilient dCache**: (NOTE: DIRECT USAGE is being phased out) handles custom user code for their grid jobs, often in the form of a tarball. Inappropriate to store any other files here (no data or ntuples). keeps many copies of tarball so storage of data or ntuples has large impact and will quickly create problems

**Tape-backed dCache**: disk based storage areas that have their contents mirrored to permanent storage on Enstore tape.  
Files are not available for immediate read on disk, but needs to be 'staged' from tape first ([see video of a tape storage robot](https://www.youtube.com/watch?v=kiNWOhl00Ao)).


## Summary on storage spaces
Full documentation: [Understanding Storage Volumes](https://cdcvs.fnal.gov/redmine/projects/fife/wiki/Understanding_storage_volumes)

In the following table, \<exp\> stands for the experiment (uboone, nova, dune, etc...)

|-------------+------------------+----------+-------------+----------------+------------+--------------+-----------|
|    | Quota/Space | Retention Policy | Tape Backed? | Retention Lifetime on disk |	Use for	| Path | Grid Accessible |
|-------------+------------------+----------+-------------+----------------+------------+--------------+-----------|
| Persistent dCache	| No/~100 TB/exp | Managed by Experiment| No| Until manually deleted | immutable files w/ long lifetime	| /pnfs/\<exp\>/persistent	| Yes |
|-------------+------------------+----------+-------------+----------------+------------+--------------+-----------|
| Scratch dCache | No/no limit | LRU eviction - least recently used file deleted | No | Varies, ~30 days (*NOT* guaranteed) | immutable files w/ short lifetime | /pnfs/\<exp\>/scratch	| Yes |
|-------------+------------------+----------+-------------+----------------+------------+--------------+-----------|
| Resilient dCache | No/no limit | Periodic eviction if file not accessed | No | Approx 30 days (your experiment may have an active clean up policy) | input tarballs with custom code for grid jobs (do NOT use for grid job outputs) | /pnfs/\<exp\>/resilient | Yes |
|-------------+------------------+----------+-------------+----------------+------------+--------------+-----------|
| Tape backed| dCache	No/O(10) PB | LRU eviction (from disk) | Yes | Approx 30 days | Long-term archive | /pnfs/dune/... | Yes |
|-------------+------------------+----------+-------------+----------------+------------+--------------+-----------|
| NAS Data | Yes (~1 TB)/ 32+30 TB total | Managed by Experiment | No | Until manually deleted | Storing final analysis samples | /dune/data | No |
|-------------+------------------+----------+-------------+----------------+------------+--------------+-----------|
| NAS App | Yes (~100 GB)/ ~15 TB total | Managed by Experiment | No | Until manually deleted | Storing and compiling software | /dune/app | No |
|-------------+------------------+----------+-------------+----------------+------------+--------------+-----------|
| Home Area (NFS mount)	| Yes (~10 GB) | Centrally Managed by CCD | No | Until manually deleted | Storing global environment scripts (All FNAL Exp) | /nashome/\<letter\>/\<uid\>| No |
|-------------+------------------+----------+-------------+----------------+------------+--------------+-----------|

![Storage Picture](../fig/Storage.png){: .image-with-shadow }

## Monitoring and Usage
Remember that these volumes are not infinite, and monitoring your and the experiment usage of these volumes is important to smooth access to data and simulation samples. To see your persistent usage visit [here](https://fifemon.fnal.gov/monitor/d/000000175/dcache-persistent-usage-by-vo?orgId=1&var-VO=dune) (bottom left):

And to see the total volume usage at Rucio Storage Elements around the world:

**Resource** [DUNE Rucio Storage](https://dune.monitoring.edi.scotgrid.ac.uk/app/dashboards#/view/7eb1cea0-ca5e-11ea-b9a5-15b75a959b33?_g=(filters:!(),refreshInterval:(pause:!t,value:0),time:(from:now-1d,to:now)))

## Commands and tools
This section will teach you the main tools and commands to display storage information and access data.

### The df command

To find out what types of volumes are available on a node can be achieved with the command `df`. The `-h` is for _human readable format_. It will list a lot of information about each volume (total size, available size, mount point, device location).
~~~
df -h
~~~
{: .language-bash}

> ## Exercise 1
> From the output of the `df -h` command, identify:
> 1. the home area
> 2. the NAS storage spaces
> 3. the different dCache volumes
{: .challenge}


### ifdh 

Another useful data handling command you will soon come across is ifdh. This stands for Intensity Frontier Data Handling. It is a tool suite that facilitates selecting the appropriate data transfer method from many possibilities while protecting shared resources from overload. You may see *ifdhc*, where *c* refers to *client*.

Here is an example to copy a file. Refer to the [Mission Setup]({{ site.baseurl }}/setup.html) for the setting up the `DUNESW_VERSION`.
~~~
source ~/dune_presetup_202205.sh
dune_setup
kx509
export ROLE=Analysis
voms-proxy-init -rfc -noregen -voms=dune:/dune/Role=$ROLE -valid 120:00
setup ifdhc
ifdh cp root://fndca1.fnal.gov:1094/pnfs/fnal.gov/usr/dune/tape_backed/dunepro/physics/full-reconstructed/2019/mc/out1/PDSPProd2/22/60/37/10/PDSPProd2_protoDUNE_sp_reco_35ms_sce_off_23473772_0_452d9f89-a2a1-4680-ab72-853a3261da5d.root /dev/null
~~~
{: .language-bash}

**Resource:** [idfh commands](https://cdcvs.fnal.gov/redmine/projects/ifdhc/wiki/Ifdh_commands)

> ## Exercise 2
> Using the ifdh command, complete the following tasks:
* create a directory in your dCache scratch area (/pnfs/dune/scratch/users/${USER}/) called "DUNE_tutorial_May2022"
* copy /dune/app/users/${USER}/my_first_login.txt file to that directory.
* copy the my_first_login.txt file from your scrtach directory DUNE_tutorial_May2022 dCache to /dev/null
* remove the directory DUNE_tutorial_May2022 using "ifdh rmdir /pnfs/dune/scratch/users/${USER}/DUNE_tutorial_May2022"
> Note, if the destination for an ifdh cp command is a directory instead of filename with full path, you have to add the "-D" option to the command line. Also, for a directory to be deleted, it must be empty.
{: .challenge}


### xrootd 
The eXtended ROOT daemon is software framework designed for accessing data from various architectures and in a complete scalable way (in size and performance). 

XRootD is most suitable for read-only data access.
[XRootD Man pages](https://xrootd.slac.stanford.edu/docs.html)


Issue the following command. Please look at the input and output of the command, and recognize that this is a listing of /pnfs/dune/scratch/users/${USER}/DUNE_tutorial_May2022. Try and understand how the translation between a NFS path and an xrootd URI could be done by hand if you needed to do so.

~~~
xrdfs root://fndca1.fnal.gov:1094/ ls /pnfs/fnal.gov/usr/dune/scratch/users/${USER}/DUNE_tutorial_May2022
~~~
{: .language-bash}

Note that you can do
~~~
lar -c <xrootd_uri> <input.fcl>
~~~
{: .language-bash}

to stream into a larsoft module configured within the fhicl file. As well, it can be implemented in standalone C++ as

~~~
TFile * thefile = TFile::Open(<xrootd_uri>)
~~~
{: .language-c++}

or PyROOT code as

~~~
thefile = ROOT.TFile.Open(<xrootd_uri>)
~~~
{: .language-python}

## Let's practice

> ## Exercise 3
> Using a combination of `ifdh` and `xrootd` commands discussed previously:
> * Use `ifdh` locateFile to find the directory for this file `PDSPProd4a_protoDUNE_sp_reco_stage1_p1GeV_35ms_sce_off_43352322_0_20210427T162252Z.root`
> * Translate the pnfs path to get an `xrootd` URI for that file.  Hint:  use the duneutil script pnfs2xrootd
> * Use `xrdcp` to copy that file to `/dev/null`
> * Using `xrdfs` and the `ls` option, count the number of files in the same directory as `PDSPProd4a_protoDUNE_sp_reco_stage1_p1GeV_35ms_sce_off_43352322_0_20210427T162252Z.root`
{: .challenge}

Note that redirecting the standard output of a command into the command `wc -l` will count the number of lines in the output text. e.g. `ls -alrth ~/ | wc -l`




## Useful links to bookmark

* [ifdh commands (redmine)](https://cdcvs.fnal.gov/redmine/projects/ifdhc/wiki/Ifdh_commands)
* [Understanding storage volumes (redmine)](https://cdcvs.fnal.gov/redmine/projects/fife/wiki/Understanding_storage_volumes)
* How DUNE storage works: [pdf](https://dune-data.fnal.gov/tutorial/howitworks.pdf)

---

{%include links.md%} 
