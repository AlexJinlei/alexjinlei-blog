---
title: "Word Count — the First Program on Hadoop"
date: 2018-03-16T00:00:00-04:00
lastmod: 2018-04-15T03:19:19-04:00
draft: false
categories: ["Technique"]
tags: ["Hadoop", "MapReduce", "Big Data"]
---

In this post, I will talk about how to run the demo program "word count" in Hadoop environment.

# 1. Source Code
<!--more-->
```java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```

# 2. Compile Source Code

## Create program folder:
```bash
$ mkdir /Users/jz0006/GoogleDrive/Hadoop/my_app/01_word_count
$ mkdir /Users/jz0006/GoogleDrive/Hadoop/my_app/01_word_count/classes
$ mkdir /Users/jz0006/GoogleDrive/Hadoop/my_app/01_word_count/src
```
Name source code as `WordCount.java`, save it in `/src` folder that we created in last step.

To compile `WordCount.java` source code, we need three classes in Hadoop folder, they are:
<pre>
\<your-hadoop-home-folder\>/share/hadoop/common/hadoop-common-3.0.0.jar
\<your-hadoop-home-folder\>/share/hadoop/mapreduce/hadoop-mapreduce-client-core-3.0.0.jar
\<your-hadoop-home-folder\>/share/hadoop/common/commons-cli-1.2.jar
</pre>
In this example, my hadoop home folder is `/Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0`
These three classes should be pass to compiler by `-classpath` flag.

You can either use command line or makefile to compile source code.

## Method 1: Compile via Command Line:

In terminal, navigate to the program root folder,
```bash
$ cd /Users/jz0006/GoogleDrive/Hadoop/my_app/01_word_count
```
Then run the following command,
```bash
$ javac -g -classpath /Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/share/hadoop/common/hadoop-common-3.0.0.jar:/Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/share/hadoop/mapreduce/hadoop-mapreduce-client-core-3.0.0.jar:/Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/share/hadoop/common/commons-cli-1.2.jar -d classes/ src/WordCount.java
```
The `-g` flag is to show all debug message, the `-classpath` flag pass the path of needed classes, the `-d` flag tell the compiler where to put the compiled file, and the last parameter is the path of source file which is to be compiles. 

Then, go to the `/classes` folder to check the compiling result. 
```bash
$ cd /Users/jz0006/GoogleDrive/Hadoop/my_app/01_word_count/classes
```
Your should see three files, `WordCount$IntSumReducer.class`, `WordCount.class`, and `WordCount$TokenizerMapper.class`.

## Method 2: Compile using makefile:
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

# Set the pathes of the source files and class files.
SOURCEDIR = src/
JCLASSDIR = classes/

# Here is our target entry for creating .class files from .java files 
# This is a target entry that uses the suffix rule syntax:
# DSTS:
#     rule
# 'TS' is the suffix of the target file, 'DS' is the suffix of the dependency 
# file, and 'rule'  is the rule for building a target.
# '$*' is a built-in macro that gets the basename of the current target 
# Remember that there must be a < tab > before the command line ('rule') 
.java.class:
    $(JC) $(JFLAGS) $*.java

# CLASSES is a macro consisting all java source files.
# If makefile and source file are not in same folder, you have to specify the path.
CLASSES = $(SOURCEDIR)WordCount.java

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
Note that, in `makefile`, the tab cannot be replaced by 4 or 8 spaces. If your text editor will convert tab to spaces, disable this function.

Save the code above and name it as makefile. Put this makefile to the folder `/Users/jz0006/GoogleDrive/Hadoop/my_app/01_word_count`. Then cd to this folder, run make command.
```bash
$ cd /Users/jz0006/GoogleDrive/Hadoop/my_app/01_word_count
$ make
```

In terminal, you will see the output:
<pre>
javac -g -classpath /Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/share/hadoop/common/hadoop-common-3.0.0.jar:/Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/share/hadoop/mapreduce/hadoop-mapreduce-client-core-3.0.0.jar:/Users/jz0006/GoogleDrive/Hadoop/hadoop-3.0.0/share/hadoop/common/commons-cli-1.2.jar -d classes/ src/WordCount.java
</pre>
# 3. Create .jar File

Use jar command to put all classes into a .jar file:
```bash
$ jar cf ../wc.jar WordCount*.class
```
Now the `wc.jar` file is in the folder `/Users/jz0006/GoogleDrive/Hadoop/my_app/01_word_count`. Go back to the parent folder, you should see this `wc.jar` file.

# 4. Put Program and Input Files into Hadoop HDFS File System

For our convenience, we are going to create a local folder to put input files.
```bash
$ mkdir /Users/jz0006/GoogleDrive/Hadoop/my_app/01_word_count/input
```
Create to text files under the input folder.
```bash
$ >file01.txt
$ >file02.txt
```
Open `file01.txt` use text editor, input: `Hello World Bye World`.
Open `file02.txt` use text editor, input: `Hello Hadoop Goodbye Hadoop`.

Save and close the input files. 

Now we are going to create folders in Hadoop file system (HDFS). Note that, the cd command in Linux/Unix system won’t make any sense in Hadoop file system, since Hadoop file system is a virtual file system. You should always provide the full path to access a folder. 

Lets check the root folder of your HDFS. In terminal, run:
```bash
$ hadoop fs -ls /
```
The output in my system is (Please ignore the warning message):

<pre>2018-03-16 15:21:26,693 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 2 items
drwx------   - jz0006 supergroup          0 2018-03-15 05:56 /tmp
drwxr-xr-x   - jz0006 supergroup          0 2018-03-15 05:56 /user
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
$ hadoop fs -mkdir /user/jz0006/wordcount
```
Under wordcount folder, we create an input folder to put our input files.
```bash
$ hadoop fs -mkdir /user/jz0006/wordcount/input
```
In this example, please don’t create output folder. Because the word count program will create output folder by itself. If this folder already exists, an exception will occur. If you run the program for the second time, the output folder will be there already, delete it by typing:
```bash
$ hadoop fs -rm -r /user/jz0006/wordcount/output/
```
Now it’s the time to put our input files into HDFS. Firstly go to the path where the input files are located. 
```bash
$ cd /Users/jz0006/GoogleDrive/Hadoop/my_app/01_word_count/input
```
Then use hadoop command to put the input files into HDFS.
```bash
$ hadoop fs -put *.txt /user/jz0006/wordcount/input
```
Check it: 
```bash
$ hadoop fs -ls /user/jz0006/wordcount/input
```
Output:
<pre>
2018-03-16 15:40:37,437 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 2 items
-rw-r--r--   1 jz0006 supergroup         21 2018-03-16 03:58 /user/jz0006/wordcount/input/file01.txt
-rw-r--r--   1 jz0006 supergroup         27 2018-03-16 03:58 /user/jz0006/wordcount/input/file02.txt
</pre>
We can see that the input files are put into HDFS successfully. 

# 5. Run Hadoop Program

Firstly, cd to the folder that contains the `.jar` file.
```bash
$ cd /Users/jz0006/GoogleDrive/Hadoop/my_app/01_word_count
```
Then run the command:
```bash
$ hadoop jar wc.jar WordCount /user/jz0006/wordcount/input /user/jz0006/wordcount/output
```
The terminal output:
<pre>
2018-03-16 15:47:28,758 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
2018-03-16 15:47:29,506 INFO client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
2018-03-16 15:47:30,061 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
2018-03-16 15:47:30,075 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/jz0006/.staging/job_1521107676874_0016
2018-03-16 15:47:30,283 INFO input.FileInputFormat: Total input files to process : 2
2018-03-16 15:47:30,342 INFO mapreduce.JobSubmitter: number of splits:2
2018-03-16 15:47:30,420 INFO Configuration.deprecation: yarn.resourcemanager.system-metrics-publisher.enabled is deprecated. Instead, use yarn.system-metrics-publisher.enabled
2018-03-16 15:47:30,563 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1521107676874_0016
2018-03-16 15:47:30,565 INFO mapreduce.JobSubmitter: Executing with tokens: []
2018-03-16 15:47:30,813 INFO conf.Configuration: resource-types.xml not found
2018-03-16 15:47:30,814 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2018-03-16 15:47:30,924 INFO impl.YarnClientImpl: Submitted application application_1521107676874_0016
2018-03-16 15:47:30,980 INFO mapreduce.Job: The url to track the job: http://localhost:8088/proxy/application_1521107676874_0016/
2018-03-16 15:47:30,981 INFO mapreduce.Job: Running job: job_1521107676874_0016
2018-03-16 15:47:38,093 INFO mapreduce.Job: Job job_1521107676874_0016 running in uber mode : false
2018-03-16 15:47:38,095 INFO mapreduce.Job:  map 0% reduce 0%
2018-03-16 15:47:44,190 INFO mapreduce.Job:  map 100% reduce 0%
2018-03-16 15:47:49,226 INFO mapreduce.Job:  map 100% reduce 100%
2018-03-16 15:47:49,232 INFO mapreduce.Job: Job job_1521107676874_0016 completed successfully
2018-03-16 15:47:49,340 INFO mapreduce.Job: Counters: 49
    File System Counters
        FILE: Number of bytes read=79
        FILE: Number of bytes written=615370
        FILE: Number of read operations=0
        FILE: Number of large read operations=0
        FILE: Number of write operations=0
        HDFS: Number of bytes read=298
        HDFS: Number of bytes written=41
        HDFS: Number of read operations=11
        HDFS: Number of large read operations=0
        HDFS: Number of write operations=2
    Job Counters 
        Launched map tasks=2
        Launched reduce tasks=1
        Data-local map tasks=2
        Total time spent by all maps in occupied slots (ms)=7193
        Total time spent by all reduces in occupied slots (ms)=2860
        Total time spent by all map tasks (ms)=7193
        Total time spent by all reduce tasks (ms)=2860
        Total vcore-milliseconds taken by all map tasks=7193
        Total vcore-milliseconds taken by all reduce tasks=2860
        Total megabyte-milliseconds taken by all map tasks=7365632
        Total megabyte-milliseconds taken by all reduce tasks=2928640
    Map-Reduce Framework
        Map input records=2
        Map output records=8
        Map output bytes=82
        Map output materialized bytes=85
        Input split bytes=250
        Combine input records=8
        Combine output records=6
        Reduce input groups=5
        Reduce shuffle bytes=85
        Reduce input records=6
        Reduce output records=5
        Spilled Records=12
        Shuffled Maps =2
        Failed Shuffles=0
        Merged Map outputs=2
        GC time elapsed (ms)=158
        CPU time spent (ms)=0
        Physical memory (bytes) snapshot=0
        Virtual memory (bytes) snapshot=0
        Total committed heap usage (bytes)=647495680
    Shuffle Errors
        BAD_ID=0
        CONNECTION=0
        IO_ERROR=0
        WRONG_LENGTH=0
        WRONG_MAP=0
        WRONG_REDUCE=0
    File Input Format Counters 
        Bytes Read=48
    File Output Format Counters 
        Bytes Written=41
</pre>
# 6. Check the Result

Firstly, let’s check what’s in the output folder:
```bash
$ hadoop fs -ls /user/jz0006/wordcount/output
```
Output is:
<pre>
2018-03-16 15:50:43,420 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 2 items
-rw-r--r--   1 jz0006 supergroup          0 2018-03-16 15:47 /user/jz0006/wordcount/output/_SUCCESS
-rw-r--r--   1 jz0006 supergroup         41 2018-03-16 15:47 /user/jz0006/wordcount/output/part-r-00000
</pre>
The result is in the file `part-r-00000`. Let's look into this file using cat command:
```bash
$ hadoop fs -cat /user/jz0006/wordcount/output/part-r-00000
```
You will see:

<pre>
2018-03-16 15:52:48,570 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Bye     1
Goodbye 1
Hadoop  2
Hello   2
World   2
</pre> 

We can see that each word in the two input files are counted correctly. 

