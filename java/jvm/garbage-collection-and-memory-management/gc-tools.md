
# 常见命令
java -XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCTimestamps -XX:+PrintGCApplicationStopedTime -Xloggc:[文件路径]

输出GC概要信息，详细信息，时间信息，应用暂停时间信息等等到指定文件路径

## GC log的相关命令

-verbose:gc
* Displays information about each garbage collection (GC) event.

-XX:+UsePerfData
* Enables the perfdata feature. This option is enabled by default to allow JVM monitoring and performance testing. Disabling it suppresses the creation of the hsperfdata_userid directories. To disable the perfdata feature, specify -XX:-UsePerfData.

-XX:+PrintGC
* Enables printing of messages at every GC. By default, this option is disabled.

-XX:+PrintGCApplicationConcurrentTime
* Enables printing of how much time elapsed since the last pause (for example, a GC pause). By default, this option is disabled.

-XX:+PrintGCApplicationStoppedTime
* Enables printing of how much time the pause (for example, a GC pause) lasted. By default, this option is disabled.

-XX:+PrintGCDateStamps
* Enables printing of a date stamp at every GC. By default, this option is disabled.

-XX:+PrintGCDetails
* Enables printing of detailed messages at every GC. By default, this option is disabled.

-XX:+PrintGCTaskTimeStamps
* Enables printing of time stamps for every individual GC worker thread task. By default, this option is disabled.

-XX:+PrintGCTimeStamps
* Enables printing of time stamps at every GC. By default, this option is disabled.

-XX:+PrintStringDeduplicationStatistics
* Prints detailed deduplication statistics. By default, this option is disabled. See the -XX:+UseStringDeduplication option.

-XX:+PrintTenuringDistribution
* Enables printing of tenuring age information. The following is an example of the output:
* Desired survivor size 48286924 bytes, new threshold 10 (max 10)
- age 1: 28992024 bytes, 28992024 total
- age 2: 1366864 bytes, 30358888 total
- age 3: 1425912 bytes, 31784800 total
...
* Age 1 objects are the youngest survivors (they were created after the previous scavenge, survived the latest scavenge, and moved from eden to survivor space). Age 2 objects have survived two scavenges (during the second scavenge they were copied from one survivor space to the next). And so on.
* In the preceding example, 28 992 024 bytes survived one scavenge and were copied from eden to survivor space, 1 366 864 bytes are occupied by age 2 objects, etc. The third value in each row is the cumulative size of objects of age n or less.
* By default, this option is disabled.