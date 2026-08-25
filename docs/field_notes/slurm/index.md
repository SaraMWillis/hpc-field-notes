
# Slurm

## Backfill

For Slurm's official documentation, see: [Backfill Scheduling](https://slurm.schedmd.com/sched_config.html#backfill)

### Email Exchange

I went through a bit of a saga trying to figure out why a particular user only had 10 jobs start at a time when the jobs were fairly small. I'll post my email exchange with our sys admin, redacting names. If it's not obvious, when I'm trying to understand something, I can be... verbose (my bad).

> Basically, I'm curious how the backfill mechanism works and how that interacts with regular scheduling. Maybe I can describe my limited understanding and you can correct where I'm off base? 
> 
> From reading through some of the documentation, my understanding is backfill is a process that runs separately from the regular scheduling mechanism to move smaller jobs through the queue to utilize available space on machines that are reserved for regular jobs, as long as those smaller jobs can run before that reservation kicks in. Since backfill is a computationally expensive process, we've set it up so only 10 jobs per user are considered every backfill cycle which is why `<User A>`'s jobs were slowly trickling through the queue. Because the queue was so bricked up yesterday processing ~5000 jobs (and today it seems), the regular scheduling mechanism is taking a long time to process everything and get things running. From my understanding and looking at slurm.conf, only 200 jobs per partition are considered during a regular scheduling cycle, so, if that's the case (and then add in a 1400 core high pri job from `<user B>`), that might explain why the high priority queue was so slow. Essentially, this user's spending a long time in queue due to scheduler overload and high cluster utilization, but his jobs are benefitting from backfill since his and other users' short running jobs are getting allocated space as part of the backfill process? And since backfill doesn't seem to prioritize one partition over another, `<User C>`'s windfall jobs were trickling through at approximately the same rate. Is this roughly accurate?
> 
> A few other things (apologies for the question deluge, figured I'd try to get everything down in one go 🙂):
> 
> 1) Looking at `<User C>`'s rate of jobs moving from queued to running yesterday (this is the windfall user `<User A>` was complaining about), I do notice that they had roughly twice as many start running compared to `<User A>` (compressing to one line for space considerations, the output is a space-delimited list of the number of jobs that started running at hours 8, 9, 10, ..., 15):
>
> ```
> (puma) [lookitsme@junonia ~]$ for i in $(seq 8 15) ; do sacct --user=<User C> --format=Start -X | grep T$i: | wc -l ; done 
>    0 0 240 206 200 200 193 969
>```
>
> vs. `<User A>`
>
>```
> (puma) [lookitsme@junonia ~]$ for i in $(seq 8 15) ; do sacct --user=<User A> --format=Start -X | grep T$i: | wc -l ; done 
> 0 0 120 120 110 110 110 70
>```
>
> Doing some poking, I'm wondering if this is in part due to `<User C>` submitting to both the standard and windfall partitions? Looking only at windfall, the numbers are roughly cut in half:
>
>``` 
> (puma) [lookitsme@junonia ~]$ for i in $(seq 8 15) ; do sacct --user=<User C> --format=Start,Partition -X | grep -v standard | grep T$i: | wc -l ; done 
>    0 0 120 86 90 90 83 339
>```
>
> So, to check, is this per-user backfill limitation on a per-partition basis? `<Coworker>` found in the man page for slurm.conf the option `bf_max_job_user_part` (which we don't have set in our conf file), so I'm curious if there's some interplay between or inheritance from the `bf_max_job_user` parameter that essentially benefits users who are using multiple partitions.
> 
> 2) Is there a way to prioritize qos/high_priority jobs when it comes to backfill? 
>
> 3) How does preemption work with backfill? I'm guessing high_priority jobs being considered for backfill do not preempt windfall jobs given the language Slurm uses:
>
>>    Backfill scheduling will start lower priority jobs if doing so does not delay the expected start time of any higher priority jobs.
>
>which to me suggests this mechanism is strictly for "gap filling", but wanted to verify. 
>
> 4) Related to the above, would this explain why I saw zero preemption with `<User C>`'s jobs (checking, there were zero preemptions for their many, many windfall jobs)? Essentially, since backfill is designed for minimal interference and is mostly a gap-filling mechanism, windfall jobs that are scheduled via backfill are using resources the scheduler has already deemed unusuable by the standard scheduling mechanism for a period of time due to a pending reservation. So regularly scheduled jobs won't run there, plus other backfill jobs won't preempt them?
>
> 5) Just in terms of priority in general, I noticed yesterday during a test that when I submitted high priority jobs and windfall jobs that they all started with the same priority level:
>
>```
>(puma) [lookitsme@junonia denard]$ usqueue
>JOBID PARTITION       NAME      PRIORITY       USER         ACCOUNT ST CPUS MIN_M NOD        TRES_PER_NODE SUBMIT_TIME NODELIST(REASON) TIME_LEFT
>17151509 high_prio scale.slur       5001  lookitsme lookitsmyaccout PD    2    5G   1                  N/A 2025-10-20T14:49:20 (Priority) 1:00:00
>17151508 high_prio no_scale.s       5001  lookitsme lookitsmyaccout PD    2    5G   1                  N/A 2025-10-20T14:49:20 (Priority) 1:00:00
>17151510  windfall windfall.s       5001  lookitsme        windfall PD    2    5G   1                  N/A 2025-10-20T14:49:20 (Priority) 1:00:00
>```
>
> I just wanted to verify that this is expected behavior. For some reason I thought higher priority jobs came into the queue with higher priority than other partitions (I could be misremembering), so I just want to make sure all is well. 

Sys admin response:

> That's a lot of questions 😂 if I miss anything just let me know.
> 
> Yes, your summary of what backfill is and how it's implemented in slurm is accurate. I didn't the scheduler /hi pri load yesterday, but I guess that makes a little bit more sense. I had thought maybe the scheduler was doing him a disservice by classifying his job as backfill, preventing him from pre-empting windfall jobs but I guess if it wasn't completing cycles and/or there was that much job pressure then it was probably doing him a favor.
>
> On to the questions

> 1. Yes, the backfill scheduler runs per partition. `Bf_max_job_user` is supposed to be the limit across all partitions so I'd expect that to have been more restrictive on a user submitting to multiple partitions than `bf_max_job_user_part` would be.
> 2. Since backfill evaluates per partition, I'd expect it to use the partition priority. We don't actually use partition priority, we just use the priority in the QoS for the partition so the partitions themselves all have the same priority. Subtle distinction that doesn't really have an impact on job scheduling,  but it's possible that adding a priority to the partitions (which gets factored into the actual job priority) could prioritize high priority in the backfill scheduler.
> 3. That's correct, backfill is gap filling, like Tetris. Without backfill, it's strict fifo. So for instance, there are 4 total cores, three are in use for the next 24 hours and 1 is free. If the next job in the queue needs 4 cores and the one behind it needs 1 core for 1 hour, the 1 core/1 hour job has to wait until the one in front of it runs before it can be scheduled. In theory backfill should only make things more efficient.
> 4. Yes, your understanding is correct.
> 5. That's not exactly expected. The partitions all have the same priority, 5000. The QoS gets added onto that, ie
>
>```
> sacctmgr: show qos part_qos_windfall format=name,priority
> Name|Priority|
> part_qos_windfall|1|
> sacctmgr: show qos part_qos_standard format=name,priority
> Name|Priority|
> part_qos_standard|3|
> sacctmgr: show qos user_qos_<lookitsme> format=name,priority
> Name|Priority|
> user_qos_<lookitsme>|7|
>```


> So, I'd have expected your high priority job to be 5007. That one might be worth asking <Other sys admin> to open up a ticket with sched md on.


### tldr; 

1. Slurm uses a mechanism called backfill to slot small/windfall jobs into the system to make sure things are running at full capacity. These jobs only run when resources are available that won't get prioritized in front of other higher priority jobs. Small windfall jobs run in this category.

2. The system is configured for fairshare such that only 10 backfill jobs are considered per user (across all partitions). So this means that if a user's jobs are being considered for backfill, only the 10 highest priority backfill jobs are considered while the rest are ignored. This means, if a user has small jobs pending for rare resources, they'll brick up their own backfill queue and the small jobs pending below them won't be considered.

3. Backfill information can be found in `slurm.conf` set in `SchedulerParameters`. Refresher: you can find `slurm.conf` with `env | grep CONF`. Some of our settings:

    |<div style="width:200px">Parameter</div>|Value|What it does|
    |-|-|-|
    |`partition_job_depth`|`200`|Specifies how many jobs are tested in any single partition, default value is 0 (no limit). Ours is set to 200, so each scheduling cycle, only the top 200 jobs are considered.|
    |`bf_interval`|`300`|How frequently, in seconds, backfill is run. So jobs are evaluated on our system every 5 minutes|
    |`bf_max_job_user`|`10`|How many jobs per user are considered for backfill across all partitions|
