---
title: "Max Global Temperature — Another Hadoop Demo Program"
date: 2018-03-19T00:00:00-04:00
lastmod: 2018-04-15T06:57:05-04:00
draft: false
categories: ["Technique"]
tags: ["Hadoop", "MapReduce", "Big Data"]
---

In this post, we are going to go through another demo program on Hadoop - Max Global Temperature.
# 1. Source Code
<!--more-->
```java
import java.io.IOException;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

// The implementation of map funciton by inheriting Mapper class and overriding its map() method.
// The Mapper class is a generic type, with four formal type parameters that specify the: input key, input value, output key, and output value types of the map function.

// In the following example:
// input key    : long integer offset (LongWritable)[like JAVA Long]
// input value  : a line of text      (Text)        [like JAVA String]
// output key   : year                (Text)        [like JAVA String]    
// output value : an air temperature  (IntWritable) [like JAVA Integer]

// Rather than using buit-in Java types, Hadoop provides its own set of basic types that are optimized for network serialization.
// These are found in the org.apache.hadoop.io package.

public class MaxTemperature {
    public static class MaxTemperatureMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
        private static final int MISSING = 9999;
        // Override the map method in Mapper class.
        @Override
        public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            // The map() method is passed a key and a value. We convert the Text value containing the line of input into a JAVA String, then use its substring() method to extract the columns we are interested in.
            String line = value.toString();
            String year = line.substring(15, 19);
            int airTemperature;
            if (line.charAt(87) == '+') {
                // parseInt doesn't like leading plus signs
                airTemperature = Integer.parseInt(line.substring(88, 92));
            } else {
                airTemperature = Integer.parseInt(line.substring(87, 92));
            }
            String quality = line.substring(92, 93);
            // The map() method also provides an instance of Context to write the output to. In this case, we write the year as a Text object (since we are just using it as a key), and the temperature is wrapped in a IntWritable. We write and output recoed only if the temperature is present and the quality code indicates the temperature reading is OK.
            if (airTemperature != MISSING && quality.matches("[01459]")) {
                context.write(new Text(year), new IntWritable(airTemperature));
            }
        }
    }
    
    // The implementation of reduce function by inheriting Reducer generic class and overriding its reduce() method.
    // Four formal type parameters are used to specify the input and output types. The input types of the reduce function must match the output types of the map function: Text and IntWritable. And in this case, the output types of the reduce function are Text and IntWritable, for a year and its maximum temperature.
    // 
    public static class MaxTemperatureReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
        @Override
        public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
            int maxValue = Integer.MIN_VALUE;
            for (IntWritable value : values) {
                maxValue = Math.max(maxValue, value.get());
            }
            context.write(key, new IntWritable(maxValue));
        }
    }
    
    // Run MapREduce job.
    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.err.println("Usage: MaxTemperature <input path> <output path>");
            System.exit(-1);
        }
        
        // A Job object forms the specification of the job and gives you control over how the job is run. When we run this job on a Hadoop cluster, we will package the code into a JAR file (which Hadoop will distribute around the cluster). Rather than explicitly specifying the name of the JAR file, we can pass a class in the Job's setJarByClass() method, which Hadoop will use to locate the relevant JAR file by looking for the JAR file containing this class.
        Job job = new Job();
        job.setJarByClass(MaxTemperature.class);
        job.setJobName("Max temperature");
        
        // addInputPath() can be called more than once to use input from multiple paths.
        FileInputFormat.addInputPath(job, new Path(args[0]));
        // The output directory shouldn't exist before running the job.
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        
        job.setMapperClass(MaxTemperatureMapper.class);
        job.setReducerClass(MaxTemperatureReducer.class);
        
        // setOutputKeyClass() and setOutputValueClass() control the output types for the reduce function, and must match what the Reduce class produces.
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        
        // The return value of the waitForCompletion() method is a Boolean indicating success (true) or failure (false), which we translate into the programs's exit code of 0 or 1.
        System.exit(job.waitForCompletion(true)? 0 : 1);
    }
}
```

# 2. Compile Source Code

Compile using a makefile.
```makefile
# Define compiler.
JC = javac

# Set compiler flag variables.
# -g will print all error message.
# -classpath contains the classes used for compiling the project.
# -d sets the destination path.
JFLAGS = -g -classpath $(HADOOP_HOME)/share/hadoop/common/hadoop-common-3.0.0.jar:$(HADOOP_HOME)/share/hadoop/mapreduce/hadoop-mapreduce-client-core-3.0.0.jar:$(HADOOP_HOME)/share/hadoop/common/commons-cli-1.2.jar -d $(JCLASSDIR)

# Clear any default targets for building .class files from .java files; we 
# will provide our own target entry to do this in this makefile.
# make has a set of default targets for different suffixes (like .c.o) 
# Currently, clearing the default for .java.class is not necessary since 
# make does not have a definition for this target, but later versions of 
# make may, so it doesn't hurt to make sure that we clear any default 
# definitions for these
.SUFFIXES: .java .class

# Set the paths of the source files and class files.
SOURCEDIR = src/
JCLASSDIR = classes/

# Here is our target entry for creating .class files from .java files 
# This is a target entry that uses the suffix rule syntax:
# DSTS:
#     rule
# 'TS' is the suffix of the target file, 'DS' is the suffix of the dependency 
# file, and 'rule' is the rule for building a target.
# '$*' is a built-in macro that gets the basename of the current target 
# Remember that there must be a < tab > before the command line ('rule') 
.java.class:
    $(JC) $(JFLAGS) $*.java


# CLASSES is a macro consisting all java source files.
# If makefile and source file are not in same folder, you have to specify the path.
CLASSES = $(SOURCEDIR)MaxTemperature.java

# The default make target entry
default: classes

# This target entry uses Suffix Replacement within a macro: 
# $(name:string1=string2)
# In the words in the macro named 'name' replace 'string1' with 'string2'
# Below we are replacing the suffix .java of all words in the macro CLASSES 
# with the .class suffix
classes: $(CLASSES:.java=.class)

# RM is a predefined macro in make (RM = rm -f)
clean:
    $(RM) $(JCLASSDIR)*.class
```

Note that, in makefile, the tab cannot be replaced by 4 or 8 spaces. If your text editor will convert tab to spaces, disable this function.

Save the code above and name it as makefile. Put this makefile to the folder `/Users/jz0006/GoogleDrive/Hadoop/my_projects/02_max_temperature`. Then cd to this folder, run make command.
```bash
$ cd /Users/jz0006/GoogleDrive/Hadoop/my_projects/02_max_temperature
$ make
```
In terminal, you will see the output:
<pre>
javac -g -classpath /Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/share/hadoop/common/hadoop-common-3.0.0.jar:/Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/share/hadoop/mapreduce/hadoop-mapreduce-client-core-3.0.0.jar:/Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/share/hadoop/common/commons-cli-1.2.jar -d classes/ src/MaxTemperature.java
Note: src/MaxTemperature.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
</pre>

# 3. Create .jar File

Use jar command to put all classes into a .jar file:
```bash
$ cd ./classes
$ jar cf ../mt.jar MaxTemperature*.class
```
Now the `mt.jar` file is in the folder `/Users/jz0006/GoogleDrive/Hadoop/my_projects/02_max_temperature`. Go back to the parent folder, you should see this `mt.jar` file.

# 4. Put Program and Input Files into Hadoop HDFS File System

For our convenient, we create a local folder to put input files.
```bash
$ mkdir /Users/jz0006/GoogleDrive/Hadoop/my_projects/02_max_temperature/input
```
Copy two year’s data to the path above, 1901 and 1902.

Now we are going to create folders in Hadoop file system (HDFS). Note that, the `cd` command in Linux/Unix system won't make any sense in Hadoop file system, since Hadoop file system is a virtual file system. You should always provide the full path to access a folder. 

Lets check the root folder of your HDFS. In terminal, run:
```bash
$ hadoop fs -ls /
```
The output in my system is (Please ignore the warning message):
<pre>
2018-03-19 02:21:27,523 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 2 items
drwx------   - jz0006 supergroup          0 2018-03-15 05:56 /tmp
drwxr-xr-x   - jz0006 supergroup          0 2018-03-15 05:56 /user
</pre>
It shows that there are two folders under the root path. I’ve already created `/user` folder. If you haven’t, create it: 
```bash
$ hadoop fs -mkdir /user
```
Then, I create a user named by `jz0006`:
```bash
$ hadoop fs -mkdir /user/jz0006
```
Next, we create a folder for our word count program. 
```bash
$ hadoop fs -mkdir /user/jz0006/02_max_temperature
```
Under wordcount folder, we create an input folder to put our input files.
```bash
$ hadoop fs -mkdir /user/jz0006/02_max_temperature/input
```
In this example, please don’t create output folder. Because the word count program will create output folder by itself. If this folder already exists, an exception will occur. If you run the program for the second time, the output folder will be there already, delete it by typing:
```bash
$ hadoop fs -rm -r /user/jz0006/02_max_temperature/output/
```
Now it’s the time to put our input files into HDFS. Firstly, go to the path where the input files are located. 
```bash
$ cd /Users/jz0006/GoogleDrive/Hadoop/my_projects/02_max_temperature/input
```
Then use hadoop command to put the input files into HDFS.
```bash
$ hadoop fs -put * /user/jz0006/02_max_temperature/input
```
Check it: 
```bash
$ hadoop fs -ls /user/jz0006/02_max_temperature/input
```
Output:
<pre>
2018-03-19 02:07:49,097 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 2 items
-rw-r--r--   1 jz0006 supergroup   888190 2018-03-19 00:36 /user/jz0006/02_max_temperature/input/1901
-rw-r--r--   1 jz0006 supergroup   888978 2018-03-19 00:36 /user/jz0006/02_max_temperature/input/1902
</pre>
We can see that the input files are put into HDFS successfully. 

# 5. Run Hadoop Program

Firstly, `cd` to the folder that contains the `.jar` file.
```bash
$ cd /Users/jz0006/GoogleDrive/Hadoop/my_projects/02_max_temperature
```
Then run the command: (command format is: `hadoop jar <filename.jar> MainClass <input path on hdfs> <output path on hdfs>`)
```bash
$ hadoop jar mt.jar MaxTemperature /user/jz0006/02_max_temperature/input /user/jz0006/02_max_temperature/output
```
The terminal output:
<pre>
2018-03-19 02:12:08,616 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
2018-03-19 02:12:09,400 INFO client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
2018-03-19 02:12:09,991 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
2018-03-19 02:12:10,007 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/jz0006/.staging/job_1521107676874_0018
2018-03-19 02:12:10,270 INFO input.FileInputFormat: Total input files to process : 2
2018-03-19 02:12:10,328 INFO mapreduce.JobSubmitter: number of splits:2
2018-03-19 02:12:10,405 INFO Configuration.deprecation: yarn.resourcemanager.system-metrics-publisher.enabled is deprecated. Instead, use yarn.system-metrics-publisher.enabled
2018-03-19 02:12:10,544 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1521107676874_0018
2018-03-19 02:12:10,545 INFO mapreduce.JobSubmitter: Executing with tokens: []
2018-03-19 02:12:10,788 INFO conf.Configuration: resource-types.xml not found
2018-03-19 02:12:10,789 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2018-03-19 02:12:10,895 INFO impl.YarnClientImpl: Submitted application application_1521107676874_0018
2018-03-19 02:12:10,940 INFO mapreduce.Job: The url to track the job: http://localhost:8088/proxy/application_1521107676874_0018/
2018-03-19 02:12:10,941 INFO mapreduce.Job: Running job: job_1521107676874_0018
2018-03-19 02:12:19,105 INFO mapreduce.Job: Job job_1521107676874_0018 running in uber mode : false
2018-03-19 02:12:19,106 INFO mapreduce.Job:  map 0% reduce 0%
2018-03-19 02:12:26,194 INFO mapreduce.Job:  map 100% reduce 0%
2018-03-19 02:12:32,243 INFO mapreduce.Job:  map 100% reduce 100%
2018-03-19 02:12:32,251 INFO mapreduce.Job: Job job_1521107676874_0018 completed successfully
2018-03-19 02:12:32,366 INFO mapreduce.Job: Counters: 49
    File System Counters
        FILE: Number of bytes read=144425
        FILE: Number of bytes written=903669
        FILE: Number of read operations=0
        FILE: Number of large read operations=0
        FILE: Number of write operations=0
        HDFS: Number of bytes read=1777424
        HDFS: Number of bytes written=18
        HDFS: Number of read operations=11
        HDFS: Number of large read operations=0
        HDFS: Number of write operations=2
    Job Counters 
        Launched map tasks=2
        Launched reduce tasks=1
        Data-local map tasks=2
        Total time spent by all maps in occupied slots (ms)=9085
        Total time spent by all reduces in occupied slots (ms)=3375
        Total time spent by all map tasks (ms)=9085
        Total time spent by all reduce tasks (ms)=3375
        Total vcore-milliseconds taken by all map tasks=9085
        Total vcore-milliseconds taken by all reduce tasks=3375
        Total megabyte-milliseconds taken by all map tasks=9303040
        Total megabyte-milliseconds taken by all reduce tasks=3456000
    Map-Reduce Framework
        Map input records=13130
        Map output records=13129
        Map output bytes=118161
        Map output materialized bytes=144431
        Input split bytes=256
        Combine input records=0
        Combine output records=0
        Reduce input groups=2
        Reduce shuffle bytes=144431
        Reduce input records=13129
        Reduce output records=2
        Spilled Records=26258
        Shuffled Maps =2
        Failed Shuffles=0
        Merged Map outputs=2
        GC time elapsed (ms)=197
        CPU time spent (ms)=0
        Physical memory (bytes) snapshot=0
        Virtual memory (bytes) snapshot=0
        Total committed heap usage (bytes)=611844096
    Shuffle Errors
        BAD_ID=0
        CONNECTION=0
        IO_ERROR=0
        WRONG_LENGTH=0
        WRONG_MAP=0
        WRONG_REDUCE=0
    File Input Format Counters 
        Bytes Read=1777168
    File Output Format Counters 
        Bytes Written=18
</pre>

# 6. Check the Result

Firstly, let’s check what’s in the output folder:
```bash
$ hadoop fs -ls /user/jz0006/02_max_temperature/output
```
Output is:
<pre>
2018-03-19 02:17:05,283 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 2 items
-rw-r--r--   1 jz0006 supergroup          0 2018-03-19 02:12 /user/jz0006/02_max_temperature/output/_SUCCESS
-rw-r--r--   1 jz0006 supergroup         18 2018-03-19 02:12 /user/jz0006/02_max_temperature/output/part-r-00000
</pre>
The result is in the file `part-r-00000`. Let look into this file using cat command:
```bash
$ hadoop fs -cat /user/jz0006/02_max_temperature/output/part-r-00000
```
The result is:
<pre>
2018-03-19 02:18:39,260 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
1901    317
1902    244
</pre>

Finally, we successfully get the maximum temperature for 1901 and 1902. Now, we are ready to apply our program to large dataset.

