paceofchange
============

[!(https://zenodo.org/badge/19804/tedunderwood/paceofchange.svg)](https://zenodo.org/badge/latestdoi/19804/tedunderwood/paceofchange)
Code, data, and metadata to support replication of the results in ["How quickly do literary standards change?"](http://figshare.com/articles/How_Quickly_Do_Literary_Standards_Change_/1418394)

If you have Python 3 with numpy, pandas and scikit-learn, you should be able to clone this repo and then run replicate.py. Different forms of analysis can be provided to replicate.py as command-line arguments. For instance,

> python3 replicate.py full

Replicates the main model represented in Figure 1 of the article. Other options include *quarters*, *halves*, *canon,* *genders*, and *nations.*

The standards we are trying to meet are pithily summarized by Jason Baldridge in ["It's okay for academic software to suck."](https://bcomposes.wordpress.com/2015/05/07/its-okay-for-academic-software-to-suck/) We want our results to be reliable, so we've worked hard to ensure that the models produced by this code are cross-validated appropriately. We want the enclosed data and metadata to be reliable -- which means, on this scale, a low level of evenly-distributed error. So we've documented our procedures and shared metadata in ["Understanding Genre in a Collection of a Million Volumes."](http://figshare.com/articles/Understanding_Genre_in_a_Collection_of_a_Million_Volumes_Interim_Report/1281251)

But this is not a project that promised to build portable tools, and you won't find any in this repo. Our view is that the really portable tools are already encapsulated in scikit-learn.

code
----

The important modules are

- **replicate.py**
- **parallel_crossvalidate.py**
- **metafilter.py**
- **modelingprocess.py**

**Replicate.py** translates a command-line option (like *halves*) into a set of parameters which it passes to **parallel_crossvalidate.py.** This module loads data, calls **metafilter** to load metadata, and figures out which volumes have to be excluded from each crossvalidation pass. Then it starts a multiprocessing pool, which calls **modelingprocess** to create each model in a leave-one-out crossvalidation scheme.

The most fragile part of this is probably the multiprocessing. We ran this so often that we needed to parallelize the process, but if that's not necessary you could set processes=1 in line 369 of parallel_crossvalidate.py. Or, if you have more than 4 cores available, you could increase the setting.

metadata
--------

The metadata we used is included as poemeta.csv. Most of the columns are self-explanatory. "actualdate" and "inferreddate" are vestiges of an older process; the models we finally produced only use "firstpub" -- the date of first publication, as best we could determine. "enumcron" contains volume numbers derived from HathiTrust.

There are also five columns related to the original review. *Pubname* identifies the venue, *pubrev* reports whether this author was published or reviewed in the venue (it's usually reviewed), *judge* indicates whether the review was positive or negative (when that's clear), *impaud* is short for implied audience and is only filled out if the review clearly implied that the author was addressing a popular or elite audience. *yrrev* is the date of the review; you will find some works published after the date of review, because we selected works by the same author in cases where we couldn't find a copy of the specific work reviewed.

data
----

The folder /poems contains word counts for the volumes modeled here; intellectual property agreements prevent us from posting the original texts. We have normalized spelling to British practice, truncated apostrophe-s, and trimmed front and back matter, as well as prose introductions, using the algorithmic methods described in ["Understanding Genre in a Collection of a Million Volumes."](http://figshare.com/articles/Understanding_Genre_in_a_Collection_of_a_Million_Volumes_Interim_Report/1281251)

results
-------

The subfolder /results contains predictions and coefficients generated by the modeling process.

In the predictions file, the column "pubdate" is date of first publication. "Logistic" is the model's prediction about likelihood the volume came from the reviewed sample. "Reviewed" tells you  which sample the volume came from. In this prediction file it will be equivalent to the "realclass" column, but if you run replication on nations or genders, the realclass variable will be different than the reviewed column, because you'll be telling the model to predict nationality or gender instead.

The coefficients file is not very transparent yet. By later today it will be. Right now, the second column is the coefficient. The third column is coefficient normalized by stdev, which is what I used for close reading.
