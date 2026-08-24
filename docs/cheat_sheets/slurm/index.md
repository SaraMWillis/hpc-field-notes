# Slurm

## sacct

### Formatting

Used with `--format`

```

Account             AdminComment        AllocCPUS           AllocNodes
AllocTRES           AssocID             AveCPU              AveCPUFreq
AveDiskRead         AveDiskWrite        AvePages            AveRSS
AveVMSize           BlockID             Cluster             Comment
Constraints         ConsumedEnergy      ConsumedEnergyRaw   Container
CPUTime             CPUTimeRAW          DBIndex             DerivedExitCode
Elapsed             ElapsedRaw          Eligible            End
Exclusive           ExitCode            Extra               FailedNode
Flags               GID                 Group               JobID
JobIDRaw            JobName             Layout              Licenses
MaxDiskRead         MaxDiskReadNode     MaxDiskReadTask     MaxDiskWrite
MaxDiskWriteNode    MaxDiskWriteTask    MaxPages            MaxPagesNode
MaxPagesTask        MaxRSS              MaxRSSNode          MaxRSSTask
MaxVMSize           MaxVMSizeNode       MaxVMSizeTask       McsLabel
MinCPU              MinCPUNode          MinCPUTask          NCPUS
NNodes              NodeList            NTasks              OriginalSLUID
OverSubscribe       Partition           Planned             PlannedCPU
PlannedCPURAW       Priority            QOS                 QOSRAW
QOSREQ              Reason              ReqCPUFreq          ReqCPUFreqGov
ReqCPUFreqMax       ReqCPUFreqMin       ReqCPUS             ReqMem
ReqNodes            ReqReservation      ReqTRES             Reservation
ReservationId       Restarts            SegmentSize         SLUID
Start               State               StdErr              StdIn
StdOut              Submit              SubmitLine          Suspended
SystemComment       SystemCPU           Timelimit           TimelimitRaw
TotalCPU            TRESUsageInAve      TRESUsageInMax      TRESUsageInMaxNode
TRESUsageInMaxTask  TRESUsageInMin      TRESUsageInMinNode  TRESUsageInMinTask
TRESUsageInTot      TRESUsageOutAve     TRESUsageOutMax     TRESUsageOutMaxNode
TRESUsageOutMaxTask TRESUsageOutMin     TRESUsageOutMinNode TRESUsageOutMinTask
TRESUsageOutTot     UID                 User                UserCPU
WCKey               WCKeyID             WorkDir
```

## squeue

### Formatting

Used with `--Format`

```
Account           AccrueTime       admin_comment     AllocNodes
AllocSID          ArrayJobID       ArrayTaskID       AssocID
BatchFlag         BatchHost        BoardsPerNode     BurstBuffer
BurstBufferState  Cluster          ClusterFeature    Command
Comment           Contiguous       Container         ContainerID
ContainerType     Cores            CoreSpec          CPUFreq
cpus-per-task     cpus-per-tres    CronJob           Deadline
DelayBoot         Dependency       DerivedEC         EligibleTime
EndTime           Exclusive        ExcNodes          exit_code
Feature           GroupID          GroupName         HetJobID
HetJobIDSet       HetJobOffset     JobArrayID        JobID
LastSchedEval     Licenses         LicensesAlloc     MaxCPUs
MaxNodes          MCSLabel         mem-per-tres      MinCpus
MinMemory         MinTime          MinTmpDisk        Name
Network           Nice             NodeList          Nodes
NTPerBoard        NTPerCore        NTPerNode         NTPerSocket
NumCPUs           NumNodes         NumTasks          Origin
OriginRaw         OverSubscribe    Partition         PendingTime
PreemptTime       Prefer           Priority          PriorityLong
Profile           QOS              Reason            ReasonList
Reboot            ReqNodes         ReqSwitch         Requeue
Reservation       ResizeTime       RestartCnt        ResvPort
SchedNodes        SCT              SegmentSize       SiblingsActive
SiblingsActiveRaw SiblingsViable   SiblingsViableRaw Sluid
Sockets           SPerBoard        StartTime         State
StateCompact      STDERR           STDIN             STDOUT
StepID            StepName         StepState         SubmitTime
system_comment    Threads          TimeLeft          TimeLimit
TimeUsed          tres-alloc       tres-bind         tres-freq
tres-per-job      tres-per-node    tres-per-socket   tres-per-step
tres-per-task     UserID           UserName          Wait4Switch
WCKey             WorkDir
```

## sacctmgr

Check group associations

```
sacctmgr show assoc where account=mygroup format=Account,Partition,User,QOS
```