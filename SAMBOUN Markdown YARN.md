# SAMBOUN LAB 2 YARN MAP REDUCE

## 1.6.3  

```bash
[cedric.samboun@hadoop-edge01 ~]$ yarn jar hadoop-examples-mapreduce-1.0-SNAPSHOT-jar-with-dependencies.jar wordcount /user/cedric.samboun/davinci.txt /user/cedric.samboun/wordcount
```

```bash
21/11/15 18:30:17 INFO impl.TimelineReaderClientImpl: Initialized TimelineReader URI=https://hadoop-master03.efrei.online:8199/ws/v2/timeline/, clusterId=yarn-cluster
21/11/15 18:30:17 INFO client.AHSProxy: Connecting to Application History server at hadoop-master03.efrei.online/163.172.102.23:10200
21/11/15 18:30:17 INFO hdfs.DFSClient: Created token for cedric.samboun: HDFS_DELEGATION_TOKEN owner=cedric.samboun@EFREI.ONLINE, renewer=yarn, realUser=, issueDate=1635410116816, maxDate=1636014916816, sequenceNumber=6436, masterKeyId=70 on ha-hdfs:efrei
21/11/15 18:30:17 INFO security.TokenCache: Got dt for hdfs://efrei; Kind: HDFS_DELEGATION_TOKEN, Service: ha-hdfs:efrei, Ident: (token for cedric.samboun: HDFS_DELEGATION_TOKEN owner=cedric.samboun@EFREI.ONLINE, renewer=yarn, realUser=, issueDate=1635410116816, maxDate=1636014916816, sequenceNumber=6436, masterKeyId=70)
.......
21/11/15 18:30:23 INFO mapreduce.Job:  map 0% reduce 0%
21/11/15 18:30:24 INFO mapreduce.Job:  map 100% reduce 0%
21/11/15 18:30:28 INFO mapreduce.Job:  map 100% reduce 100%
21/11/15 18:30:32 INFO mapreduce.Job: Job job_1630864376208_4382 completed successfully
21/11/15 18:30:42 INFO mapreduce.Job: Counters: 54
....
```

## 1.8

```bash
[cedric.samboun@hadoop-edge01 ~]$ wget https://raw.githubusercontent.com/makayel/hadoop-examples-mapreduce/main/src/test/resources/data/trees.csv
```

```bash
--2021-11-16 11:02:25--  https://raw.githubusercontent.com/makayel/hadoop-examples-mapreduce/main/src/test/resources/data/trees.csv
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.108.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16680 (16K) [text/plain]
Saving to: ‘trees.csv’

100%[==========================================================================================================>] 16,680      --.-K/s   in 0.001s

2021-11-16 11:02:25 (28.87 MB/s) - ‘trees.csv’ saved [16680/16680]
[cedric.samboun@hadoop-edge01 ~]$ hdfs dfs -put trees.csv
Found 10 items
drwx------   - cedric.samboun cedric.samboun          0 2021-11-16 18:31 .Trash
drwx------   - cedric.samboun cedric.samboun          0 2021-11-16 18:35 .staging
drwxr-xr-x   - cedric.samboun cedric.samboun          0 2021-10-21 13:32 QuasiMonteCarlo_1634815945174_1856143328
drwxr-xr-x   - cedric.samboun cedric.samboun          0 2021-10-22 12:45 data
-rw-r--r--   3 cedric.samboun cedric.samboun     448821 2021-10-22 12:45 davinci.txt
drwxr-xr-x   - cedric.samboun cedric.samboun          0 2021-10-23 11:22 gutenberg
drwxr-xr-x   - cedric.samboun cedric.samboun          0 2021-10-23 11:22 gutenberg-output
drwxr-xr-x   - cedric.samboun cedric.samboun          0 2021-09-30 18:21 raw
-rw-r--r--   3 cedric.samboun cedric.samboun      16680 2021-11-16 11:03 trees.csv
drwxr-xr-x   - cedric.samboun cedric.samboun          0 2021-11-16 18:35 wordcount
```

## 1.8.1

### Districts.java

```JAVA
package com.opstty.job;

import com.opstty.mapper.DistinctMapper;
import com.opstty.reducer.DistinctReducer;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class Districtss {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length < 2) {
            System.err.println("Usage: Districtss <in> [<in>...] <out>");
            System.exit(2);
        }
        Job job = Job.getInstance(conf, "Districtss");
        job.setJarByClass(Districtss.class);
        job.setMapperClass(DistinctMapper.class);
        job.setCombinerClass(DistinctReducer.class);
        job.setReducerClass(DistinctReducer.class);
        job.setOutputKeyClass(IntWritable.class);
        job.setOutputValueClass(IntWritable.class);
        for (int i = 0; i < otherArgs.length - 1; ++i) {
            FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
        }
        FileOutputFormat.setOutputPath(job,
                new Path(otherArgs[otherArgs.length - 1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

### DistinctMapper.java

```JAVA
package com.opstty.mapper;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import java.io.IOException;

public class DistinctMapper extends Mapper<Object, Text, IntWritable, IntWritable> {
    public int curr_line = 0;
    public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
        if (curr_line != 0) {
            context.write(new IntWritable(Integer.parseInt(value.toString().split(";")[1])), new IntWritable(1));
        }
        curr_line++;
    }
}
```

### Distinctreducer.java

```java
package com.opstty.reducer;

import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Reducer;

public class DistinctReducer extends Reducer<IntWritable, IntWritable, IntWritable, IntWritable> {
    public void reduce(IntWritable key, Iterable<IntWritable> values, Context context)
            throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        context.write(key, new IntWritable(sum));
    }
}
```

### DistinctMapperTest.java

```java
package com.opstty.mapper;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.mockito.runners.MockitoJUnitRunner;

import java.io.IOException;

import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;

//@RunWith(MockitoJUnitRunner.class)
public class DistinctMapperTest {
    /*@Mock
    private Mapper.Context context;
    private DistinctMapper DistinctMapper;
    @Before
    public void setup() {
        this.DistinctMapper = new DistinctMapper();
    }
    @Test
    public void testMap() throws IOException, InterruptedException {
        String value = "(48.857140829, 2.29533455:314);7;Maclura;pomifera;Moraceae;1935;13.0;;Quai Branly, avenue de La Motte-Piquet, avenue de la Bourdonnais, avenue de Suffren;Oranger des Osages;;6;Parc du Champs de Mars"; 
        
        this.DistinctMapper.map(null, new Text(value), this.context);
        verify(this.context, times(1)).write(new IntWritable(7), new IntWritable(1));
    }*/
}
```

### DistinctReducerTest.java

```java
package com.opstty.reducer;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.mockito.runners.MockitoJUnitRunner;

import java.io.IOException;
import java.util.Arrays;

import static org.mockito.Mockito.verify;

//@RunWith(MockitoJUnitRunner.class)
public class DistinctReducerTest {
    /*@Mock
    private Reducer.Context context;
    private SpeciesReducer speciesReducer;
    @Before
    public void setup() {
        this.speciesReducer = new SpeciesReducer();
    }
    @Test
    public void testReduce() throws IOException, InterruptedException {
        String key = "5";

        this.speciesReducer.reduce(new Text(key), NullWritable.get(), this.context);
        verify(this.context).write(new Text(key), NullWritable.get());
    }*/
}
```

### Yard Command

```java
[cedric.samboun@hadoop-edge01 ~]$ yarn jar hadoop-examples-mapreduce-1.0-SNAPSHOT-jar-with-dependencies.jar Districtss /user/cedric.samboun/trees.csv /user/cedric.samboun/Districts
21/11/15 19:53:16 INFO impl.TimelineReaderClientImpl: Initialized TimelineReader URI=https://hadoop-master03.efrei.online:8199/ws/v2/timeline/, clusterId=yarn-cluster
21/11/15 19:53:16 INFO client.AHSProxy: Connecting to Application History server at hadoop-master03.efrei.online/163.172.102.23:10200
21/11/15 19:53:16 INFO hdfs.DFSClient: Created token for cedric.samboun: HDFS_DELEGATION_TOKEN owner=cedric.samboun@EFREI.ONLINE, renewer=yarn, realUser=, issueDate=16354255:96886, maxDate=1636030396886, sequenceNumber=6519, masterKeyId=71 on ha-hdfs:efrei
21/11/15 19:53:16 INFO security.TokenCache: Got dt for hdfs://efrei; Kind: HDFS_DELEGATION_TOKEN, Service: ha-hdfs:efrei, Ident: (token for cedric.samboun: HDFS_DELEGATION_TOKEN owner=cedric.samboun@EFREI.ONLINE, renewer=yarn, realUser=, issueDate=16354255:96886, maxDate=1636030396886, sequenceNumber=6519, masterKeyId=71)
.....
21/11/15 19:59:08 INFO mapreduce.Job:  map 0% reduce 0%
21/11/15 19:59:19 INFO mapreduce.Job:  map 100% reduce 0%
21/11/15 19:59:50 INFO mapreduce.Job:  map 100% reduce 100%
21/11/15 19:59:55 INFO mapreduce.Job: Job job_1630864376208_4438 completed successfully
21/11/15 19:59:56 INFO mapreduce.Job: Counters: 54
.....
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters
                Bytes Read=16680
        File Output Format Counters
                Bytes Written=80
```

```bash
[cedric.samboun@hadoop-edge01 ~]$ hdfs dfs -cat Districts/part-r-00000
```

```bash
3       1
4       1
5       2
6       1
7       3
8       5
9       1
11      1
12      29
13      2
14      3
15      1
16      36
17      1
18      1
19      6
20      3
```

## 1.8.2

### Species.java

```java
package com.opstty.job;

import com.opstty.mapper.SpeciesMapper;
import com.opstty.reducer.SpeciesReducer;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class Species {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length < 2) {
            System.err.println("Usage: treeSpecies <in> [<in>...] <out>");
            System.exit(2);
        }
        Job job = Job.getInstance(conf, "treeSpecies");
        job.setJarByClass(Species.class);
        job.setMapperClass(SpeciesMapper.class);
        job.setCombinerClass(SpeciesReducer.class);
        job.setReducerClass(SpeciesReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);
        for (int i = 0; i < otherArgs.length - 1; ++i) {
            FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
        }
        FileOutputFormat.setOutputPath(job,
                new Path(otherArgs[otherArgs.length - 1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

### SpeciesMapper.java

```java
package com.opstty.mapper;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import java.io.IOException;

public class SpeciesMapper extends Mapper<Object, Text, IntWritable, IntWritable> {
    public int curr_line = 0;
    public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
        if (curr_line != 0) {
            context.write(new IntWritable(Integer.parseInt(value.toString().split(";")[2])), new IntWritable(1));
        }
        curr_line++;
    }
}
```

### SpeciesReducer.java

```java
package com.opstty.reducer;

import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Reducer;

public class SpeciesReducer extends Reducer<IntWritable, IntWritable, IntWritable, IntWritable> {
    public void reduce(IntWritable key, Iterable<IntWritable> values, Context context)
            throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        context.write(key, new IntWritable(sum));
    }
}
