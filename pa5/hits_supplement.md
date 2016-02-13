
### Finding the Relevant Subgraph

The first step in HITS is finding a subgraph of the overall web graph. 
This subgraph is tailored to be highly relevant to the user's query. In class I presented the following method for computing the subgraph:

a) Find all pages that contain the user's query. This is the Seed Set, sometimes also called the Root Set.

b) For each page in the Seed Set, find all pages that are connected to it. Add these new pages to the Base Set. 
Also add everything in Seed Set to the Base Set.

**There are two subtleties here:**

* When computing the Seed set, you should limit yourself to h items, where h is provided by command line parameter.

* You may find that some queries have items in the Seed set that are much more popular than others. 
As a result, the contents of the Base Set can be quite lopsided. 
E.g., if "yahoo.com/index.html" is in the Seed set, then it will bring a huge number of pages into the Base Set, 
far more than any other page in the Seed set. To address this problem, it is OK to limit the number of Base 
Set pages "brought in" by a single page. So we'll revise the above algorithm slightly:

  * Find all pages that contain the user's query. This is the Seed Set, sometimes also called the Root Set.
  * For each page in the Seed Set, find up to 50 pages (pages are ordered by ID) that are connected to it. Add these new pages to the Base Set.
Also add everything in Seed Set to the Base Set.

This modification to step (b) will prevent a single popular page from swamping the results.


We then compute Hubs and Authority scores on items in the Base Set.

### Normalizing Hub and Authority Scores

 
The lecture notes and text discuss normalizing Hub and Authority scores after each round. Here is what you should do.
Recall the basic algorithm for computing hubs and authorities:

 * Initialize all Hub and Authority scores for each page to 1

 * Loop until scores converge:

  * The Authority score for each page x is the sum of the (previous round) Hub scores for each page y where there's a link y->x.

  * The Hub score for each page x is the sum of the (previous round) Authority scores for each page y where there's a link x->y.

  * I) Normalize all the Authority scores

  * II) Normalize all the Hub scores


None of this should be controversial so far. 

We want the sum of all squares of Authority scores to be 1. Same with Hub Scores. So for (I) and (II) above:

 * I) Divide each Authority score by the square root of the sum of all the squares of the Authority scores.

 * II) Divide each Hub score by the square root of the sum of all the squares of the Hub scores.

For example, imagine the initial case where each Hub score is 1. If there are 5 nodes, 
then the normalizer is sqrt(1^2 + 1^2 + 1^2 + 1^2 + 1^2) == sqrt(5) == 2.236. 
After dividing each Hub score by this value, each Hub score will be 0.448. The sum of squares of Hub scores will be 1.
