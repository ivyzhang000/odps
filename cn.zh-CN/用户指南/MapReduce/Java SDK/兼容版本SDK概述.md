# 兼容版本SDK概述 {#concept_ytg_mcg_vdb .concept}

MaxCompute 兼容版本的 MapReduce 与 Hadoop MapReduce 兼容性的详细列表，如下表所示：

|类型|接口|是否兼容|
|:-|:-|:---|
|Mapper|void map\(KEYIN key, VALUEIN value, org.apache.hadoop.mapreduce.Mapper.Context context\)|是|
|Mapper|void run\(org.apache.hadoop.mapreduce.Mapper.Context context\)|是|
|Mapper|void setup\(org.apache.hadoop.mapreduce.Mapper.Context context\)|是|
|Reducer|void cleanup\(org.apache.hadoop.mapreduce.Reducer.Context context\)|是|
|Reducer|void reduce\(KEYIN key, VALUEIN value, org.apache.hadoop.mapreduce.Reducer.Context context\)|是|
|Reducer|void run\(org.apache.hadoop.mapreduce.Reducer.Context context\)|是|
|Reducer|void setup\(org.apache.hadoop.mapreduce.Reducer.Context context\)|是|
|Partitioner|int getPartition\(KEY key, VALUE value, int numPartitions\)|是|
|MapContext（继承TaskInputOutputContext）|InputSplit getInputSplit\(\)|否，抛异常|
|ReduceContext|nextKey\(\)|是|
|ReduceContext|getValues\(\)|是|
|TaskInputOutputContext|getCurrentKey\(\)|是|
|TaskInputOutputContext|getCurrentValue\(\)|是|
|TaskInputOutputContext|getOutputCommitter\(\)|否，抛异常|
|TaskInputOutputContext|nextKeyValue\(\)|是|
|TaskInputOutputContext|write\(KEYOUT key, VALUEOUT value\)|是|
|TaskAttemptContext|getCounter\(Enum<?\> counterName\)|是|
|TaskAttemptContext|getCounter\(String groupName, String counterName\)|是|
|TaskAttemptContext|setStatus\(String msg\)|空实现|
|TaskAttemptContext|getStatus\(\)|空实现|
|TaskAttemptContext|getTaskAttemptID\(\)|否，抛异常|
|TaskAttemptContext|getProgress\(\)|否，抛异常|
|TaskAttemptContext|progress\(\)|是|
|Job|addArchiveToClassPath\(Path archive\)|否|
|Job|addCacheArchive\(URI uri\)|否|
|Job|addCacheFile\(URI uri\)|否|
|Job|addFileToClassPath\(Path file\)|否|
|Job|cleanupProgress\(\)|否|
|Job|createSymlink\(\)|否，抛异常|
|Job|failTask\(TaskAttemptID taskId\)|否|
|Job|getCompletionPollInterval\(Configuration conf\)|空实现|
|Job|getCounters\(\)|是|
|Job|getFinishTime\(\)|是|
|Job|getHistoryUrl\(\)|是|
|Job|getInstance\(\)|是|
|Job|getInstance\(Cluster ignored\)|是|
|Job|getInstance\(Cluster ignored, Configuration conf\)|是|
|Job|getInstance\(Configuration conf\)|是|
|Job|getInstance\(Configuration conf, String jobName\)|空实现|
|Job|getInstance\(JobStatus status, Configuration conf\)|否，抛异常|
|Job|getJobFile\(\)|否，抛异常|
|Job|getJobName\(\)|空实现|
|Job|getJobState\(\)|否，抛异常|
|Job|getPriority\(\)|否，抛异常|
|Job|getProgressPollInterval\(Configuration conf\)|空实现|
|Job|getReservationId\(\)|否，抛异常|
|Job|getSchedulingInfo\(\)|否，抛异常|
|Job|getStartTime\(\)|是|
|Job|getStatus\(\)|否，抛异常|
|Job|getTaskCompletionEvents\(int startFrom\)|否，抛异常|
|Job|getTaskCompletionEvents\(int startFrom, int numEvents\)|否，抛异常|
|Job|getTaskDiagnostics\(TaskAttemptID taskid\)|否，抛异常|
|Job|getTaskOutputFilter\(Configuration conf\)|否，抛异常|
|Job|getTaskReports\(TaskType type\)|否，抛异常|
|Job|getTrackingURL\(\)|是|
|Job|isComplete\(\)|是|
|Job|isRetired\(\)|否，抛异常|
|Job|isSuccessful\(\)|是|
|Job|isUber\(\)|空实现|
|Job|killJob\(\)|是|
|Job|killTask\(TaskAttemptID taskId\)|否|
|Job|mapProgress\(\)|是|
|Job|monitorAndPrintJob\(\)|是|
|Job|reduceProgress\(\)|是|
|Job|setCacheArchives\(URI\[\] archives\)|否，抛异常|
|Job|setCacheFiles\(URI\[\] files\)|否，抛异常|
|Job|setCancelDelegationTokenUponJobCompletion\(boolean value\)|否，抛异常|
|Job|setCombinerClass\(Class<? extends Reducer\> cls\)|是|
|Job|setCombinerKeyGroupingComparatorClass\(Class<? extends RawComparator\> cls\)|是|
|Job|setGroupingComparatorClass\(Class<? extends RawComparator\> cls\)|是|
|Job|setInputFormatClass\(Class<? extends InputFormat\> cls\)|空实现|
|Job|setJar\(String jar\)|是|
|Job|setJarByClass\(Class<?\> cls\)|是|
|Job|setJobName\(String name\)|空实现|
|Job|setJobSetupCleanupNeeded\(boolean needed\)|空实现|
|Job|setMapOutputKeyClass\(Class<?\> theClass\)|是|
|Job|setMapOutputValueClass\(Class<?\> theClass\)|是|
|Job|setMapperClass\(Class<? extends Mapper\> cls\)|是|
|Job|setMapSpeculativeExecution\(boolean speculativeExecution\)|空实现|
|Job|setMaxMapAttempts\(int n\)|空实现|
|Job|setMaxReduceAttempts\(int n\)|空实现|
|Job|setNumReduceTasks\(int tasks\)|是|
|Job|setOutputFormatClass\(Class<? extends OutputFormat\> cls\)|否，抛异常|
|Job|setOutputKeyClass\(Class<?\> theClass\)|是|
|Job|setOutputValueClass\(Class<?\> theClass\)|是|
|Job|setPartitionerClass\(Class<? extends Partitioner\> cls\)|是|
|Job|setPriority\(JobPriority priority\)|否，抛异常|
|Job|setProfileEnabled\(boolean newValue\)|空实现|
|Job|setProfileParams\(String value\)|空实现|
|Job|setProfileTaskRange\(boolean isMap, String newValue\)|空实现|
|Job|setReducerClass\(Class<? extends Reducer\> cls\)|是|
|Job|setReduceSpeculativeExecution\(boolean speculativeExecution\)|空实现|
|Job|setReservationId\(ReservationId reservationId\)|否，抛异常|
|Job|setSortComparatorClass\(Class<? extends RawComparator\> cls\)|否，抛异常|
|Job|setSpeculativeExecution\(boolean speculativeExecution\)|是|
|Job|setTaskOutputFilter\(Configuration conf, org.apache.hadoop.mapreduce.Job.TaskStatusFilter newValue\)|否，抛异常|
|Job|setupProgress\(\)|否，抛异常|
|Job|setUser\(String user\)|空实现|
|Job|setWorkingDirectory\(Path dir\)|空实现|
|Job|submit\(\)|是|
|Job|toString\(\)|否，抛异常|
|Job|waitForCompletion\(boolean verbose\)|是|
|Task Execution & Environment|mapreduce.map.java.opts|空实现|
|Task Execution & Environment|mapreduce.reduce.java.opts|空实现|
|Task Execution & Environment|mapreduce.map.memory.mb|空实现|
|Task Execution & Environment|mapreduce.reduce.memory.mb|空实现|
|Task Execution & Environment|mapreduce.task.io.sort.mb|空实现|
|Task Execution & Environment|mapreduce.map.sort.spill.percent|空实现|
|Task Execution & Environment|mapreduce.task.io.soft.factor|空实现|
|Task Execution & Environment|mapreduce.reduce.merge.inmem.thresholds|空实现|
|Task Execution & Environment|mapreduce.reduce.shuffle.merge.percent|空实现|
|Task Execution & Environment|mapreduce.reduce.shuffle.input.buffer.percent|空实现|
|Task Execution & Environment|mapreduce.reduce.input.buffer.percent|空实现|
|Task Execution & Environment|mapreduce.job.id|空实现|
|Task Execution & Environment|mapreduce.job.jar|空实现|
|Task Execution & Environment|mapreduce.job.local.dir|空实现|
|Task Execution & Environment|mapreduce.task.id|空实现|
|Task Execution & Environment|mapreduce.task.attempt.id|空实现|
|Task Execution & Environment|mapreduce.task.is.map|空实现|
|Task Execution & Environment|mapreduce.task.partition|空实现|
|Task Execution & Environment|mapreduce.map.input.file|空实现|
|Task Execution & Environment|mapreduce.map.input.start|空实现|
|Task Execution & Environment|mapreduce.map.input.length|空实现|
|Task Execution & Environment|mapreduce.task.output.dir|空实现|
|JobClient|cancelDelegationToken\(Token <DelegationTokenIdentifier\> token\)|否，抛异常|
|JobClient|close\(\)|空实现|
|JobClient|displayTasks\(JobID jobId, String type, String state\)|否，抛异常|
|JobClient|getAllJobs\(\)|否，抛异常|
|JobClient|getCleanupTaskReports\(JobID jobId\)|否，抛异常|
|JobClient|getClusterStatus\(\)|否，抛异常|
|JobClient|getClusterStatus\(boolean detailed\)|否，抛异常|
|JobClient|getDefaultMaps\(\)|否，抛异常|
|JobClient|getDefaultReduces\(\)|否，抛异常|
|JobClient|getDelegationToken\(Text renewer\)|否，抛异常|
|JobClient|getFs\(\)|否，抛异常|
|JobClient|getJob\(JobID jobid\)|否，抛异常|
|JobClient|getJob\(String jobid\)|否，抛异常|
|JobClient|getJobsFromQueue\(String queueName\)|否，抛异常|
|JobClient|getMapTaskReports\(JobID jobId\)|否，抛异常|
|JobClient|getMapTaskReports\(String jobId\)|否，抛异常|
|JobClient|getQueueAclsForCurrentUser\(\)|否，抛异常|
|JobClient|getQueueInfo\(String queueName\)|否，抛异常|
|JobClient|getQueues\(\)|否，抛异常|
|JobClient|getReduceTaskReports\(JobID jobId\)|否，抛异常|
|JobClient|getReduceTaskReports\(String jobId\)|否，抛异常|
|JobClient|getSetupTaskReports\(JobID jobId\)|否，抛异常|
|JobClient|getStagingAreaDir\(\)|否，抛异常|
|JobClient|getSystemDir\(\)|否，抛异常|
|JobClient|getTaskOutputFilter\(\)|否，抛异常|
|JobClient|getTaskOutputFilter\(JobConf job\)|否，抛异常|
|JobClient|init\(JobConf conf\)|否，抛异常|
|JobClient|isJobDirValid\(Path jobDirPath, FileSystem fs\)|否，抛异常|
|JobClient|jobsToComplete\(\)|否，抛异常|
|JobClient|monitorAndPrintJob\(JobConf conf, RunningJob job\)|否，抛异常|
|JobClient|renewDelegationToken\(Token<DelegationTokenIdentifier\> token\)|否，抛异常|
|JobClient|run\(String\[\] argv\)|否，抛异常|
|JobClient|runJob\(JobConf job\)|是|
|JobClient|setTaskOutputFilter\(JobClient.TaskStatusFilter newValue\)|否，抛异常|
|JobClient|setTaskOutputFilter\(JobConf job, JobClient.TaskStatusFilter newValue\)|否，抛异常|
|JobClient|submitJob\(JobConf job\)|是|
|JobClient|submitJob\(String jobFile\)|否，抛异常|
|JobConf|deleteLocalFiles\(\)|否，抛异常|
|JobConf|deleteLocalFiles\(String subdir\)|否，抛异常|
|JobConf|normalizeMemoryConfigValue\(long val\)|空实现|
|JobConf|setCombinerClass\(Class<? extends Reducer\> theClass\)|是|
|JobConf|setCompressMapOutput\(boolean compress\)|空实现|
|JobConf|setInputFormat\(Class<? extends InputFormat\> theClass\)|否，抛异常|
|JobConf|setJar\(String jar\)|否，抛异常|
|JobConf|setJarByClass\(Class cls\)|否，抛异常|
|JobConf|setJobEndNotificationURI\(String uri\)|否，抛异常|
|JobConf|setJobName\(String name\)|空实现|
|JobConf|setJobPriority\(JobPriority prio\)|否，抛异常|
|JobConf|setKeepFailedTaskFiles\(boolean keep\)|否，抛异常|
|JobConf|setKeepTaskFilesPattern\(String pattern\)|否，抛异常|
|JobConf|setKeyFieldComparatorOptions\(String keySpec\)|否，抛异常|
|JobConf|setKeyFieldPartitionerOptions\(String keySpec\)|否，抛异常|
|JobConf|setMapDebugScript\(String mDbgScript\)|空实现|
|JobConf|setMapOutputCompressorClass\(Class<? extends CompressionCodec\> codecClass\)|空实现|
|JobConf|setMapOutputKeyClass\(Class<?\> theClass\)|是|
|JobConf|setMapOutputValueClass\(Class<?\> theClass\)|是|
|JobConf|setMapperClass\(Class<? extends Mapper\> theClass\)|是|
|JobConf|setMapRunnerClass\(Class<? extends MapRunnable\> theClass\)|否，抛异常|
|JobConf|setMapSpeculativeExecution\(boolean speculativeExecution\)|空实现|
|JobConf|setMaxMapAttempts\(int n\)|空实现|
|JobConf|setMaxMapTaskFailuresPercent\(int percent\)|空实现|
|JobConf|setMaxPhysicalMemoryForTask\(long mem\)|空实现|
|JobConf|setMaxReduceAttempts\(int n\)|空实现|
|JobConf|setMaxReduceTaskFailuresPercent\(int percent\)|空实现|
|JobConf|setMaxTaskFailuresPerTracker\(int noFailures\)|空实现|
|JobConf|setMaxVirtualMemoryForTask\(long vmem\)|空实现|
|JobConf|setMemoryForMapTask\(long mem\)|是|
|JobConf|setMemoryForReduceTask\(long mem\)|是|
|JobConf|setNumMapTasks\(int n\)|是|
|JobConf|setNumReduceTasks\(int n\)|是|
|JobConf|setNumTasksToExecutePerJvm\(int numTasks\)|空实现|
|JobConf|setOutputCommitter\(Class<? extends OutputCommitter\> theClass\)|否，抛异常|
|JobConf|setOutputFormat\(Class<? extends OutputFormat\> theClass\)|空实现|
|JobConf|setOutputKeyClass\(Class<?\> theClass\)|是|
|JobConf|setOutputKeyComparatorClass\(Class<? extends RawComparator\> theClass\)|否，抛异常|
|JobConf|setOutputValueClass\(Class<?\> theClass\)|是|
|JobConf|setOutputValueGroupingComparator\(Class<? extends RawComparator\> theClass\)|否，抛异常|
|JobConf|setPartitionerClass\(Class<? extends Partitioner\> theClass\)|是|
|JobConf|setProfileEnabled\(boolean newValue\)|空实现|
|JobConf|setProfileParams\(String value\)|空实现|
|JobConf|setProfileTaskRange\(boolean isMap, String newValue\)|空实现|
|JobConf|setQueueName\(String queueName\)|否，抛异常|
|JobConf|setReduceDebugScript\(String rDbgScript\)|空实现|
|JobConf|setReducerClass\(Class<? extends Reducer\> theClass\)|是|
|JobConf|setReduceSpeculativeExecution\(boolean speculativeExecution\)|空实现|
|JobConf|setSessionId\(String sessionId\)|空实现|
|JobConf|setSpeculativeExecution\(boolean speculativeExecution\)|否，抛异常|
|JobConf|setUseNewMapper\(boolean flag\)|是|
|JobConf|setUseNewReducer\(boolean flag\)|是|
|JobConf|setUser\(String user\)|空实现|
|JobConf|setWorkingDirectory\(Path dir\)|空实现|
|FileInputFormat|不涉及|否，抛异常|
|TextInputFormat|不涉及|是|
|InputSplit|mapred.min.split.size.|否，抛异常|
|FileSplit|map.input.file|否，抛异常|
|RecordWriter|不涉及|否，抛异常|
|RecordReader|不涉及|否，抛异常|
|OutputFormat|不涉及|否，抛异常|
|OutputCommitter|abortJob\(JobContext jobContext, int status\)|否，抛异常|
|OutputCommitter|abortJob\(JobContext context, JobStatus.State runState\)|否，抛异常|
|OutputCommitter|abortTask\(TaskAttemptContext taskContext\)|否，抛异常|
|OutputCommitter|abortTask\(TaskAttemptContext taskContext\)|否，抛异常|
|OutputCommitter|cleanupJob\(JobContext jobContext\)|否，抛异常|
|OutputCommitter|cleanupJob\(JobContext context\)|否，抛异常|
|OutputCommitter|commitJob\(JobContext jobContext\)|否，抛异常|
|OutputCommitter|commitJob\(JobContext context\)|否，抛异常|
|OutputCommitter|commitTask\(TaskAttemptContext taskContext\)|否，抛异常|
|OutputCommitter|needsTaskCommit\(TaskAttemptContext taskContext\)|否，抛异常|
|OutputCommitter|needsTaskCommit\(TaskAttemptContext taskContext\)|否，抛异常|
|OutputCommitter|setupJob\(JobContext jobContext\)|否，抛异常|
|OutputCommitter|setupJob\(JobContext jobContext\)|否，抛异常|
|OutputCommitter|setupTask\(TaskAttemptContext taskContext\)|否，抛异常|
|OutputCommitter|setupTask\(TaskAttemptContext taskContext\)|否，抛异常|
|Counter|getDisplayName\(\)|是|
|Counter|getName\(\)|是|
|Counter|getValue\(\)|是|
|Counter|increment\(long incr\)|是|
|Counter|setValue\(long value\)|是|
|Counter|setDisplayName\(String displayName\)|是|
|DistributedCache|CACHE\_ARCHIVES|否，抛异常|
|DistributedCache|CACHE\_ARCHIVES\_SIZES|否，抛异常|
|DistributedCache|CACHE\_ARCHIVES\_TIMESTAMPS|否，抛异常|
|DistributedCache|CACHE\_FILES|否，抛异常|
|DistributedCache|CACHE\_FILES\_SIZES|否，抛异常|
|DistributedCache|CACHE\_FILES\_TIMESTAMPS|否，抛异常|
|DistributedCache|CACHE\_LOCALARCHIVES|否，抛异常|
|DistributedCache|CACHE\_LOCALFILES|否，抛异常|
|DistributedCache|CACHE\_SYMLINK|否，抛异常|
|DistributedCache|addArchiveToClassPath\(Path archive, Configuration conf\)|否，抛异常|
|DistributedCache|addArchiveToClassPath\(Path archive, Configuration conf, FileSystem fs\)|否，抛异常|
|DistributedCache|addCacheArchive\(URI uri, Configuration conf\)|否，抛异常|
|DistributedCache|addCacheFile\(URI uri, Configuration conf\)|否，抛异常|
|DistributedCache|addFileToClassPath\(Path file, Configuration conf\)|否，抛异常|
|DistributedCache|addFileToClassPath\(Path file, Configuration conf, FileSystem fs\)|否，抛异常|
|DistributedCache|addLocalArchives\(Configuration conf, String str\)|否，抛异常|
|DistributedCache|addLocalFiles\(Configuration conf, String str\)|否，抛异常|
|DistributedCache|checkURIs\(URI\[\] uriFiles, URI\[\] uriArchives\)|否，抛异常|
|DistributedCache|createAllSymlink\(Configuration conf, File jobCacheDir, File workDir\)|否，抛异常|
|DistributedCache|createSymlink\(Configuration conf\)|否，抛异常|
|DistributedCache|getArchiveClassPaths\(Configuration conf\)|否，抛异常|
|DistributedCache|getArchiveTimestamps\(Configuration conf\)|否，抛异常|
|DistributedCache|getCacheArchives\(Configuration conf\)|否，抛异常|
|DistributedCache|getCacheFiles\(Configuration conf\)|否，抛异常|
|DistributedCache|getFileClassPaths\(Configuration conf\)|否，抛异常|
|DistributedCache|getFileStatus\(Configuration conf, URI cache\)|否，抛异常|
|DistributedCache|getFileTimestamps\(Configuration conf\)|否，抛异常|
|DistributedCache|getLocalCacheArchives\(Configuration conf\)|否，抛异常|
|DistributedCache|getLocalCacheFiles\(Configuration conf\)|否，抛异常|
|DistributedCache|getSymlink\(Configuration conf\)|否，抛异常|
|DistributedCache|getTimestamp\(Configuration conf, URI cache\)|否，抛异常|
|DistributedCache|setArchiveTimestamps\(Configuration conf, String timestamps\)|否，抛异常|
|DistributedCache|setCacheArchives\(URI\[\] archives, Configuration conf\)|否，抛异常|
|DistributedCache|setCacheFiles\(URI\[\] files, Configuration conf\)|否，抛异常|
|DistributedCache|setFileTimestamps\(Configuration conf, String timestamps\)|否，抛异常|
|DistributedCache|setLocalArchives\(Configuration conf, String str\)|否，抛异常|
|DistributedCache|setLocalFiles\(Configuration conf, String str\)|否，抛异常|
|IsolationRunner|不涉及|否，抛异常|
|Profiling|不涉及|空实现|
|Debugging|不涉及|空实现|
|Data Compression|不涉及|是|
|Skipping Bad Records|不涉及|否，抛异常|
|Job Authorization|mapred.acls.enabled|否，抛异常|
|Job Authorization|mapreduce.job.acl-view-job|否，抛异常|
|Job Authorization|mapreduce.job.acl-modify-job|否，抛异常|
|Job Authorization|mapreduce.cluster.administrators|否，抛异常|
|Job Authorization|mapred.queue.queue-name.acl-administer-jobs|否，抛异常|
|MultipleInputs|不涉及|否，抛异常|
|Multi\{anchor:\_GoBack\}pleOutputs|不涉及|是|
|org.apache.hadoop.mapreduce.lib.db|不涉及|否，抛异常|
|org.apache.hadoop.mapreduce.security|不涉及|否，抛异常|
|org.apache.hadoop.mapreduce.lib.jobcontrol|不涉及|否，抛异常|
|org.apache.hadoop.mapreduce.lib.chain|不涉及|否，抛异常|
|org.apache.hadoop.mapreduce.lib.db|不涉及|否，抛异常|

