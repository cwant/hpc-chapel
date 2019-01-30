---
title: "Running code on multiple machines"
teaching: 120
exercises: 60
questions:
- "What is a locale?"
objectives:
- "First objective."
keypoints:
- "Locale in Chapel is a shared-memory node on a cluster."
- "We can cycle in serial or parallel through all locales."
---

So far we have been working with **single-locale** (one machine) Chapel
codes that may run on one or many cores on a single
compute node.

Our programs have made use of the **shared memory space** of a
single machine, and have accelerated computations by launching concurrent
tasks on individual cores in parallel.

Chapel codes can also run on multiple machines (nodes) on a compute
cluster. In Chapel this is referred to as **multi-locale** execution.

We will get set up for multi-locale work by first creating a very
simple script called `locales.chpl`:

```
writeln(Locales);
```
{: .code}

This code will print the built-in global array `Locales`, a list of
locales that our program has access to (these locales are usually
provisioned through a scheduler).

Before we can do anything else though, we will need to do some setup to
compile and run this small program.

## Setup

Setting up chapel to run multi-locale can be tricky, as the software
is tied closely to both the scheduler used and the interconnect
(the networking between nodes) of the cluster. Chapel (mostly)
assumes that your jobs will be spawned through a scheduler.

### If you are on our training cluster ...

To compile the multi-locale program, we need to configure our environment
first:

```
module unload chapel-single
source ~centos/start-chapel-multi-locale.sh
```
{: .code}

We can now compile our program as usual:

```
chpl --fast locales.chpl -o locales
```
{: .code}

We will submit our job to our cluster using the following
submission script `locales_submit.sh`:

```
#!/bin/bash
#SBATCH --time=00:05:00
#SBATCH --mem-per-cpu=1000
#SBATCH --nodes=4
#SBATCH --cpus-per-task=3
#SBATCH --output=locales.txt

./locales -nl 4
```
{: .code}

And submit it to the cluster:

```
sbatch locales_submit.sh
```
{: .code}

Skip the next session, as it doesn't apply to the training cluster
(you may want to revisit it later if you start submitting jobs to
Compute Canada machines).

### If you are on a 'real' Compute Canada cluster ...

To compile the multi-locale program on `cedar` (for example), we need to
configure our environment first:

```
# This might be needed if we were doing previous chapel work:
module unload chapel-single

module load gcc
module load chapel-slurm-gasnetrun_ibv
# Or if you want a specific version do:
# module load chapel-slurm-gasnetrun_ibv/1.15.0
```
{: .code}

We can now compile our program as usual:

```
chpl --fast locales.chpl -o locales
```
{: .code}

We will submit our job to the cluster using the following
submission script `locales_submit.sh`, but keep in mind that
you will need to provide a valid accounting group below:

```
#!/bin/bash
#SBATCH --time=00:05:00
#SBATCH --mem-per-cpu=1000
#SBATCH --nodes=4
#SBATCH --cpus-per-task=3
#SBATCH --output=locales.txt
#SBATCH --account=YOUR-ACCOUNTING-GROUP-HERE

# The following prevent out-of-memory errors ...
export GASNET_PHYSMEM_MAX=1G
export GASNET_PHYSMEM_NOPROBE=1

srun ./locale_real -nl 4
```
{: .code}

And submit it to the cluster:

```
sbatch locales_submit.sh
```
{: .code}

## What just happened?

A few things to note about the job we queued:

* We have asked for three CPUs of four machines;
* We have asked for a maximum of 1GB of memory per CPU
  (`--mem-per-cpu` is expressed in MB);
* We have requested that the scheduler run our job for a maximum of
  five minutes (the job will take much less than that to run);
* We have requested that the output from the program be placed in the
  file `locales.txt`

Also, you will notice that the program we have run has the command line
option `-nl 4`. This indicates that the number of locales (different machines
to run on) is 4.

You can use `squeue` to see if your job has run. When it completes, we
can inspect the output in `locales.txt`:

~~~
LOCALE0 LOCALE1 LOCALE2 LOCALE3
~~~
{: .output}

This is not terribly informative, but lets us know that our program
does indeed have acces to four machines.

## Getting some more information

Let's modify our code in `locales.chpl` to get some more information
about the locales. To do this, we will actually run some code on
each locale:

```
for loc in Locales do {
  on loc do {
    writeln("Locale ", loc, " is named ", here.name);
  }
}
```
{: .code}

We are starting a **serial** `for` loop, cycling through
all locales, and on each locale we print the locale's name
(the hostname of each node).

The built-in variable class `here` refers to the locale on which
the code is running, and `here.name` is its hostname.

Re-compile and re-submit your job:
```
chpl --fast locales.chpl -o locales
sbatch locales_submit.sh
```
{: .bash}

This will produce output similar to

~~~
Locale LOCALE0 is named node4.training.uofa.c3.ca
Locale LOCALE1 is named node2.training.uofa.c3.ca
Locale LOCALE2 is named node1.training.uofa.c3.ca
Locale LOCALE3 is named node3.training.uofa.c3.ca
~~~
{: .output}

This program ran in serial, starting a task on each locale
only after completing the same task on the previous locale.

Also of note: the node names of the locales are in an
arbitrary order.

To run this code in **parallel**, starting four simultaneous tasks
(one per locale) we can simply replace
`for` with `forall`:

```
forall loc in Locales do {
  on loc do {
    writeln("Locale ", loc, " is named ", here.name);
  }
}
```
{: .code}

Four tasks will start in parallel when you re-compile and
re-submit the job.

The output looks similar to before, except perhaps the order
and the names may differ depending on
runtime conditions:

~~~
Locale LOCALE0 is named node1.training.uofa.c3.ca
Locale LOCALE3 is named node3.training.uofa.c3.ca
Locale LOCALE2 is named node2.training.uofa.c3.ca
Locale LOCALE1 is named node4.training.uofa.c3.ca
~~~
{: .output}

We can print some other attributes of each locale.
Here it is actually useful to revert to the serial loop
`for` so that the print statements appear in order:

~~~
use Memory;

for loc in Locales do {
  on loc {
    writeln("Locale #", here.id, " (", loc, ") ...");
    writeln("  * is named: ", here.name);
    writeln("  * has ", here.numPUs(), " processor cores");
    writeln("  * has ", here.physicalMemory(unit=MemUnits.GB, retType=real), " GB of memory");
    writeln("  * has ", here.maxTaskPar, " maximum parallelism");
  }
}
~~~
{: .source}

Re-compile/re-submit yeilds the output in `locales.txt`:

~~~
Locale #0 (LOCALE0):
  * is named: node2.training.uofa.c3.ca
  * has 3 processor cores
  * has 7.14685 GB of memory
  * has 3 maximum parallelism
Locale #1 (LOCALE1):
  * is named: node3.training.uofa.c3.ca
  * has 3 processor cores
  * has 7.14685 GB of memory
  * has 3 maximum parallelism
Locale #2 (LOCALE2):
  * is named: node4.training.uofa.c3.ca
  * has 3 processor cores
  * has 7.14685 GB of memory
  * has 3 maximum parallelism
Locale #3 (LOCALE3):
  * is named: node1.training.uofa.c3.ca
  * has 3 processor cores
  * has 7.14685 GB of memory
  * has 3 maximum parallelism
~~~
{: .output}

Note that while Chapel correctly determines the number of cores
available inside our job on each node,
and the **maximum parallelism** (which is the same as the number
of cores available!), it lists the **total physical memory** on
each node available to all running jobs which is not the same as
the total memory per node allocated to our job.

{% include links.md %}
