##Programming Assignment 6 Technical Notes: MapReduce

###Introduction

MapReduce is, as covered in the class, a fault-tolerant distributed system for large-scale computation. 
MapReduce programming is one of major parts in our programming assignment 6. 
We use MapReduce to compute the inverted index with TF-IDF scores. 

Since MapReduce automatically coordinates the map and reduce phases, all we have to do is to implement 
the two functions and indicate the correct input and output files. Although MapReduce is 
designed to run on thousands of machines, we run the MapReduce framework in a single-node mode for easy debugging.

This note is written to give a more detailed explanation about MapReduce programming. 
It supplements the main programming assignment manual. We will cover download and installation of Apache Hadoop, 
and cover a couple of examples including the word count covered in the class. 
Apache Hadoop is a publicly available software providing distributed file system (HDFS) and MapReduce framework.

### Download and Installation

In this section, we will download and install Apache Hadoop. We finish this section by running an 
example program (word count) included in the downloaded code. Although we can download 
and install Hadoop from its official repository, we will use our own version of the code along with dataset and a couple of more examples.


To download it, you can use wget on your servers. After ssh’ing to your machine, go to your working directory, and type the following command:

  `wget http://www-personal.umich.edu/~zhezhao/projects/hadoop-example.tar.gz`
  
Download will take a few minutes. Once download has finished, you need to uncompress it:

  `tar -xvzf hadoop-example.tar.gz`

Now youc an find the downloaded code in the directory hadoop-example. Let’s call this directory `{HOME}` from now on. The dataset we will use is stored in `{HOME}/dataset`, 
and example code provided is in `{HOME}/mysrc`. Let’s move to `{HOME}/mysrc` to run our first MapReduce program. 
In that directory, type the following commands:

    make 
    ./wordCountTest.sh
  
The make command will build our example code, and the script will run a MapReduce program which reads data from `{HOME}/dataset/test` 
and output the result into `{HOME}/mysrc/output`. To see the result of the execution, type:
  
    cat output/*
  
It will print out pairs of (word, count) from our test data file 
(this file includes two Wikipedia articles which is extracted from our another big data file, mining.articles.xml). 
If you have gotten to this point without any problems, we are ready to write our own MapReduce program.

### WordCount

Word Count is a very simple example. However, by taking a look into the code, we can understand most of the MapReduce programming model. Other more complicated applications are not so different from this simple example.
The code is copied from [Here](http://wiki.apache.org/hadoop/WordCount), and is also stored in `{HOME}/mysrc`.

    package edu.umich.cse.eecs485;
  
    import java.io.IOException;
    import java.util.*;
          
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.conf.*;
    import org.apache.hadoop.io.*;
    import org.apache.hadoop.mapreduce.*;
    import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
    import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
    import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
    import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
            
    public class WordCount {
            
     public static class Map extends Mapper<LongWritable, Text, Text, IntWritable> {
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();
            
        public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            String line = value.toString();
            StringTokenizer tokenizer = new StringTokenizer(line);
            while (tokenizer.hasMoreTokens()) {
                word.set(tokenizer.nextToken());
                context.write(word, one);
            }
        }
     } 
            
     public static class Reduce extends Reducer<Text, IntWritable, Text, IntWritable> {
        public void reduce(Text key, Iterable<IntWritable> values, Context context) 
          throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            context.write(key, new IntWritable(sum));
        }
     }
            
     public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
            
            Job job = new Job(conf, "wordcount");
        
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
            
        job.setMapperClass(Map.class);
        job.setReducerClass(Reduce.class);
            
        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);
            
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
            
        job.waitForCompletion(true);
     }
    }


MapReduce programs divide into three parts: map function in Mapper class, reduce function in Reducer class, 
and several configuration options in the main function. The map 
function is defined in Map class that extends a generic base class `Mapper<K, V>`. This map class 
is a custom class which is not provided by the library, and we are stating that we will use 
this class as a mapper by calling the function `job.setMapperClass()` in the configuration part. 
The map function simply tokenizes provided input and emits pairs using context. 
We will discuss how to parse the input file and pass the key-value pairs to the map function shortly.
Once the results from map functions are collected (or a part of them is available), 
the MapReduce system automatically invokes the reduce function defined in Reduce class. 
As in Map class, we also mention in the main function that we will use this class for our reduce job. 
The pairs emitted from the reduce function are written to the destination folder we designated in 
one of program arguments (you can find the arguments in the bash script you ran).
We also need to address the following lines of code in main():

    job.setInputFormatClass(TextInputFormat.class);
    job.setOutputFormatClass(TextOutputFormat.class);

These two lines set how to parse the input file and how to output into a result file. 
The class TextInputFormat is the one provided by Hadoop library, and it splits all the files in the 
input directory by their physical block size. 
Such a file split policy may be adequate for an application like word count since it does not depend on 
logical boundaries within files. However, using this class would not be proper 
for our dataset in which data is stored in XML format and logical boundaries should be maintained. 
To resolve the issue, we will use another file-split library in the next section that is developed to 
parse files in XML format. 

**Note:** some unfamiliar types used in the above code such as LongWritable, Text, 
etc are types provided by the Hadoop library, and used to pass keys or values between map 
and reduce functions. They are simply a wrapper of basic 
types like long and String, and we can get the values by calling, for example, `variable.get()`, and can 
set the value by `variable.set(myvalue)`.

### Simple Inverted Index

Before we get down to the inverted index example, let’s first take a look at the input file in XML format, 
to set up a strategy about how to split the large file into smaller chunks.

    <eecs485_articles>
    <eecs485_article>
    <eecs485_article_id>303</eecs485_article_id>
    <eecs485_article_title>Alabama</eecs485_article_title>
    <eecs485_article_body>many paragraphs here
    </eecs485_article_body>
    </eecs485_article>
    # <eecs485_article> pairs repeated
    </eecs485_articles>

To build the inverted index from the file in the above format, we will extract every article 
from the file and pass it to the map function. The map function, in turn, 
can read the article id and article body, and emit key-value pairs. To extract a document 
from the file, we should get every part that starts with `<eecs485_article>` and ends with `</eecs485_article>`. 
We can perform this task exactly as we want by using `XmlInputFormat` class 
(which comes from the Mahout machine learning project). The file is already included in our `{HOME}/mysrc` directory 
and is used together with our inverted index example. Let’s take a look at the inverted index example, 
to see how to configure to use `XmlInputFormat` class. The main function of `InvertedIndex.java` is as follows:

    public static void main(String[] args) throws Exception
    {
        Configuration conf = new Configuration();
    
        conf.set("xmlinput.start", "<eecs485_article>");
        conf.set("xmlinput.end", "</eecs485_article>");
    
        Job job = new Job(conf, "XmlParser");
    
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);
    
        job.setMapperClass(Map.class);
        job.setReducerClass(Reduce.class);
    
        job.setInputFormatClass(XmlInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);
    
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
    
        job.waitForCompletion(true);
    }

In the above example, we can see the line `job.setInputFormatClass()`. This part indicates 
the class we are going to use to parse the input file in XML format. 
Also we are telling the XmlInputFormat class the start and end of the 
blocks we want to pass to map function by setting two values with the following lines.

    conf.set("xmlinput.start", "<eecs485_article>");
    conf.set("xmlinput.end", "</eecs485_article>");

The framework automatically passes this information to XmlInputFormat during initialization, 
and the class reads those values when splitting our input files.

Other parts of the inverted index example are quite straightforward. Mapper parses XML input to read 
article id and article body. After that, it tokenizes all the words in the body and emit the pairs (word, id).  
Reducer simply combine a list of id’s it gets into text to write into output files. (We do not attach other parts of the code here. Please take a look at the file.)
Finally we can run the inverted index example by typing

    ./invIndexTest.sh
    
in `{HOME}/mysrc` directory (First clean the output directory and build again by typing make clean; make; 
if you ran the word count example. Or you can simply remove output directory by typing rm –rf output). 
Now what’s left is to write your own program: inverted index with TF-IDF values.

### Dataset

Building the inverted index with MapReduce requires just one input file: the article file called dataset/mining.articles.xml. 
The article data was generated by selecting all Wikipedia pages that 
include the word 'mining' at least once. It contains roughly 40,000 articles.
You should first try to run everything on dataset/test/small.articles.xml and then run it on th larger data in mining.articles.xml.

