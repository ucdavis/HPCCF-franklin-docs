# Jobs

After [logging in to Franklin](../general/access.md), your session exists on the **head node**: a single,
less powerful computer that serves as the gatekeeper to the rest of the cluster.
To do actual work, you will need to write submission scripts that define your job and submit
them to the cluster along with resource requests.

## Batch Jobs: `sbatch`

Most of the time, you will want to submit jobs in the form of job scripts.
The batch job script specifies the resources needed for the job, such as the number of nodes, 
cores, memory, and walltime.
A simple example would be:

``` { .slurm .copy title="jobscript.sh" }
#!/bin/bash 
# (1)
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --time=01:00:00
#SBATCH --mem=100MB
#SBATCH --partition=low

echo "Running on $(hostname)"
```

1. This will determine the shell Slurm uses to execute your script. You could, for example, use `/bin/sh` or `/bin/zsh`.

Which can be submitted to the scheduler by running:

``` console
$ sbatch jobscript.sh
Submitted batch job 629
```

The job script is a normal shell script -- note the `#!/bin/bash` -- that contains additional directives.
`#SBATCH` lines specify directives to be sent to the scheduler; in this case, our resource requests:

- `--ntasks`: Number of tasks to run. Slurm may schedule tasks on the same or different nodes.
- `--cpus-per-task`: Number of CPUs (cores) to allocate per task.
- `--time`: Maximum wallclock time for the job.
- `--mem`: Maximum amount of memory for the job.
- `--partition`: The queue partition to submit to. See the [queueing](queues.md) section for more details.

More information on resource requests can be found in the [**Resources**](resources.md) section.

??? warning

    Jobs that exceed their memory or time constraints will be automatically killed.
    There is no limit on spawning threads, but keep in mind that using far more threads than requested cores
    will result in rapidly decreasing performance.

`#SBATCH` directives directly correspond to arguments passed to the `sbatch` command. As such, one could remove
the lines starting with `#SBATCH` from the previous job script and submit it with:

``` console
$ sbatch --ntasks=1 --cpus-per-task=1 --time=01:00:00 --mem=100MB --partition=low jobscript.sh
```

Using directives with job scripts is recommended, as it helps you document your resource requests.

## Interactive jobs: `srun`

Sometimes, you want to run an interactive shell session on a node, such as running an [IPython](https://ipython.org/) session.
`srun` takes the same parameters as `sbatch`, while also allowing you to specify a shell.
For example:

``` console
$ srun --ntasks=1 --time=01:00:00 --mem=100MB --partition=low --pty /bin/bash
srun: job 630 queued and waiting for resources
srun: job 630 has been allocated resources
camw@c-8-42:~$
```

Note that addition of the `--pty /bin/bash` argument.
You can see that the job is queued and then allocated resources, but instead of exiting, you are brought
to a new prompt.
In the example above, the user `camw` has been moved onto the node `c-8-42`, which is indicated by the new terminal
prompt, `camw@c-8-42`.
The same resource and time constraints apply in this session as in `sbatch` scripts.

??? note

    This is the only way to get direct access to a node: you will not be able to simply do `ssh c-8-42`, for example.

## Listing jobs: `squeue`

`squeue` can be used to monitor running and queued jobs.
Running it with no arguments will show *all* the jobs on the cluster; depending on how many users are active, this could be a lot!

``` console
$ squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
               589 jawdatgrp Refine3D adtaheri  R 1-13:51:39      1 gpu-9-18
               631       low jobscrip     camw  R       0:19      1 c-8-42
               627       low Class2D/ mashaduz  R      37:11      1 gpu-9-58
...
```

To view only *your* jobs, you can use `squeue --me`.

``` console
$ squeue --me
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
               631       low jobscrip     camw  R       0:02      1 c-8-42
```

The format -- which columns and their width -- can be tuned with the [`--format`](https://slurm.schedmd.com/squeue.html#OPT_format) parameter.
For example, you might way to also include how many cores the job requested, and widen the fields:

``` {.console .copy}
$ squeue --format="%10i %.9P %.20j %.10u %.3t %.25S %.15L %.10C %.6D %.20R"
JOBID      PARTITION                 NAME       USER  ST                START_TIME       TIME_LEFT       CPUS  NODES     NODELIST(REASON)
589        jawdatgrp     Refine3D/job015/   adtaheri   R       2023-01-31T22:51:59         9:58:38          6      1             gpu-9-18
627              low      Class2D/job424/   mashaduz   R       2023-02-02T12:06:27        11:13:06         60      1             gpu-9-58
```

## Canceling jobs: `scancel`

## Job Information: `sstat` and `scontrol`

``` yaml
theme:
  features:
    - content.code.annotate # (1)
```

1.  :man_raising_hand: I'm a code annotation! I can contain `code`, __formatted
    text__, images, ... basically anything that can be written in Markdown.