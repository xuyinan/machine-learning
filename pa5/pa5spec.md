# Assignment 5: PageRank and HITS
### Due Tuesday, Nov 24, 2015, at 11:59 PM

In this assignment you will compute PageRank and HITS values for a portion of the Web graph. This assignment will be standalone: it will not build on your previous work and you do not need to integrate it with your previous inverted index work (at least, not yet). You should focus exclusively on generating an accurate set of PageRank and HITS (hub and authority) scores for the hyperlink graph that you are given.

**Note:** For this assignment, we highly recommend you to use your own laptop or CAEN’s computer to do the computation. 
You don't need Apache for this project, and the heavy computational load of this PA will place a 
strain on the eecs485 machines.

![Figure 1. A part of graph we will use](https://github.com/EECS485-Fall2015/admin/blob/master/pa5/pa5_graph.png)

## PageRank

### The Graph

We will provide you with two datasets for this assignment (You can find the links at the end of this manual). 
They are both derived from the hypertext graph described by Wikipedia pages, in which each page 
(e.g., the Barack Obama Wikipedia page) is a node and an intra-Wikipedia link is a directed edge in that graph. 
We have removed all of the links that connect the Wikipedia pages to outside non-Wikipedia sites.

There’s nothing special about using Wikipedia; PageRank works on any kind of hypertext network 
and is designed to work on the overall Web. However, Wikipedia is nice to use: the pages are 
easy to understand, the links are not spammy, the graph is interesting, and so on.

The first dataset, small, contains 362 nodes and all of the directed edges between them. 
It is small enough that you can trace connections by hand when debugging your code.

The second dataset, large, contains about 8M nodes and all of the appropriate directed edges. 
It will be harder for you to debug your code on large than on small. We strongly recommend that 
you get PageRank working on small first.

### The Code
You should implement this project in a traditional language: C, C++, or Java. Do not use PHP/Python.

You will receive two datafiles: small.net and large.net. 
The dataset is in the 'Pajek NET' format. The dataset is appropriate to 
represent the graph structure for our purpose. A simple example of the format from mapequation.org is as follows:

    *Vertices 6
    1 "Node 1" 1.0
    2 "Node 2" 1.0 
    3 "Node 3" 1.0 
    4 "Node 4" 1.0 
    5 "Node 5" 1.0 
    6 "Node 6" 1.0
    *Arcs 8
    1 2 2.0
    2 3 2.0
    3 1 2.0
    1 4 1.0
    4 5 2.0
    5 6 2.0
    6 4 2.0
    4 1 1.0

The two lines beginning with `*` tells us the number of nodes and the number of directed edges in the graph, respectively. From the 2nd line to the 7th line, each line is a pair of node id, node label, and node weight. From the 9th line to the end, each line corresponds to a directed edge with source node id, target node id, and edge weight. In both node lines and edge lines, the weights are optional, and in our dataset, the weights are not included.

The first few lines of our small.net file are as follows:

    *Vertices 362
    534366 "Barack_Obama"
    307 "Abraham_Lincoln"
    569 "Anthropology"
    863 "American_Civil_War"

You really only need the edge connectivity information in order to compute PageRank; 
the node labels aren't used by the algorithm. However, you will probably want to make sure 
that the resulting PageRank weights make sense, giving large weights to important topics and small 
weights to unimportant ones.  Using the node id to node label mapping will help you test and debug your results.

### PageRank Details

We went over the PageRank details in class. Be sure to keep in mind a few details:
Every node must have incoming links. However, you do not need to do this explicitly, 
as long as d < 1.0. The (1-d) term that describes the “teleport” quantity 
will implicitly add outgoing links from every node to every other node. 
The d term should be a command-line argument of your tool that we can change.
Every node must also have at least one outgoing link. In the event that a node has 
no outgoing links, create “virtual” outgoing links to every other node in the graph. 
For such nodes you will basically distribute all of its outgoing PageRank in the "teleport" mechanism. 
Make sure to remove all self edges before processing data.

You should implement two different stopping criteria: stop after K iterations, and stop after every node changes no more than X (X= (|current_round_value - previous_round_value| / previous_round_value). The values X and K should be command-line arguments of your tool that we can change.

### Result Output
Your program should write the answer to the output file as a pair of id, pagerank pairs. 
Each line in the output file should start with the integer id of a page, 
then a comma, then the pagerank value you have computed for that page. For example:

    1,0.18
    2,0.005
    .....
    and so on.

Your source code should be compiled by invoking ```make``` in the pa5_secretkey/pagerank directory 
(use a Makefile). After the compile is finished, 
your program should be invoked by a command-line tool called eecs485pa5p. 
If you are writing in C or C++, this tool can simply be your binary. 
If you are writing in Java, this tool should be a bash script that invokes the Java virtual machine.

The command-line tool should take the following set of arguments:

   `eecs485pa5p <dvalue> (-k <numiterations> | -converge <maxchange>) inputnetfile outputfile`

For example:

   `eecs485pa5p 0.85 -k 10 small.net pagerankSmallOut`

It should yield a PageRank computation 
over the small graph. It will use a d value of 0.85 and will stop after 10 iterations. 
It will write the results to a file called pagerankSmallOut.

We could also use this command line:

   `eecs485pa5p 0.7 -converge 0.01 large.net pagerankLargeOut`

It will use a different d value, will use a percent-change convergence criterion (|current_round_value - previous_round_value| / previous_round_value), and will operate on the large dataset.

**Note that** on the large graph, computing PageRank can take a long time. However, 
the running time for the large graph should be definitely under 2 hours. 
If you are taking much longer than this (perhaps each iteration takes more than 12 minutes), 
your code may need more attention.

## HITS

###  Graph

  The graph data provided for HITS is in the same 
  format as in PageRank. 
  We again use the 'Pajek NET' format to represent the graph. 
  However, since HITS constructs a seed set from the input query, 
  we additionally need text data to help choose the nodes. We pre-processed the wiki text data and 
  create an inverted index for you to load and use: no need to compute it yourself. You will use this 
  data to run HITS algorithm.

### The Code

As with PageRank, you should implement this project only in a traditional language: C, C++, or Java.

The inverted index data is in the format like:

    Acadians 5042916 
    Acadie 21184
    Acadien 21184
    Acadien 731832
    Acadiensis 21184
    AcadSA 17416221
    acad_pol 25417
    Acahualinca 21362
    Acajutla 9356
    acalorado 1889736 
    acamado 1523829 
    ...

Each line is a “tuple” where the first column is a word, and the
second column is the page id which contains the word. The same word
can appear many times in the file. Your query should be handled in
a case-insensitive manner, so bear that in mind when reading the
inverted index file. 

Your code should read both files and use them to compute hub and authority values.

### HITS Details

Follow the general algorithm for HITS covered in the class. A few things to keep in mind are as follows:

* Select the seed set by Boolean retrieval. Find top-h results from the given query ( documentIDs of results in increasing order). 
The set of query terms is given as a command line argument inside double quotes. 
To find the relevant results, use Boolean retrieval (you don't need to use TF-IDF scoring for this project). 
This set of result pages will become the seed set. 

* You should handle multiple-word queries as a conjunction. There is no need to handle them as phrase queries. When you try to load the whole index into memory, you may need to configure the memory limit for your program. 

* Grow the seed set into base set by including pages that are pointed to, or that point to, a page in the seed set. 
Now calculate hub and authority values iteratively. After every iteration of the algorithm, 
normalize hub and authority scores so that the sum of the squares of the hub scores is equal to 1 and the sum of the squares of the authority scores is equal to 1.

* You should implement two different stopping criteria: stop after K iterations, and stop after every node changes 
no more than X  (|current_round_value - previous_round_value| / previous_round_value). The values X and K should be command-line arguments of your tool that we can change.

* The running time of computing hub and authority scores should be about 10 minutes. If your program is taking much longer than this, you likely need to optimize your code.

* Please check [Here](https://github.com/EECS485-Fall2015/admin/blob/master/pa5/hits_supplement.md) for more details.

### Result Output

Your program should emit the answer as a pair of id, hub, authority pairs.  
Each line in the output file should start with the integer id of a page, 
followed by hub and authority values you have computed for that page (delimited by comma).  For example:

    1,0.18,0.02
    2,0.005,0.001
    ....

Your source code should be compiled by invoking make in the pa5_secretkey/hits directory 
(use a Makefile). After the compile is finished, your program 
should be invoked by a command-line tool called eecs485pa5h. If you are writing in C or C++, 
this tool can simply be your binary. If you are writing in Java, this tool should be 
a bash script that invokes the Java virtual machine.

The command-line tool should take the following set of arguments:

   `eecs485pa5h <h value> (-k <numiterations> | -converge <maxchange>) “queries” <input-net-file> <input-inverted-index-file> <output-file>`

For example:

   `eecs485pa5h 100 -k 10 “iPhone Microsoft” hits.net hits.inv hitsOutput`

It should yield a HITS computation over the hits.net graph and corresponding inverted index in hits.inv. It will use an h value of 100 and will stop after 10 iterations (h value was the value to obtain most relevant top-h results). It will write the results to a file called hitsOutput.

We could also use this command line:

  `eecs485pa5h 50 -converge 0.01 “iPhone Microsoft” hits.net hits.inv hitsOutput`

It will use a different h value, will use a percent-change convergence criterion (|current_round_value - previous_round_value| / previous_round_value).

## Folder Structure
* You should have 2 folders under your pa5_secretkey repo: pagerank/ and hits/
* There should be 1 makefile under each folder, 2 makefiles in total.
* You don't need to upload the graph files or inverted index files to your repo

## Sample Answers
Command: ```./eecs485pa5p 0.85 -k 10 small.net pagerank0```

Sample Answer:
```534366,0.03112
24113,0.02177
645042,0.02104
3414021,0.01802
18935732,0.01782
5043544,0.01703
31756,0.01683
......
```

Command: ```./eecs485pa5h 100  -k 10 "iPhone Microsoft" hits.net hits_invindex hits3```

Sample Answer:
```8841749,0.1212,0.1225
856,0.1187,0.2144
18394233,0.1074,0.0066
19006979,0.1070,0.0856
7878457,0.1018,0.03333
210892,0.09317,0.01894
20640,0.09002,0.2048
......
```

## Grading

* We will check your pa5_secretkey/pagerank and pa5_secretkey/hits directory for your command-line tool. 
* We will run it on graphs that may be different from the Wikipedia test graphs, and with different parameter settings than the ones in the sample. 
* You will receive full points for a test case if the average distance for each node does not exceed 3% (|ours - yours| / ours).
* You will receive 0 points for the test case if it exceeds 3%.
* Make sure to keep the code compileable and executable without any problems. 
* We will deduct points if you do not follow all of the format guidelines.
* We will be going into both your pagerank/ and hits/ folders and typing "make" to compile your code.
* Make sure your executables (or for Java, your bash script) are called "eecs485pa5p" (for PageRank) and "eecs485pa5h" (for HITS)!!!
* We will deduct points if we cannot compile the code by typing make in your pagerank/ and hits/ folders.
* For PageRank only, you won't receive credit if your runtime for 10 iteration on large.net exceeds 2 hours (it takes 4-15mins normally). There wil be no point deduction for running time under 2 hours.
* We highly highly highly recommend you test on CAEN as we will be grading on CAEN.
* There is no extra credit available for this assignment.

## Dataset
#### PageRank
* [Small wiki graph (61K uncompressed) ](https://drive.google.com/open?id=0Bw_xoHXcwyEIZ1BxLWs4cms0cDQ)
* [Large wiki graph (499M compressed)]( https://drive.google.com/open?id=0Bw_xoHXcwyEIdzdEV0xXalY1NmM)

#### HITS

* [Medium wiki graph (3M uncompressed)](https://drive.google.com/open?id=0Bw_xoHXcwyEIMWloVzdWTURDLTQ)
* [Inverted Index for the graph (15M compressed) ](https://drive.google.com/open?id=0Bw_xoHXcwyEIcndDazk4dmM0Yms)

