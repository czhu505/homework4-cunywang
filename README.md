## DATA622 HW #4
- Assigned on October 25, 2018
- Due on October 14, 2018 11:59 PM EST
- 15 points possible, worth 15% of your final grade

### Instructions:

Use the two resources below to complete both the critical thinking and applied parts of this assignment.

1. Listen to all the lectures in Udacity's [Intro to Hadoop and Mapreduce](https://www.udacity.com/course/intro-to-hadoop-and-mapreduce--ud617) course.  

2. Read [Hadoop A Definitive Guide Edition 4]( http://javaarm.com/file/apache/Hadoop/books/Hadoop-The.Definitive.Guide_4.edition_a_Tom.White_April-2015.pdf), Part I Chapters 1 - 3.

### Critical Thinking (10 points total)

Submit your answers by modifying this README.md file.

1. (1 points) What is Hadoop 1's single point of failure and why is this critical?  How is this alleviated in Hadoop 2?

The NameNode is the single point of failure in Hadoop 1.0. Each cluster has a single NameNode and if that machine is not available, the whole cluster will be not available.

This impacts the total availability of HDFS in two ways:
For any unplanned event such as machine crashes, the whole cluster is not available until the Name node is brought up manually.
For planned maintenance such as Hardware or Software upgrades on NameNode would result in cluster unavailability.

In Hadoop 2.0, HDFS High Availability feature addresses the above problem, by providing an option to run two NameNodes in the same cluster in an Active/Passive configuration with a hot standby.

This allows fast Failover to a new NameNode for any machine crashes or administrator initiated fail-over for any planned maintenance activities.

https://data-flair.training/forums/topic/what-is-single-point-of-failure-in-hadoop-1-and-how-it-is-resolved-in-hadoop-2/






2. (2 points) What happens when a data node fails?

Single point of failure problem implies that if the NameNode fails, then that Hadoop Cluster will become out of service. This may be a rare scenario because everyone uses high configuration hardware for NameNode.

In Hadoop 1.0 if NameNode failure occurred, the administrator would need to recover the data from secondary namenode.
With the introduction of High availability in Hadoop 2.0, the SPOF problem has been solved by adding 2 Namenodes, one in active mode and another in standby mode. Both the NamNodes have same data and in failure on active Namenode, the standby Namenode automatically takes over. It requires no manual intervention as the architecture is itself designed so. This also ensures availability with no downtime.

https://data-flair.training/forums/topic/explain-single-point-of-failure-in-hadoop/


3. (1 point) What is a daemon?  Describe the role task trackers and job trackers play in the Hadoop environment.

Job Tracker –
JobTracker process runs on a separate node and not usually on a DataNode.
JobTracker is an essential Daemon for MapReduce execution in MRv1. It is replaced by ResourceManager/ApplicationMaster in MRv2.
JobTracker receives the requests for MapReduce execution from the client.
JobTracker talks to the NameNode to determine the location of the data.
JobTracker finds the best TaskTracker nodes to execute tasks based on the data locality (proximity of the data) and the available slots to execute a task on a given node.
JobTracker monitors the individual TaskTrackers and the submits back the overall status of the job back to the client.
JobTracker process is critical to the Hadoop cluster in terms of MapReduce execution.
When the JobTracker is down, HDFS will still be functional but the MapReduce execution can not be started and the existing MapReduce jobs will be halted.

TaskTracker –
TaskTracker runs on DataNode. Mostly on all DataNodes.
TaskTracker is replaced by Node Manager in MRv2.
Mapper and Reducer tasks are executed on DataNodes administered by TaskTrackers.
TaskTrackers will be assigned Mapper and Reducer tasks to execute by JobTracker.
TaskTracker will be in constant communication with the JobTracker signalling the progress of the task in execution.
TaskTracker failure is not considered fatal. When a TaskTracker becomes unresponsive, JobTracker will assign the task executed by the TaskTracker to another node.

http://hadoopinrealworld.com/jobtracker-and-tasktracker/


4. (1 point) Why is Cloudera's VM considered pseudo distributed computing?  How is it different from a true Hadoop cluster computing?

Coming to standalone and pseudo distributed mode ,one aspect u should consider is size of the file and second being actual difference in standalone and pseudo distributed mode.

In standalone mode there is no concept of HDFS,data is not copied to hadoop distributed file system (obviously time saved).Where as in pseudo distributed mode ,hdfs involved which need to be copied with the data that's need to be processed.
Small size data files better to use traditional file processing and if the file size become huge and huge ,hadoop framework gives better processing time! 

https://stackoverflow.com/questions/10402625/performance-comparison-between-hadoop-pseudo-distributed-operation-and-standalon



5. (1 point) What is Hadoop streaming? What is the Hadoop Ecosystem?

Hadoop streaming is a utility that comes with the Hadoop distribution. The utility allows you to create and run Map/Reduce jobs with any executable or script as the mapper and/or the reducer. 

Hadoop Ecosystem is a platform or framework which solves big data problems. You can consider it as a suite which encompasses a number of services (ingesting, storing, analyzing and maintaining) inside it.



6. (1 point) During a reducer job, why do we need to know the current key, current value, previous key, and cumulative value, but NOT the previous value?

In the reducer, every key have a value along with it. And the keys have been sorted. The tracker keeps tracking the changing keys and updating the last value based on the calculation. That's why the previous value is not useful.



7. (3 points) A large international company wants to use Hadoop MapReduce to calculate the # of sales by location by day.  The logs data has one entry per location per day per sale.  Describe how MapReduce will work in this scenario, using key words like: intermediate records, shuffle and sort, mappers, reducers, sort, key/value, task tracker, job tracker.  

Applications typically implement the Mapper and Reducer interfaces to provide the map and reduce methods. Mapper maps input key/value pairs (loaction/sales) to a set of intermediate key/value pairs. Reducer has 3 primary phases: shuffle, sort and reduce.Shuffle:
Input to the Reducer is the sorted output of the mappers. In this phase the framework fetches the relevant partition of the output of all the mappers(loaction/sales), via HTTP. Sort:The framework groups Reducer inputs by keys (since different mappers may have output the same key) in this stage. The shuffle and sort phases occur simultaneously; while map-outputs are being fetched they are merged. Reduce: 
In this phase the reduce(WritableComparable, Iterable<Writable>, Context) method is called for each <location, sales> pair in the grouped inputs.The output of the reduce task is typically written to the FileSystem via Context.write(WritableComparable, Writable).
Applications can use the Counter to report its statistics.The output of the Reducer (<location, sales>) is not sorted.


JobTracker receives the requests (look for total sales in each location) for MapReduce execution from the client.JobTracker talks to the NameNode (location of <location, sales> been stored) to determine the location of the data.JobTracker finds the best TaskTracker nodes to execute tasks based on the data locality (proximity of the data) and the available slots to execute a task on a given node.JobTracker monitors the individual TaskTrackers and the submits back the overall status of the job back to the client.
TaskTracker runs on DataNode. Mostly on all DataNodes.TaskTracker will be in constant communication with the JobTracker signalling the progress of the task in execution.

At the end of reducer function, older key is updated in each loop, and added current sales to last value, and updated the total sales. 




### Applied (5 points total)

Submit the mapper.py and reducer.py and the output file (.csv or .txt) for the first question in lesson 6 for Udacity.  (The one labelled "Quiz: Sales per Category")  Instructions for how to get set up is inside the Udacity lectures.  
