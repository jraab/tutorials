


#Preliminary notes for before we get started:

1. This is just an example of how the process is done. There are lots of ways to speed each step up in a production environment, and there are lots of different ways to do the same things using other programs.

1. All of these programs will run on a Macintosh computer. If you have a Mac, you can do this at home!

1. Kure is a big fancy cluster that lots of people on campus use. As a result, things on Kure are a little more complicated than they would be on your personal computer in exchange for “playing well with others”, and speed, RAM and disk space.

1. This is the official documentation on how to do things in kure: http://help.unc.edu/help/getting-started-on-kure/

#Preliminary house keeping things to take care of the first time you login.

 **Logging in and Creating a Directory For Yourself in Our Lab’s Space:**
When you log into Kure, it puts you in your home directory (in the figure below, I’ve indicated the location of your home directory relative to other important directories):

![image of directory structure](MagnusonKureDirectories.png)

Because my “onyen” username is “starmer”, my home directory is `/nas02/home/s/t/starmer`

Your home directory will also be related to your “onyen” and will be similar to mine. To find out what yours is, type these two commands:

    cd
    pwd

The first command, `cd`, just makes sure that your current working directory is your home directory, and the second one, `pwd`, prints out the path to your current working directory. Don’t worry too much about the details of what I just wrote at this time, just type `cd` (followed by return) and then `pwd` (followed by return).

The RNA-seq data for the Magnuson lab is stored here:

`/proj/magnuslb/HTSF`

To look and see what data is there, you have a few options. Here are two:

    ls /proj/magnuslb/HTSF

or

    cd /proj/magnuslb/HTSF
    ls

There is a “users” directory that folks affiliated with the Magnuson lab can use. This directory gives you room to store large files. You don’t have much space in your “home” directory, so the “users” directory is where you do your work.

`/proj/magnuslb/users`

I’ve already added a directory for myself in the “users” directory:

`/proj/magnuslb/users/starmer`

You should make one for yourself, too.  There are lots of ways to make a directory for yourself. Here are two of them (replace YOUR_ONYEN with your actual onyen username):

    mkdir /proj/magnuslb/users/YOUR_ONYEN

or

    cd /proj/magnuslb/users
    mkdir YOUR_ONYEN

#One last thing: What to do when things go wrong: Ctrl-c

Once we start this pipeline, keep in mind that we are working with extremely large files. Some of these might have over a billion lines in them. At some point, you will make a mistake - you will type everything right except you'll leave out a small, but very important detail - and suddenly all billion lines will start printing to the screen. This is panic inducing. Your terminal will turn into a blur and you're hands will start to sweat. Don't freak out, just type:

`Ctrl-c`

That stands for "control c". `Ctrl-c` is the emergency break in Unix. When things freak out, just type it and all will be well again.

#The Pipeline

**STEP 1: Make a new directory for the analysis.** You will want to make a separate directory for each experiment you want to do an analysis for.

Do this from within `/proj/magnuslb/users/YOUR_ONYEN/` (Replace `EXP_NAME` with the name of the experiment. `EXP_NAME` could something like `rna_seq_dec14` or `ko_v_wt_rna_dec14`):

    mkdir EXP_NAME

Now change directories so that you’re in the new directory:

    cd EXP_NAME

**STEP 2: Figure out where your actual sequence reads are.** The high-throughput sequencing facility puts the reads in a directory called “HTSF” (which is short for “high-throughput sequencing facility”). However, they put each run worth of data into a subdirectory which starts with a date, like `141203` or `140124`, where the first two digits are for the year, the second two are for the month, and the last two digits are for the day. You can figure out which directory is yours with one of the following commands.

    ls /proj/magnuslb/HTSF

or

    ls ../../../HTSF

The second command assumes you’re in the `EXP_NAME` directory that you just made.

To make things easier to describe, we’ll call the directory in HTSF that has your data `HTSF_DATA`. Now, sometimes all of the data `HTSF_DATA` is yours, but sometimes it’s not. You can see the files in it (to see which one’s are yours) with one of the following commands:

    ls /proj/magnuslb/HTSF/HTSF_DATA

or

    ls ../../../HTSF/HTSF_DATA

You should see a lot of filenames that end in “fastq.gz”. Figure out which ones are yours. For the rest of the tutorial, I’ll say one file is called `rna_seq_wt.fastq.gz` and one file is called `rna_seq_ko.fastq.gz.`


**STEP 3: Make a link (aka, alias) to your fastq.gz files.** This will keep you from having to type in a path to the file when you run the analysis commands. For the sake of the tutorial, assume you have one RNA-seq file from WT and one RNA-seq file from a knock-out mutant. The following two commands will create links to the original files in your new directory: (Note: these commands assume you are here: `/proj/magnuslb/users/YOUR_ONYEN/EXP_NAME`):

    ln -s /proj/magnuslb/HTSF/HTSF_DIR/rna_seq_wt.fastq.gz .
    
    ln -s /proj/magnuslb/HTSF/HTSF_DIR/rna_seq_ko.fastq.gz .

Note the `.` in those commands. It's important. Everything before it means "make a link to this file", and the `.` says, "make that link right here!".

**STEP 4: Make directories to hold the output of various analyses for each file.** You’re going to run “tophat” and you might run “fastqc”, and the output from these programs needs to go into a directory (one per file of reads you want to align).
Here are the commands to make directories for the WT and KO file samples: (Note these commands assume you are here: `/proj/magnuslb/users/YOUR_ONYEN/EXP_NAME`):

    mkdir rna_seq_wt_results

and

    mkdir rna_seq_ko_results

**STEP 5: Look at the first few alignments for each file.**

    zmore rna_seq_wt.fastq.gz

and

    zmore rna_seq_ko.fastq.gz

In these commands, `zmore` allows you to look inside a text file that has been compressed with `gzip`. If the file is not compressed, you can just use `more`. Both `zmore` and `more` show you data one page at a time. `rna_seq_wt.fastq.gz` could have close to a billion lines in it and we don't (and can't) look at them all. We use `zmore` and `more` to look at the first few pages. After the first page of alignments is shown, you can hit the spacebar to see the second page of alignments. Hit the spacebar again and you'll see the third page of alignments, etc.
 
If you see “N” at the start or ends of alignments, you can trim them off with trimmomatic.

Note: before we get started with the actual analysis (running programs to determine the quality of the reads, aligning, counting and making UCSC tracks, etc.), let me say that there are three ways to get work done on Kure. Each way has different pros and cons, and, ultimately, it makes sense to know them all. All three methods are covered in this tutorial, but not all at once (I don't give you three ways to get each step done).

**STEP 6: (OPTIONAL - But read this section anyway.) Run FASTQC to asses the overall quality of the run.** Why optional? The information this generates is really only useful to help figure out what happened when things went wrong in a super bad away. If things went “sort of wrong” (i.e. you didn’t get a ton of reads, or a lot of your reads map to repetitive sequences), you’re still going to align what you got reads and try to squeeze everything you can out of it. Rarely has information generated by FASTQC caused me to change course or take a different plan.

First load the module (you only have to do this once after you login):

    module load fastqc

Now run the following fastqc (see http://www.bioinformatics.babraham.ac.uk/projects/fastqc/ for more details):

    bsub -n 1 -J fastqc_wt -o fastqc_wt_stdout.txt -e fastqc_wt_stderr.txt fastqc --outdir=rna_seq_wt_results rna_seq_wt.fastq.gz

If you want to run that command on the "ko" data, you can replace `wt` with `ko`

What do all those commands mean?

`-n 1` 	This tells kure to use one processor on a compute node (there are 8 CPUs per compute node).

`-J fastqc_wt` 	This specifies the name of the "job". The bsub command submits jobs to be done. This job is going to be called `fastqc_wt`, and when we look at our jobs (and whether they are running, or stuck or completed) with the `bjobs` command, it will be listed under `fastqc_wt`.

`-o fastqc_wt_stdout.txt`	This is the “standard output” file for the program we want to run (in this case, `fastqc`). Since we are submitting `fastqc` as a job to a compute node, `fastqc `can’t write messages directly to your terminal. Instead some of those messages will go in this file. This file will rarely be of use, but is here for completeness sake.

`-e fastqc_wt_stderr.txt`	This is the “standard error” file for the program we want to run (in this case, `fastqc`). Since we are submitting `fastqc` as a job to a compute node, `fastqc` can’t write error messages directly to your terminal. Instead, `fastqc`’s error messages will go in this file. This file can be extremely useful for debugging. If things fail, you can view it’s contents with the following command: `more fastqc_wt_stderr.txt`

`fastqc`	This is the program we actually want to run on a compute node.

`--outdir=rna_seq_wt_results`	This tells `fastqc` where we want to save the results.

`rna_seq_wt.fastq.gz`	This tells` fastqc` the name of the file we want to process.

Now, how do we tell if the program is running or not? You type `bjobs`. If all went well, and the program is currently running, you should see something that looks like this:

    JOBID   USER    STAT  QUEUE      FROM_HOST   EXEC_HOST   JOB_NAME   SUBMIT_TIME
    875410  starmer RUN   week       kure-login1 public_host fastqc_wt  Dec 15 09:39

If the program is done running, will see this:

    No unfinished job found

Once the program is done running, let's make sure it did what we expected it to do.  First, type:

    ls

You should see two new files, one called `fastqc_wt_stderr.txt` and one called `fastqc_wt_stdout.txt`. Generally, `fastqc_wt_stderr.txt` is more interesting than `fastqc_wt_stdout.txt`. Let's look inside them. Since they are text files, but not compressed, we can use `more` to look at them. The following command:

    more fastqc_wt_stderr.txt

will show you progress messages and error messages that a `fastqc` generated when it ran. If things worked, you should see the following progress statements:

    Approx 5% complete for rna_seq_wt.fastq.gz
    Approx 10% complete for rna_seq_wt.fastq.gz
    Approx 15% complete for rna_seq_wt.fastq.gz
    Approx 20% complete for rna_seq_wt.fastq.gz
    Approx 25% complete for rna_seq_wt.fastq.gz
    Approx 30% complete for rna_seq_wt.fastq.gz
    Approx 35% complete for rna_seq_wt.fastq.gz
    Approx 40% complete for rna_seq_wt.fastq.gz
    Approx 45% complete for rna_seq_wt.fastq.gz
    Approx 50% complete for rna_seq_wt.fastq.gz
    Approx 55% complete for rna_seq_wt.fastq.gz
    Approx 60% complete for rna_seq_wt.fastq.gz
    Approx 65% complete for rna_seq_wt.fastq.gz
    Approx 70% complete for rna_seq_wt.fastq.gz
    Approx 75% complete for rna_seq_wt.fastq.gz
    Approx 80% complete for rna_seq_wt.fastq.gz
    Approx 85% complete for rna_seq_wt.fastq.gz
    Approx 90% complete for rna_seq_wt.fastq.gz
    Approx 95% complete for rna_seq_wt.fastq.gz

If things didn't work, you would see some sort of error message.

The following command will tell you that you ran `fastqc` and that you should check `fastqc_wt_stderr.txt` for progress and error messages:

    more fastqc_wt_stdout.txt

If `fastqc` ran successfully, you can look at it's output. This is stored in rna_seq_ko_results. We can look at the file names with this command:

    ls rna_seq_wt_results/

And we should see two files, `rna_seq_wt_fastqc.html` and  `rna_seq_wt_fastqc.zip`. `rna_seq_wt_fastqc.html` is a webpage that you can view in your web browser. To do this, download both files to your computer, unzip `rna_seq_wt_fastqc.zip` (this zip file contains graphics and other files that will be displayed as part of the web page) and then open up `rna_seq_wt_fastqc.html` in your browser ("File->Open File...")

If you don't want to go through the trouble of downloading those files, or you don't particularly want to see the fancy graphical output, you can view a text summary of the results on Kure. Here's how:

    unzip rna_seq_wt_results/rna_seq_wt_fastqc.zip
    more rna_seq_wt_results/rna_seq_wt_fastqc/summary.txt

The first command `unzip rna_seq_wt_results/rna_seq_wt_fastqc.zip` uncompresses `rna_seq_wt_fastqc.zip` and the second command looks at the text summary that `fastqc` generated. Here's what mine looked like:

    PASS	Basic Statistics	rna_seq_wt.fastq.gz
    PASS	Per base sequence quality	rna_seq_wt.fastq.gz
    WARN	Per tile sequence quality	rna_seq_wt.fastq.gz
    PASS	Per sequence quality scores	rna_seq_wt.fastq.gz
    FAIL	Per base sequence content	rna_seq_wt.fastq.gz
    PASS	Per sequence GC content	rna_seq_wt.fastq.gz
    PASS	Per base N content	rna_seq_wt.fastq.gz
    PASS	Sequence Length Distribution	rna_seq_wt.fastq.gz
    WARN	Sequence Duplication Levels	rna_seq_wt.fastq.gz
    PASS	Overrepresented sequences	rna_seq_wt.fastq.gz
    PASS	Adapter Content	rna_seq_wt.fastq.gz
    FAIL	Kmer Content	rna_seq_wt.fastq.gz


Remember, regardless of what `fastqc` tells you about your data, you're probably going to want to press on with the analysis. Don't worry too much about whether something passed or failed a test until you look at your results.

**STEP 7: (OPTIONAL) Trim reads with trimmomatic.** This program will do a lot of things, but I only recommend it if you want to remove leading or trailing “N” that are in the reads. Additionally, it will remove the Illumina adapter sequences (which are usually already removed) and low quality reads (which can be filtered after alignment.)

First load the module (you only have to do this once after you login)

    module load trimmomatic

Now run trimmomatic (see http://www.usadellab.org/cms/?page=trimmomatic for details). Everything below should go on a single, continuous line - don’t type “return” until the very end:

    bsub -n 1 -J trim_wt -o trim_wt_stdout.txt -e trim_wt_stderr.txt trimmomatic SE -threads 1 -phred33 -trimlog trimmomatic_log.txt rna_seq_wt.fastq.gz rna_seq_wt_trim.fastq.gz ILLUMINACLIP:/nas02/apps/trimmomatic-0.32/Trimmomatic-0.32/adapters/NexteraPE-PE.fa:2:30:10 SLIDINGWINDOW:4:15 MINLEN:30 LEADING:3 TRAILING:3

This command does a lot… Just like with `fastqc`, `-n 1` tells Kure that we want to use one processor on the compute node, `-J trim_wt` is the name of our job, and `-o` and `-e` specify file names for “standard output” and “standard error” files. Then we tell `bsub` that we want to run `trimmomatic`. The following are it’s options (which are fully explained on it’s website):

`SE`	 This tells `trimmomatic` that we are working with “single-end” data (not paired-end, which can be specified with `PE`)

`-threads 1`	 This tells `trimmomatic` that we are only using one processor on the compute node.

`-phred33` 	This tells `trimmomatic` that we are using Phred33 style quality scores (the quality of each sequenced base in each read is included in a Phred33 format. Google this if you’re interested in more details).

`-trimlog trimmomatic_log.txt` 	This tells `trimmomatic` where to save a record of all the trimming it does to your file.

`rna_seq_wt.fastq.gz`	This is the name of the file we want to analyze and trim.

`rna_seq_wt_trim.fastq.gz` 	This is the name of the new file that trimmomatic will create with the trimmed reads.

`ILLUMINACLIP:/nas02/apps/trimmomatic-0.32/Trimmomatic-0.32/adapters/NexteraPE-PE.fa:2:30:10` 	This tells `trimmomatic` where the illumina adapter sequences are so it can remove them from the reads if they are still there. They are almost always removed, but I’ve included this bit here to be complete.

`SLIDINGWINDOW:4:15` 	This tells `trimmomatic` to toss out reads that have more with 4 low quality bases in a row.

`MINLEN:30`	This tells` trimmomatic` that a read must be at least 30 bp in length (if it trims so many Ns that the read gets shorter than 30bp, it tosses the whole read.)

`LEADING:3 TRAILING:3`	This tells `trimmomatic` to trim leading and trailing “N”s.

If you want to run that command on the "ko" data, you can replace "wt" with "ko".

You can look at your new "trimmed" file to verify that you removed leading and trailing "N"s with the following command:

    zmore rna_seq_wt_trim.fastq.gz

If things look good, do the system a favor a delete the "log" file that trimmomatic generated. This file contains descriptions of the modifications it made to each line in the original file. You won't ever use this information. So delete this file since it is pretty large and takes up a lot of space.

    rm trimmomatic_log.txt

Lastly, check the standard error file. Do this if things worked to see how many reads were dropped because they were low quality, and do this if things didn't work to figure out what went wrong:

    more trim_wt_stderr.txt

**STEP 8: Align the reads with Tophat** (see http://ccb.jhu.edu/software/tophat/manual.shtml for details):

Load the module:

    module load tophat

Now do the alignment (type this without hitting return until the end). This next command works with MM9 (to use MM10, just change the 9 to 10). It was also written assuming you ‘trimmed’ your file. If you didn’t, change `rna_seq_wt_trim.fastq.gz` to `rna_seq_wt.fastq.gz` :

    bsub -n 8 -R "span[hosts=1]" -J tophat_wt -o tophat_wt_stdout.txt -e tophhat_wt_stderr.txt tophat2 --library-type fr-unstranded -o rna_seq_wt_results -p 8 --no-novel-juncs -G /proj/seq/data/MM9_UCSC/Annotation/Genes/genes.gtf /proj/seq/data/MM9_UCSC/Sequence/Bowtie2Index/genome rna_seq_wt_trim.fastq.gz

Just like for `fastqc` and `trimmomatic`, we specify the job-name with `-J` and the standard output and standard error files with `-o` and `-e`.

For the alignment, we are using `-n 8`, this tells Kure we want to use 8 CPUs. We are also specifying `-R "span[hosts=1]"`, which tells Kure that we want all 8 of those CPUs to be on the same compute node. This means we want an entire compute node all to ourselves. In theory, this means `tophat2` will run 8 times as fast.

If Kure is really busy and people are running a ton of jobs already, than it can sometimes take a while before a whole node is available for a single job. You'll first get a sense that this is the case if you type `bjobs` a few times and you see "PEND" instead of "RUN". If this is the case, first, cancel this job submission with the command:

    bkill JOBID

where `JOBID` is the first number listed when you run `bjobs`. Then modify the `bsub` command by replacing  `-n 8` and `-p 8` with `-n 1` and `-p 1`. This will use a single CPU, like all the other commands we submitted so far (as for `-R "span[hosts=1]"`, you can leave it in or take it out. It shouldn't matter. A single CPU can only be run on a single host, so it doesn't matter, one way or the other).

Since kure seems to be busy these days, I'm going to go ahead and give you the single CPU command (this should take about 45 to an hour to run):

    bsub -n 1 -R "span[hosts=1]" -J tophat_wt -o tophat_wt_stdout.txt -e tophhat_wt_stderr.txt tophat2 --library-type fr-unstranded -o rna_seq_wt_results -p 1 --no-novel-juncs -G /proj/seq/data/MM9_UCSC/Annotation/Genes/genes.gtf /proj/seq/data/MM9_UCSC/Sequence/Bowtie2Index/genome rna_seq_wt_trim.fastq.gz

These are options we are sending to `tophat2`:

`--library-type fr-unstranded`	 Tells `tophat2` about what to expect in terms of how the reads should align. This option is more important if you have paired-end reads. Read the `tophat2` manual for more details if you run a paired in experiment.

`-o rna_seq_wt_tophat`	Tells `tophat2` where we want it to save the results of the alignment. Generally, `tophat2` will create a bunch of files, the one you will be most interested in is called `accepted_hits.bam`.

`-p 8`	This tells `tophat2` that we are going to let it use 8 CPUs. (Again, if Kure is really busy, you can change this to `-p 1` and change the `-n 8` to `-n 1` and your job might get run sooner).

`--no-novel-juncs` 	Tells `tophat2` to only look at splice junctions specified in a genome annotation file.

`-G /proj/seq/data/MM9_UCSC/Annotation/Genes/genes.gtf`	Tells `tophat2` where the genome annotation file is. In this case, we are specifying the MM9 genome annotation. Kure has annotations for MM10, HG19, etc. This file is used for, among other things, by the `--no-novel-juncs` option. It is also used to build a “transcriptome” from the genome. `Tophat2` will first try to align reads to this transcriptome and then it will align all remaining genes to the genome.

`/proj/seq/data/MM9_UCSC/Sequence/Bowtie2Index/genome`	Tells `tophat2` where the genome is. In this case we’re going to align to the MM9 genome. Kure has annotations for MM10, HG19, etc.

`rna_seq_wt_trim.fastq.gz` This is the name of the file we want to align.

For the ko file, just replace `rna_seq_wt` with `rna_seq_ko`

**STEP 9: Verify that the alignment worked and examine summary statistics.**

If you look inside `rna_seq_wt_results`, you should see a bunch of new files.

    ls rna_seq_wt_results/

There are two most important are: `accepted_hits.bam` and `align_summary.txt`. 

First, let's figure out how many reads aligned and get other summary statistics from `tophat2`.

    more rna_seq_wt_results/align_summary.txt

You should see something that looks like this:

    Reads:
              Input     :  21294910
               Mapped   :  20800204 (97.7% of input)
                of these:   1288217 ( 6.2%) have multiple alignments (5814 have >20)
    97.7% overall read mapping rate.

This tells us that there were over 21 million reads, and of those, 20 million mapped to somewhere in the genome. Of those, about 1 million reads mapped to multiple locations. Thus, of the 20 million reads that mapped to the genome, about 19 million mapped to unique locations. This is great.

`accepted_hits.bam` has all of the alignments for all of your reads. It is a "BAM" file. A "BAM" file is just a compressed version of a "SAM" file. Great... OK, a "SAM" file is just a text file that has your read, and a location in the genome where it aligned and the strand it aligned to, along with scores that indicate the quality of the alignment (these scores are called "mapq" scores, where "mapq" is short for "MAPping Quality") and a handful of other odds and ends. Reads that do not align are also in here, and just don't have genome coordinates (these have very low mapq scores). Reads that map to multiple locations have an entry for each location it could map to (note, these will have low mapq scores). Thus, these files can get really big pretty fast. That's way there is the "BAM" format, to compress these large files.

In order to take a look at the first ten lines of `accepted_hits.bam`, we have to use `samtools view` to uncompress it, and then we'll pipe the uncompressed alignments into `head`. If you forget to pipe the uncompressed alignments into `head`, your terminal will go crazy. If this happens, just type `Ctrl-c` as quickly as you can. 

First, load the `samtools` module:

    module load samtools

Now look at the first 10 lines in `accepted_hits.bam`

    samtools view rna_seq_wt_results/accepted_hits.bam | head

On each line, you should see something that looks like a read, and you should be able to figure out which chromosome and position it aligned to. `samtools` can filter out reads that did not map or reads with low mapq scores, and a ton of other things. See http://www.htslib.org/doc/samtools-1.1.html for more details.

**STEP 10: Count the number of reads per gene.**  This will be done with `htseq-count`. (See http://www-huber.embl.de/users/anders/HTSeq/doc/count.html for more details.) The version of `htseq-count` on Kure is old, and does not read "BAM" files directly (the new version does). To use the old version, you first have to translate the BAM file into SAM format. Here's how to do it.. 

First, load in the two modules you need (Note, `htseq-count` is a program written in “python”, and we are going to use a helper program, `samtools`, to get the aligned reads into `htseq-count`).

    module load python
    module load samtools

Now we are going to use `samtools` and `htseq-count`, connected with a pipe, `|`, to count the reads that map to genes. In unix, the pipe, `|`, takes the output of one program and redirects it to be used as the input of another program. In this case, it takes the output from `samtools view`, which decompresses the aligned reads and it “pipes” it into `htseq-count` to count the reads. Lastly, the output from `htseq-count` is dumped into `rna_seq_wt_results/wt_counts.txt`. This is done using the file redirection character `>`.

    bsub -n 1 -J count -o count_stdout.txt -e count_stderr.txt "samtools view rna_seq_wt_results/accepted_hits.bam | htseq-count --stranded=reverse -m intersection-nonempty - /proj/seq/data/MM9_UCSC/Annotation/Genes/genes.gtf > rna_seq_wt_results/wt_counts.txt"

Hey!!! We’ve put the output of a lot of programs so far into files and we haven’t used the `>` thingy yet! Well, some programs will accept an output file name as an input (so the program knows the name of the file you want to name the output). This is what we have done before. Here, htseq-count doesn’t accept an output filename as input. It wasn’t coded that way (some people code things this way, some folks don’t). So you have to use `>` to do the job.

For the ko file, just replace `rna_seq_wt` with `rna_seq_ko`

When this command finishes running, there should be a file called `wt_counts.txt` in the `rna_seq_wt_result` directory. You can use `head` to look at the counts for the first 10 genes (in alpha-numeric order).

    head rna_seq_wt_results/wt_counts.txt 

To see a summary of the reads, you can use `tail`, which will show you the last 10 lines in a file. The last 5 lines in `wt_counts.txt` contain summary statistics for how many reads mapped to genes and how many didn't.

    tail rna_seq_wt_results/wt_counts.txt

This is what I got when I ran tail on my file:

    Zyx	    1484
    Zzef1	933
    Zzz3	1686
    a	    38
    l7Rn6	503
    no_feature	4156660
    ambiguous	143286
    too_low_aQual	0
    not_aligned	0
    alignment_not_unique	5279595

The first five lines are the last 5 genes that reads were counted for and the last five lines are summary statistics for all the reads that were not counted. As you can see, about 4 million reads did not map to exons (this is `no_feature`), about 150,000 reads mapped to multiple genes (this is `ambigous`), and about 5 million reads mapped to multiple locations (this is `alignment_not_unique`). Out of the the 20 million reads that aligned to the genome, this is pretty good.

**STEP 11: Now make tracks to view on the UCSC genome browser.** We’ll do this with bedtools. (See http://bedtools.readthedocs.org/en/latest/content/bedtools-suite.html for more details).

    module load bedtools

Now we are going to start an "interactive" session on a compute node. When you do this, you don’t need to run `bsub` explicitly, because everything you do will already be run on a compute node. **Note:** This command will generate a few errors before giving you a new shell prompt. Don't worry about the errors. 

    bsub -q int -Ip /bin/bash

In the preceding command, `-q int` tells `bsub` to start an "interactive session", and `-Ip /bin/bash` tells `bsub` that you want to use the "bash" shell during that interactive session. "What is a shell?" you ask. A shell is, in some ways, just the prompt that you type all of the commands in. However, shells are also programming languages of their own. That means we can create variables and store information, like filenames or any other value, in them. There are different shells (`bash` is just one of many, there is also `csh`, `tcsh` and `sh`), and each one has it’s own programming syntax. `Bash` is a very commonly used shell and it supports tab-completion, so we’ll stick with it and in the next steps you’ll see how to create variables in bash and how to use them.

Now create a variable called `BAMFILE` that holds the name of the file we want to create a UCSC track for. We do this because it will save typing. We can refer to the variable, `BAMFILE`, which is relatively short compared to typing the full filename, which is long.

    BAMFILE="rna_seq_wt_results/accepted_hits.bam"

Now we want to determine how many reads are in the file (we use this to normalize the graph so it can be compared to other experiments in the UCSC genome browser). We can do this with the following commands:

    COUNTS=`samtools view -c -F 4 $BAMFILE`
    echo "COUNTS $COUNTS"

In the first line, we are counting the number of reads in `$BAMFILE` and saving the number in a new variable, `COUNTS`. Counting is done with `samtools view -c`, the `-F 4` option forces `samtools` to only count the mapped reads that are in the file. The second line, `echo "COUNTS $COUNTS"`, just prints the number of reads to the screen. You can save this number for reference later.

One thing to note, this number may be different from the one in the alignment summary. This is because we are including reads that map to multiple locations. One reason you might want to include these reads is to show that repetitive elements are transcribed. We don't know exactly which one is being transcribed, but it might be nice to see peaks over all of them.

Now we can calculate the scaling factor (reads per 10,000,000) that we will use to normalize the bedgraph. We save this scaling factor in a new variable, `NORM`. We use `bc` to calculate the scaling factor and we pipe in two parameters, `scale=4`, which rounds the scaling factor at the 4th decimal point, and `10000000/$COUNTS`, which is just the math that we want `bc` to do. Here are the commands:

    NORM=`echo "scale=4; 10000000/$COUNTS" | bc`
    echo "NORM:  $NORM"

Now we will call bedtools to create a bedgraph file that is scaled with the normalization factor that we just created. See the bedtools website for an explanation for the various options. In the following command, change "my track" to something more useful, (like "ko_h3k27me3").

    bedtools genomecov -split -ibam $BAMFILE -bg -scale $NORM -g /proj/seq/data/MM9_UCSC/Annotation/Genes/ChromInfo.txt -trackline -trackopts 'name="my track"' > wt.bedgraph
        
     exit

When this is done running, you should have a file called "wt.bedgraph". Let's use `head` to check out the first 10 lines and make sure that the track line is correct (See http://genome.ucsc.edu/goldenPath/help/hgTracksHelp.html#TRACK for more details on how to modify track lines).

    head wt.bedgraph

If all went well, you should see something that looks a little bit like this (your genome coordinates and track name should be different):

    track type=bedGraph name="my track"
    chr1	3005252	3005302	0.4033
    chr1	3006015	3006065	0.4033
    chr1	3006145	3006195	0.4033
    chr1	3006224	3006225	0.4033
    chr1	3006225	3006227	0.8066
    chr1	3006227	3006274	1.2099
    chr1	3006274	3006275	0.8066
    chr1	3006275	3006277	0.4033
    chr1	3006357	3006372	0.4033

If that is what you see, than it worked! You can download this file to your computer and then upload it to the UCSC genome browser as a custom track. Check it out! If it looks good, you can pat yourself on the back! 


**BONUS:** The last way to use `bsub`!!!

So far I’ve demonstrated two of the three ways to use bsub: 1) using it to issue commands all typed out on the prompt, possibly surrounded by quotation marks if you are using a pipe, and 2) using it in interactive mode where you can just type the commands one at a time.

The last way is where you put all of the commands in a file and have `bsub` execute all of the those commands. This is called "submitting a batch" job, because you submit a "batch" of commands all at once. "Batch" jobs are useful if you want to do something that is complicated (i.e. takes several steps), but you don’t want to wait until Kure is done processing before you tell it to do something else. Making a bunch of bedgraphs is one example - it’s complicated, and you might have 10 different files you want to do it for. Wouldn’t it be great if you could submit all 10 jobs at once, and not have to wait for one to complete before submitting the next one?

Here’s how to do this:

First, you have to make a file with the commands you want to run. There are lots of ways to make this file. Kure has built in text editors (`vi`, `emacs`, `pico`), or you can use an SCP application (like `Cyberduck` on Macs) and edit text files on Kure using `Text Wrangler`. Regardless, this what you put in the file:

    BAMFILE=$1
    TRACK_NAME=$2
    OUTFILE=$3
    
    QUOTED_NAME="name=\"$TRACK_NAME\""
    TRACK_OPT="'$QUOTED_NAME'"
    echo "TRACK_OPT $TRACK_OPT"
    
    COUNTS=`samtools view -c -F 4 $BAMFILE`
    echo "COUNTS $COUNTS"
    
    NORM=`echo "scale=4; 10000000/$COUNTS" | bc`
    echo "NORM:  $NORM"
    
    bedtools genomecov -split -ibam $BAMFILE -bg -scale $NORM -g /proj/seq/data/MM9_UCSC/Annotation/Genes/ChromInfo.txt -trackline -trackopts $TRACK_OPT > $OUTFILE

You’ll notice some similarities and differences in this file from when we ran these commands interactively. For the most part, things are the same, except now we have `BAMFILE=$1`, which means "make a variable called `BAMFILE` and set it to the first thing I see." We’ve also added `TRACK_NAME=$2`, which means "make a variable called `TRACK_NAME` and set it to the second thing I see." Lastly, we’ve added `OUTFILE=$3`, which means "make a variable called `OUTFILE` and set it to the third thing I see." Later in the commands we use these variables instead of typing out "my track" and "wt.bedgraph".

What does “set it to the first (second or third) thing I see” mean? Hang in there, you’ll understand in just a second...

Now, assume we saved that file as "makeBedgraph.sh". To run it, use this command:

    bsub -n 1 -J bg_wt -o bg_wt_stdout.txt -e bg_wt_stderr.txt "sh makeBedgraph.sh rna_seq_wt_results/accepted_hits.bam my_track wt.bedgraph"

The first part, `bsub -n 1 -J bg_wt -o bg_wt_stdout.txt -e bg_wt_stderr.txt`, is, by now, obvious. After that, things get interesting. The very first bit is just this:

`sh`

This tells `bsub` to “execute” everything that follows using the `sh` shell. (Note, you could probably use bash here, but writing your code so that `sh` can run it ensures maximum portability. Every unix machine on the planet can run `sh` scripts. Not all of them, however, can run `bash` scripts.) The first thing `sh` will do is put:

`rna_seq_wt_results/accepted_hits.bam my_track wt.bedgraph`

into `$1`, `$2`, and `$3`. That is, it will put `rna_seq_wt_results/accepted_hits.bam` into `$1`, it will put `my_track` into `$2` and, lastly, it will put `wt.bedgraph` into `$3`. After it does that, it will run `makeBedgraph.sh`. At this point the three variables, `$1`, `$2`, and `$3` in `makeBedgraph.sh` have been defined (and are no longer just "put the first thing I see in `$1`") and, if all goes well, you will end up with a new file called `wt.bedgraph`.

Now, here’s something cool. What if you had 3 files of WT data? You could enter the commands really quickly (if you use the "up-arrow" key) like this:

    bsub -n 1 -J bg_wt1 -o bg_wt1_stdout.txt -e bg_wt1_stderr.txt "sh makeBedgraph.sh rna_seq_wt1_results/accepted_hits.bam wt1_track wt1.bedgraph"
    
    bsub -n 1 -J bg_wt2 -o bg_wt2_stdout.txt -e bg_wt2_stderr.txt "sh makeBedgraph.sh rna_seq_wt2_results/accepted_hits.bam wt2_track wt2.bedgraph"
    
    bsub -n 1 -J bg_wt3 -o bg_wt3_stdout.txt -e bg_wt3_stderr.txt "sh makeBedgraph.sh rna_seq_wt3_results/accepted_hits.bam wt3_track wt3.bedgraph"

Now, if you want to find out what the read counts and normalization factors were, you can find them inside of `bg_wt_stdout.txt`

For extra credit…. Can you figure out how to add one more line to `makeBedgraph.sh` so that the final output has been compressed by `gzip`?
