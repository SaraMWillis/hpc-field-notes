
# Slurm

## Backfill

### Root of the Question and the Explanation

!!! info "Official docs"
    For Slurm's official documentation, see: [Backfill Scheduling](https://slurm.schedmd.com/sched_config.html#backfill)

I went through a bit of a saga trying to figure out why a particular user only had 10 jobs start at a time, after submitting something like 1000 jobs, when the jobs were fairly small. 

Essentially, we have a few different partitions of varying levels of priority, one of which is very low priority and can be preempted (jobs can be interrupted and restarted). The user I mentioned had high priority jobs and came to us asking why only 10 of their jobs were starting at a time when another user, who was using the lowest priority preemptible partition, also had jobs starting and those jobs weren't getting interrupted. 

The problem boiled down to Slurm's backfill. Basically, backfill is a process that runs separately from the regular scheduling mechanism to move smaller jobs through the queue to utilize available space on machines that are reserved for larger scheduled jobs, as long as those smaller jobs can run before the higher priority reservation kicks in. Since backfill is a computationally expensive process, only a finite number of jobs per user are considered every backfill cycle.

A lot of the time, backfill quietly makes things more efficient and folks may never know that it exists. However, when the scheduler gets bricked up processing a large number of jobs and resources are in short supply, backfill behavior becomes a lot more noticeable. 

In this case, we had several thousand jobs in queue, and the highest priority partition had a 1400 core job pending that was clogging up the pipes for other high priority jobs. As a result, most of what was making its way through the queue were small backfill jobs that could be run and completed before all resources were cleared to make way for the 1400 core job. 

Since our backfill is set to consider 10 jobs per user per backfill cycle, jobs were trickling in from all users across all partitions. For example, the low priority partition user:

```console title="Number of low priority jobs started per hour"
(cluster) [lookitsme@login_node ~]$ for i in $(seq 8 15) ; do sacct --user=<User A> --format=Start -X | grep T$i: | wc -l ; done 
0 0 240 206 200 200 193 969
```

and the high priority partition user:

```console title="Number of high priority jobs started per hour"
(cluster) [lookitsme@login_node ~]$ for i in $(seq 8 15) ; do sacct --user=<User B> --format=Start -X | grep T$i: | wc -l ; done 
0 0 120 120 110 110 110 70
```

In terms of the lowest priority jobs not getting preempted, essentially the issue is that backfill jobs fall outside of standard job scheduling behavior. The preemptible jobs were allocated resources that Slurm had determined could be used temporarily without delaying higher-priority jobs scheduled to start in the future. The only other jobs that could potentially use those resources would be other backfill jobs, but backfill jobs don't preempt other jobs. The end result is low priority, preemptible jobs are not typically preempted when run as backfill, unless, hypothetically, the larger job set to run on the resources they're occupying starts earlier than anticipated.

The one thing that still bugs me is that, in the example shown above, the lower priority partition user had twice as many jobs working their way through, approximately. I found they were submitting from multiple partitions, so it seems like the user who was using the lower priority partitions may have had 10 jobs considered *per partition* rather than globally. This doesn't fully mesh with the setting in our slurm.conf which sets `bf_max_job_user` which, hypothetically, should override per-partition limits for users, so this is still an unresolved question for me. 

### tldr; 

1. Slurm uses a mechanism called backfill to slot small/windfall jobs into the system to make sure things are running at full capacity. These jobs only run when resources are available that won't get prioritized in front of other higher priority jobs. Small windfall jobs run in this category.

2. The system is configured to only consider 10 backfill jobs per user across all partitions. So this means that if a user's jobs are being considered for backfill, only the 10 highest priority backfill jobs are considered while the rest are ignored. This means, if a user has small jobs pending for rare resources, they'll brick up their own backfill queue and the small jobs pending below them won't be considered.

3. Backfill information can be found in `slurm.conf` set in `SchedulerParameters`. Refresher: you can find `slurm.conf` with `env | grep CONF`. Some of our settings:

    |<div style="width:200px">Parameter</div>|Value|What it does|
    |-|-|-|
    |`partition_job_depth`|`200`|Specifies how many jobs are tested in any single partition, default value is 0 (no limit). Ours is set to 200, so each scheduling cycle, only the top 200 jobs are considered.|
    |`bf_interval`|`300`|How frequently, in seconds, backfill is run. So jobs are evaluated on our system every 5 minutes|
    |`bf_max_job_user`|`10`|How many jobs per user are considered for backfill across all partitions|
