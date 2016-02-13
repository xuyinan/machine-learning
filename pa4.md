# Assignment 4: Information Retrieval

### Due Tuesday, Nov 10th, 2015, at 11:55:00 PM EDT

In this assignment you will learn how to build and maintain an inverted index, and use it to do simple search queries.

## Overview

For this assignment, you will allow users to search for photos with a
text query. You will implement this search feature with an inverted
index that will enable efficient retrieval of documents relevant to
the user's query. 

Your tool should run in two phases: "index-time" and "query-time".
The index-time phase occurs after your search engine has acquired a
text corpus, prior to any query processing. Your code should read the
collection of text, compute the index, and write it to disk. Then
later, immediately prior to query-time query processing, your search
application should load the index from disk into memory.

(Note that unlike many search engines, the corpus will be small enough
that you can build the entire index in memory.)

For query-time processing, when a user enters a search query, you will use this loaded in-memory
inverted index to rank photos for the user. You should load the
following XML file of 200 photos with captions into your image
database (perhaps the photo table) and reuse your PA1 database for this project. 
These files are available in the github repo for this project at the following locations:

* XML file to load: resources/search.xml
* Actual images: resources/flickr-images/*
* Portion of the sample Inverted Index: sample.txt

You will index all of the captions for this set of photos. Treat each
caption as a separate (small) document, with its own doc-id, etc. A
"hit" with this search index will be a photo, not a traditional web
page. All the files related to your inverted index should be stored in
the pa4/index/ subdirectory. The sample.txt file above describes
a sample inverted index format. (Note that the log for IDF for the sample is computed with base 10). Your data structure should probably be
roughly similar, but does not have to be exactly the same; you may,
for example, want to add special query features that require
additional information in the data structure. Of course, the contents
of the index will depend on the documents you process.

When building the inverted index file, you should use a list of
pre-defined stopwords to remove words that are so common that they do
not add anything to search quality ("and", "the", "etc", etc). You can
find a list of these words at: 

* [Stop words](http://jmlr.csail.mit.edu/papers/volume5/lewis04a/a11-smart-stop-list/english.stop)

You can find useful information about this assignment from this book:
* [Information Retrieval](http://nlp.stanford.edu/IR-book/information-retrieval-book.html)

Unlike earlier PAs, only a small amount of your code for PA4 will be
in Python or JavaScript. You can choose to write most of your code in
either C++ or Java. 

## System Design

To get you started, we have provided a skeleton code framework. It consists of some source code as well as compiled libraries. The following diagram shows the relevant software components.

![Framework](http://www-personal.umich.edu/~chjun/eecs485/p1.png)

Figure 1: Framework of the system

**`Indexer`**: This is where you create your inverted index. 
The input of Indexer is the captions file which you download from images and search.xml, 
and the output is the inverted index file. The indexer would only be run once to generate the inverted index. 

NOTE: When creating your index you should treat capital and lowercase letters as the same (case-insensitive). You should also only include alphanumeric words in your index and ignore any other symbols.   

**`IndexServer`**: The IndexServer loads an inverted index from disk and uses it to process 
search queries. When the IndexServer is run, it will load the inverted index file into memory 
and wait for queries. Every time you make an appropriate call using the Python library, 
the IndexServer will invoke a virtual processQuery() method that you will implement. This processQuery() method is provided with the user's search string, and must return a list of all the "hit" docids, along with relevance scores. The results of the search() method are then returned to your web app via the network and the Python API. 

**`Web Pages`**: You will create a search front-end in Python. 
It contacts a back-end server that processes search queries and returns 
hits to your web app. Your code will then render the results onscreen to the user. 
We have provided a Python library to you that hides the details of
communicating with the back-end server; you should simply pass in a
target portnumber and a search string. 

Here are the libraries and source files you will need. We will explain how to use these later in the spec.

* C++ Library: pa4_CPP
* Java Library: pa4_Java
* Python connector: http://www.python-requests.org/en/latest/ 

### How to use the provided Library (Java and C++)

You may write your project in Java or C++. There are language-specific
sets of code and libraries to help you. Clearly, you should pick one
language and just one library. 

#### For the Java Library:

Under pa4_Java, the only two files that you should add code into are
src/edu/umich/eecs485/pa4/Indexer.java and
src/edu/umich/eecs485/pa4/IndexServer.java.  
Please don't modify any other file in the library. 

NOTE: If you use Eclipse as your IDE, you might have to add the lib/json-simple-1.1.1.jar manually to the build path.

**`Indexer.java, IndexServer.java`**: You will need to fill out some specific functions. The code contains hints as to where you should place your own code. Please do not modify the command-line parser or the constructor. You may add other functions or data structures to these source files. You should be able to build and run your indexer and index server without modifying any other files under the pa4 folder. 

1) Build and Run: Go to pa4 folder,
    To build your code: `ant dist`

2) To run the indexer:

    java -cp dist/lib/pa4.jar:lib/json-simple-1.1.1.jar edu.umich.eecs485.pa4.Indexer <input-content-file> <index-file>

3) To run the server:

    java -cp dist/lib/pa4.jar:lib/json-simple-1.1.1.jar edu.umich.eecs485.pa4.IndexServer <portnum> <index-file>

The port number you pick will be the port number that your IndexServer listens on. Thus, it must be different from the HTTP port numbers that your web server opens. Add 200 to your PA3 port numbers to get the port number for the IndexServer. Specify the port number you chose in your README.

#### For C++ Library:

After unzipping the pa4_C++.tar.gz, you will get a pa4 folder. Under pa4, the only 
four files that you should add code into are `Indexer.cpp&Indexer.h and Index_server.cpp&Index_server.h`, and you only need to add your code 
in Index_server::init, Index_server::process_query and Indexer::index, and add other functions and data structures as necessary. However, you cannot delete any function 
from the original files. We will use the provided MakeFile to build your code (if you feel the need to use C++11 features you will need to modify your makefile).

1) Build and Run: Go to pa4 folder, To build your code:
      `make`
      
2) To run the indexer:
    `./indexer input.txt output.txt`

3) To run the server:
    `./indexServer <portnum> <indexfile>` 

The port number you pick will be the port number that your IndexServer listens on. Thus, it must be different from the HTTP port numbers that your web server opens. Add 200 to your PA3 port numbers to get the port number for the IndexServer. Specify the port number you chose in your README.

####To connect your Python code to the IndexServer: (For Java and C++)

Here are the steps you should follow: 

* `import requests`   //enable you to use the library
* `result = requests.get('http://SERVERHOSTNAME:PORTNUMBER/search?q=QUERY')` //send a query to the server
* result is a Response object. You can use `print result.text` to see the details.


## Search

After finishing the previous part, you will have a small search engine. 
Please create a new web page `/search` to be your search homepage. 

For the PA4 website, use the same port numbers that you used for PA1. You might have to kill the server running for PA1 for this to work.

For creating the PA4 search homepage, copy over the code from PA3 the same way you copied over code from PA1 to PA2. Also, be careful not to push code to your PA3 github repo. You'll only be using the Klein/Smarty/Flask skeleton code for this project and won't use much of the code from PA3, but you don't have to delete anything, just add the new search page.

In this project, you can assume anyone who is in `/search` can use your search engine.

Your engine should receive a simple AND, non-phrase query (assuming the words in a query are independent), 
and return the ranked list of images. No in-doc position information is necessary for the inverted index. 
However, frequency information IS necessary inside the inverted index, so that tf-idf can be computed. 


### Using TF-IDF to score the documents

We use the normalized similarity formula to calculate the sim(Q, D) score.

It is critical that your search feature use the inverted index for answering all queries. 
Because the set of photos is relatively small, you could probably iterate directly through all of the 
comment strings without a huge computational burden. However, the purpose of this assignment is to learn 
about the inverted index. With an inverted index, we could scale your
system up to very large document collections; that scaling would be impossible
if you simply examine the comment strings directly. If your search
feature simply examines the comment strings directly, you will not receive any credit.

### Finding Similar Pictures

Your photo should also implement a feature for retrieving photos that
are similar to a given "query photo".  That is, after clicking a
picture the user should be able to see a list of photos that are the
most relevant to the picture. Recall that to the search engine, a
"query" and a "document" are basically the same thing; to implement
this feature, simply take the caption string as a query, and
return the results. 

### Search Results
The SERP (Search Engine Results Page, `/search`) should present a
nice-looking list of hits, with photo thumbnails next to each.  Please
show the number of results you get, as Google does when processing a
search query.  

## Extra Credit

Extra credit is given to the groups that can add one or more of the following features to their websites. There is a 2% bonus for each of the features. As in the previous projects, extra credit will only go towards missed points on the project and not towards your overall course grade.

* Implementing phrase queries.  This requires modifying the inverted index so that it maintains position information.
* Updating the index file in place: Upon modifying a caption, the user should be able to update the index file with the new caption, without reindexing the entire caption set.

If you have successfully implemented at least one of above mentioned, please describe it in README.txt 
including where to, how to, and what to check. 

## Submitting Your Assignment

In the `README.md` at the root of your repository please provide the following details:
* **Link to Homepage (We will deduct you by 10 points if you do not have this)**
* Specify whether you used Java or C++ to create files
* Specify the database table where you've stored the images
* Group Name (if you have one)
* List `User Name (uniqname): "agreed upon" contributions`.
* Details about how and if you deviated from this spec - avoid if possible.
* Extra details about how to clone and run your code - simple as possible.
* Anything else you want us to know, like how many late days you took.

```
Link to Homepage:
  http://eecs485 ...
Group Name:   
  Rabid Ocelots
Members:
  Otto Sipe (ottosipe): setup the database, setup the routes, did the project alone  
  ...
Details: 
  We called our /pic endpoint /foto
  We implemented the extra credit(2)
Deploy: 
  (just an example)
  pip install < requirement.txt
  foreman start
Extra:
  I took 2 late days.
