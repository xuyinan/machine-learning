# Assignment 6: The Next Great Search Engine
### Due Tuesday, Dec 8, 2015, at 11:59 PM

There are no late days for this assignment.

In this assignment you will build an integrated Web search engine that has several features:
* Ranking based on both tf-idf and PageRank scoring
* Indexing implemented with MapReduce so it can scale to very large corpus sizes
* An HTML and JavaScript-powered search engine interface with two special features: user-driven scoring and “deep page summarization”.

You have now learned enough in class to build a scalable search engine that is similar to the commercial systems out there. 

## Part 1.  Integrated Ranking
So far we have computed both tf-idf and PageRank, but we have not done so together. 
Now is the time to fix this oversight.  Your search engine should rank documents based on both 
the query-dependent tf-idf score as well as the query-independent PageRank score.  

The formula for the score of a query q on a single document d should be:

	Score(q, d) = (w * PageRank(d)) + ((1-w) * tfIdf(q, d))

where w is a number between 0 and 1.  This value w should be a query-time parameter of your search system. 
The final score contains two parts, one from pagerank  and the other from a tf-idf based cosine similarity score.
The PageRank(d) is the pagerank score of d. And the tfIdf(q, d) is the **cosine similarity** between query's and document's tf-idf vectors. 

In PA4, Your ranker received a query string q as input and emitted a ranked list of documents. 
In this project, you will still treat query q as a simple AND, non-phrase query as you did in PA4. 
However, your new ranker should also take a parameter w, then score documents according to the formula above. 

Your ranking results should be computed by a separate index-lookup-and-rank program, as you did in PA4. 
However in this project, you need to implement Indexer using MapReduce, and modify IndexServer a bit to emit 
correct search result. Same as PA4, you need to handle stop word and build a case-insensitive inverted index.

Integrating PageRank scores will require a second index, which maps each document id to its corresponding 
precomputed PageRank score. This index should be accessed at query time by your separate ranker program. 
This index should be loaded when the separate search ranker program starts up, then kept in memory.

You will build an index over the Wikipedia corpus that you used in PA5. 
However, you must now process the actual Wikipedia content, not just the link graph (described in Part 2). 

Your search engine interface should have a JavaScript-enabled slider that allows the user to adjust  the parameter w. 
Adjusting it should be optional: set it to a reasonable default and allow the user to decide whether or not to adjust it.

## Part 2.  MapReduce Indexing
Constructing an index over a very large corpus requires that we build it using many machines. 
The MapReduce framework offers a helpful tool for building the distributed indexer.

You should reimplement your inverted index software using the map reduce framework. You will not actually run your program on hundreds of nodes: It’s possible to run a MapReduce program on just one machine: your local one.  However, a good MapReduce program that can run on a single node will run fine on clusters of any size.  In principle, we could take your MapReduce program and use it to build an index on 100B Web pages.

We have provided a local installation of the Hadoop open-source project. A map reduce tutorial using Hadoop can be found in MapReduce.md file in this repo. You should download this directory ([hadoop-example.tar.gz](http://www-personal.umich.edu/~zhezhao/projects/hadoop-example.tar.gz)) that contains the sample hadoop code as well as all the data. (details for this are in MapReduce.md).

You should go through it and make sure you understand it and can run it before attempting to write your own indexing code. Once you have the sample Hadoop map reduce code running, you can copy the code and write your own indexing code by modifying it. Note that you will most likely need to write several Map Reduce programs (3-4) and run them in a pipeline to be able to generate the index files.

The input of your MapReduce program will be dataset/mining.articles.xml in the hadoop-example.tar.gz download ([hadoop-example.tar.gz](http://umich.edu/~rahuljha/hadoop-example.tar.gz)).
The output of your MapReduce program will be the inverted index that you load in Part 1. 

There is no need to reimplement PageRank in MapReduce: you can use the code you wrote in PA5. 
The input file to calculate pagerank score will be dataset/mining.edges.xml in the hadoop-example.tar.gz. 
The file contains multiple edges with tag "<eecs485_edge>". The format is different from the input data in PA5. You will need to modify your code from PA5 to take this new input format.

Hadoop MapReduce can use many different programming languages, 
but we recommend you use Java as shown in the example (in hadoop-example.tar.gz). Other Hadoop programs add deployment complexity that we may not be able to help with.

When implementing this part, please do not run MapReduce tasks on your assigned eecs485 server. Run it locally.

## Part 3.  The New Search Interface

The third component of your project is a new HTML interface for the search engine. This should roughly follow the standard Google model: a search box for text queries, a Search button, and a result list of ten blue links.

However, your interface should have two features that mark it as new.

1) **Slider**  The slider to control w, as described in Part 1.

2) **Deep Page Summary**  Unlike standard search engines, you know that your search engine will be used on Wikipedia pages.  Because Wikipedia pages have a more regular structure than standard Web pages, your summary snippet can be much more interesting than what Google can do for a standard page.

In particular, your search engine should:

* By default, show only the URLs on the search engine result page.
* When a special “show summary” button is clicked, use JavaScript to display several pieces of information: the first image from the page, the page “categories”, and all data from the Wikipedia page’s infobox. This information is available in different XML's in hadoop-example/dataset in hadoop-example.tar.gz. You should load the information for each article from these XML's into a database table (you can use a database from any of your previous projects for this. Mention the name of the database and table you use in your README file.)
* Have this information be “retractable” to minimize clutter on the screen.  That is, a click should show the extra info, and another click should hide it. 

## Grading

Please write a bash script to generate inverted index files and PageRank values in the directory `pa6_secretString/invoutput` and `pa6_secretString/proutput`. In other words, if we type

	eecs485pa6inv.sh

the output files from your inverted index MapReduce program should be stored in pa6_secretString/invoutput, and if we type

	eecs485pa6pr.sh

For this, you will have to commit your hadoop-example directory in your repo. For this, **remove the dataset directory (because it's very large!)** and commit the rest of the hadoop-example directory with your code files to your github repo.

Output files from your PageRank (non-MapReduce) program should be stored in pa6_secretString/proutput. Make sure if the scripts run OK when we type the same script repeatedly.

We will deduct you by 5 points if you do not follow the folder structure properly. You should make the cleaning task include removal of old output directories, etc. 

The example as to how to run MapReduce program is included in the code we provide.  
Modify them appropriately following the instructions here.

Also, we should be able to see the results of running your scripts by typing:

		cat invoutput/*

and

		cat proutput/*


## Important Directories and Files in Submission

### Github

* Your MapReduce code will most likely be in hadoop-example/mysrc directory. Commit your entire hadoop-example directory after removing the hadoop-example/dataset directory to Github. You need to remove hadoop-example/dataset because the files there are huge and will not allow you to commit. Mention the path to this directory relative to your root in Github in your README.
* Add the directory with your Pagerank source code to Github. Mention the directory name in your README. 
* As listed above, you should have scripts eecs485pa6inv.sh and eecs485pa6pr.sh in your root Github directory that when run, will compute the Pagerank and Inverted Index respectively. These scripts should use the code in hadoop-example and your Pagerank code to generate the desired output.
* The output files in pa6_secretString/invoutput/ and pa6_secretString/proutput/ should not be commited to Github.

### Server

* Please keep the files that have your inverted index (invoutput) in the eecs server and mention the path in README. This is the output of your Hadoop code and we will check this to make sure you have computed your inverted index properly.
* Please keep the files that have your Pagerank output (proutput) in the eecs server and mention the path in README. This is the output of your Pagerank code and we will check this to make sure you have computed the pagerank values correctly.
* You will be using your port numbers in PA4. please mention the urls in README.

## Submitting Your Assignment

Again, please keep your proutput and invoutput in eecs server, and specify the path in README. Put your scripts and java source code you made in pa6_secretString directory in Github. You can put MapReduce program or related library in any other place, but please keep your pa6_secretString directory well organized for us to grade faster.

The desired format of the output files is shown below, and the format will not be quite 
different from the one we followed in the previous assignment.

### For Inverted Index
Your MapReduce program should compute both inverted index itself and tf-idf scores for each pair of term and document. The output format should be as follows (the tf-idf scores in the below example are not correct):
	
	word	DF pageDid:tf-idf					
	yi	1 303:3.8291e-02
	york	2 359:9.6016e-02 303:3.6233e-02

Here DF, refers to the document frequency for the word or the number of documents that it appears in. Please note that different from PA4:
* 1. you are storing DF instead of IDF.
* 2. For each document, you are only storing normalized tf * idf score. where IDF can be calcualted based on DF and total # of document.
* 3. The tf-idf score is normalized (Same nomralization method as in PA4). There are many ways to do this in MapReduce. This also means you need to write more than one MapReduce jobs to get the correct results.
					
If stated in C printf format:
			
	printf("%s\t%d",word, document-frequency);
	for *number of posting list* {
		printf("%d:%.4e ", pageDid, tf-idf score);
	}
				
The tab separation between key and value is done by default.
					
### For PageRank					
Basically same as in the previous assignment.
	
	pageDid, PR_value				
	303, 3.2313e-02
	10293, 1.2933e-06
				
The only difference is that you should print 4 digits after decimal points. 
To generate good pagerank result, we recommend you to run 50 iterations with d set to 0.85.

### README format

In the `README.md` at the root of your repository please provide the following details:

* Group Name (if you have one)
* List `User Name (uniqname): "agreed upon" contributions`.
* Details about how and if you deviated from this spec - avoid if possible.
* Extra details about how to clone and run your code - simple as possible.
* Anything else you want us to know.
* The formatting is not critical, we just need the information.

```
Group Name:   
  Rabid Ocelots
Members:
  Otto Sipe (ottosipe): setup the database, setup the routes, did the project alone  
  ...
Details: 
  The homepage URL for this project is ...
  We called our /pic endpoint /foto
  We implemented the extra credit...
Deploy: 
  (just an example)
  pip install < requirement.txt
  foreman start
Extra:
  
```
