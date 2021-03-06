---
title: 'Getting started on CBSUBrito'
data: 2019-04-26
permalink: /posts/2019/04/blog_post_cbsubrito/
tags:
  - server
  - SGE
  - brito lab
---

Disclaimer: This blog post will only be relevant to members of the Brito Lab. 

Software
--------
To see which software are currently available on the server, [go to CBSU's website](https://cbsu.tc.cornell.edu/lab/labsoftware.aspx). If the program you need is not available, email cbsu@cornell to submit a request.


Using a Queue
-------------
All CBSUBrito machines are equipped with a Sun Grid Engine (SGE) queue manager. It is important to use the queueing system since we are all trying to access shared resources. When you submit a submission script ("a qsub"), you must specify which queue you are submitting to. 

There are long queues for jobs that will take more than 4 hours to complete. 
 
  ```console
    long.q@cbsubrito
    long.q@cbsubrito2
    long.q@cbsubrito3
 ```

There are short queues for shorter jobs (<4 hours). These jobs will be terminated after 4 hours. This queue is always available and will supercede long.q jobs if they are submitted. A certain number of cores are kept for the short.q at all times. 
 
 ```console
    short.q@cbsubrito
    short.q@cbsubrito2
    short.q@cbsubrito3
```

Basic commands for SGE queuing system
-------------------------------------
To use the SGE queue manager, you need to make a shell script. I always name my submission scripts "run_<program>.qsub" so that all my shell scripts are easily recognizable. Then, you submit the job as follows:
  
   `$ qsub run_myjob.qsub`

You can check the status of that job with:
  
   `$ qstat run_myjob.qsub`
        
 You can add options to the 'qstat' command such as 'qstat -u fnn3' if you want to see only jobs owned by user fnn3.
 You can delete a job using:
 
 ```console
    qdel JOB_NAME
    qdel -j JOB_ID  # Find the job ID number by checking 'qstat'
    qdel *          # Delete all of your own jobs  
 ```       
        
 For more information:
   [This is a good guide for commonly used QSUB options](https://www.nas.nasa.gov/hecc/support/kb/commonly-used-qsub-options-in-pbs-scripts-or-in-the-qsub-command-line_175.html)
 
   [A quickstart guide for using SGE](http://star.mit.edu/cluster/docs/latest/guides/sge.html)


Example submission scripts
--------------------------
[You can download a template qsub here](http://fnew.github.io/files/qsub_template.qsub).

There are example scripts for running QC, alignments, and more on all three machines: /workdir/scripts.

```console
       #$ -S /bin/bash                              #Set the environment to bash
       #$ -N job123                                 # Name the job
       #$ -o /workdir/users/username/job123.out     # Set the standard out 
       #$ -e /workdir/users/username/job123.err     # Set the standard error log
       #$ -l h_vmem=5G                              # Request 5GB of memory for this task
       #$ -t 1-4                                    # Tells the scheduler which array jobs to run, i.e. 1-4, only 4, 50-100, etc
       #$ -q short.q@cbsubrito                      # Specifies which queue to use on which machine
 
        WRK=/workdir/users/username/                               # Set path to the working directory
        REF=/workdir/users/username/data/references/some_genome    # Set path to the reference database
        OUTDIR=/workdir/users/username/out                         # Set the output directory
 
        LIST=$WRK/design_files/zoo_sample_names.txt                #Set the list of file names
        DESIGN=$(sed -n "${SGE_TASK_ID}p" $LIST)                   #Look for the specific file that you want to run
        NAME=`basename "$DESIGN"`                                  #Pull out the name of the file you want to run
 
        READ1=$WRK/data/${NAME}.derep_1.trim3.fastq                #Set READ1 for task_id 1, 2, 3, ...
        READ2=$WRK/data/${NAME}.derep_2.trim3.fastq                #Set READ2 for task_id 1, 2, 3, ...

        cd $OUTDIR                                                 # Changes directory to the output directory to run the program 
        export PATH=/programs/program_that_i_want_to_run           # Load the program you want to run, i.e. BWA, BLAST, etc
        bwa mem -a $REF $READ1 $READ2 > ${NAME}.sam                # Run the program according to its specifications
```
