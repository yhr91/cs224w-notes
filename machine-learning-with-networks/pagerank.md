---
layout: post
title: PageRank
---

In this section, we study PageRank which is a method for ranking webpages by the web graph link structure.  This is very useful for ranking search results and was developed by Google founders Larry Page and Sergrey Brin in 1996. Before discussing PageRank, let us first conceptualize the web as a graph and attempt to study its structure using the language of graph theory.

## Web as a graph

We can represent the entire web as a graph by considering the web pages as nodes and the hyperlinks connecting them as directed edges. We will also make a set of simplifying assumptions for the sake of this analysis

Assumptions:
- Every page has a unique URL
- Consider only statics webpages
- Ignore dark matter on the web (i.e. inaccessible material, pages behind firewalls)
- All links are navigable. Transactional links (eg: like, buy, follow etc.) are not considerd

Conceptualizing the web this way is similar to how some popular search engines view it. For instance, Google creates a webgraph by indexing webpages. For this, it uses crawlers that consume links and do a breadth-first search (BFS) exploration of the web. Some other examples of graphs that can be viewed this way are: the graph of citations between papers, references in an encyclopaedia

## What does the graph of the Web look like

In the year 2000, the founders of a popular search engine at that time - AltaVista - ran an experiment to explore what the Web looked like. 

They asked the question: given node $$v$$, what are the nodes $$v$$ can reach? What other nodes can reach $$v$$?

This produced two sets of nodes:
$$In(v) = { w| w can reach v}$$
$$Out(v) = { w| v can reach w}$$

This can be ascertained by running a simple BFS. In the following graph:

In(A) = {A,B,C,E,G}
Out(A) = {A,B,C,D,F}

Now, there are two types of directed graphs
- Strongly connected graph: A graph where any node can reach any other node. 
- Directed Acyclic Graph (DAG): A graph where there are no cycles and if $$u$$ can reach $$v$$, then $$v$$ cannot reach $$u$$

Any directed graph can be represented as a combination of these two types. To do so: 
- Identify the strongly connected components within a directed graph. A strongly connected component is a set of nodes S in a graph G that satisfy this property such that there is no larger set in G containing $$S$$ with this property. This is equivalent to performing a BFS on all the nodes with in-links into a given node and all the edges with out-links from the same node and then calculating the intersection over these two sets.
- Merge these into supernodes, creating a new graph G'. Create an edge between the nodes of G' if there is an edge between corresponding SCCs in G.
G' is now a DAG

Broder et al. (!999) took a large snapshot of the web and tried to understand how the SCCs fit together as a DAG

## Bowtie structure of the graph
-	Their finding are presented in the figure below. Here the starting nodes are sorted by the number of nodes that BFS visits when starting from that node
-	If you look at the nodes on the left colored blue, they are able to reach a very small number of nodes before stopping
-	If you look at the nodes on the left colored blue, they are able to reach a very large number of nodes before stopping
-	Using these plots we can determine the number of nodes in the IN and OUT component of the bowtie shaped web graph shown below. 
- SCC corresponds to the largest strongly connected component in the graph.
- The IN component corresponds to nodes which have out-links to SCC but no in-links from SCC. The OUT component corresponds to nodes which have in-links from SCC but no out-links to SCC. 
- Nodes with both in and out links to the SCC fall in SCC.
- Tendrils correspond to edges going out from IN or those going into OUT. TUBES are connections from IN to OUT that bypass SCC.
-	Disconnected components are not connected to SCC at all

## PageRank - Ranking nodes on the graph

Not all pages on the web are equal. When we run a query, we don’t want to find all the pages that contain query words but we also want to learn how we rank the pages. How can we decide on importance of pages based on the link structure of the web graph

The main idea here is that links are votes

In-links into a page are counted as votes for that page which help determine the importance of the page. In-links from important pages count more turning this into a recursive question: A's importance depends on B, whose importance depends on C and so on.

Each link's vote is proportional to the importance of its source page. In the figure below, node j has links from i and k which each contribute importance values equal to $r_i/3$ and $r_k/4$. This is because there are three out-links from node $r_i$ and four out-links from node $k$. Similarly node j's importance is euqally distirbuted between the three out-links that it shares with other nodes.

To summarize, a page is important if it is pointed to by other important pages. Define a rank r_j for node j as:
$$ eqn $$

### Matrix formulation
We can formulate this as N PageRank equations using N variables. We could use Gaussian elimination to solve this however however it would take a very long for a large number of pages such as those in the web graph. 

Instead, we can represent the graph as an adjacency matrix M that is column stochstic. This means that all the columns must add up to 0. So for node 'j' all the entries, in column 'j' will sum to 0. This will fulfill the requirement that the importance of any node must sum to 1. We now rewrite the PageRank vector as the following matrix euqaion:
r = Mr

In this case, the PageRank vector will be the eigen vector of the stochastic web matrix M that corresponds to the eigen value of 1. 

<EV formulation slide text>

### Random walk formulation

PageRank relations are very related to RandomWwalks. Imagine a webgraph and a random surfer. The surfer surfs the graph randomly. At any time (t) he is at any page (i). Then the surfer will select any outgoing link, pick it at random and make a new step, and this process continues infinitely. Let p(t) be the probability that a surfer is at a given page I at time t. P(t) is a probability distirubtion over pages at agiven time t.
So p(t+1) = M p(t)
The probability that at (t+1) the surfer is at page (j) will depend on the prob that the surfer was at pages(i) at time (t) and their out-degree. After some time the random walk will reach some steady state. It will converge. 
So, if we solve r=Mr, r is really just the probability distribution of where this surfer will be at time t. It’s modelling the stationary distrubtion of this random walker process on the graph and will converge to a distribution where the time doesn’t matter anymore, if you let it walk long enough.

### Power law iteration

In other words, r is the limit when we take a vector u and multiply with M long enough. This means that we can efficiently solve for r using power iterations. We keep iterating until we converge based on epsilon. In practice, this tends to converge within 50,100 iterations. This is great because we don’t need to solve a system of equations. We’re just multiplying a vector with a matrix enough times

<Add a vector example to show the math>
  
The r you are left with is the page importances. So page 1 has importance 6/15, page 2 has ….

### PageRank: Problems

1.	Dead ends: These are pages that you can get into but have no out-links. As a random surfer, it’s like coming to a cliff and having nowhere else to go. This will leak out importance. Mathematically, this would imply that matrix is no longer column stochastic. 
2.	Spider traps: These are pages with only seld-edges as outgoing edges causing the surfer to get trapped. Everntually the spider trap will absorb all importance. Imagine I have this graph with a self-loop in B. RS starter somewhere and when he navigates to b he gets stuck in b for the whole time. Powere iteration will converge with b having all the importance but a has no importance. But we don’t want this.

How do we solve this: using random teleportation or random jump

Whenver a RW makes a step, the surfer has two options. The walker can come, flip a coin and with probability Beta, it will keep following the links and with (1- beta) it will teleport to a different webpage. Usually this is around 0.8 to 0.9. This can be visualized as jumping out from any node. 

In spider trap, teleport out in a finite number of steps

How to solve dead end: Follow random teleport links with total probability from 1.0. So if you come to a dead end always teleport. Where do you juimp – to any of the nodes with equal probability. To make the matrix column stochastic the solution is by always teleporting when there is nowhere else to go

Importance of j is B times taking one of the outlinks and 1-B probability of jumping anywhere else in the graph
So we now define a google matrix A which is related to the page rank matrix from before. So, even after adding teleportation we can use the eigenvector

### Computing PageRank: Sparse matrix formulation

The key step is matrix-vector multiplication
 rnew = A rold
We want to be able to iterate this as many times as possible. If A, r_old r_new are small and can fit in memory then there is no problem. But if N = 1 billion pages and each entry is 4 bytes. Just for storing r_old and R-new, we would need 8GB of memory. Matrix A has N^2 = 10^{18} entries. This is a large number, close to 10millionGB of memory! 

We can rearrange the computation to look like this:

𝒓 = 𝜷𝑴 ⋅ 𝒓 + [𝟏 − 𝜷/𝑵]/𝑵

This is easier to compute because M is a sparse matrix, multiplying it with a scalar is still sparse, and then multiplying it with a vector is not as computationally intensive. After this, we simply add a constant which is the probability of the random walker directly jumping to r_new. So, now the amount of memory that we need goes down from N^2 to O(N). At every iteration some of the pagerank can leak out and by renomalizing M we can re-insert the leaked page rank.


M is a sparse matrix! (with no dead-ends)
- 10 links per node, approx 10𝑁 entries

So in each iteration, we need to:
- compute rnew = b M · rold
- add a constant value (1-b)/N to each entry in rnew

Here is an example of how PageRank would work if applied to a graph:
Numbers sum to 1000, size of the node is proportional to the number. These are pagerank scores. Node B has veyr high importance because a lot of nodes point to it. These nodes still have importance without in-links because random jump can jump to them. Node C has only one-link but since its from B it also becomes very important. But this is less than B which has a ot of in-links goinginto it.

<Insert the complete algorithm>
  
  
RandomWalk with restarts and Personalized PageRank

Imagine you have a grpah of computer science conferences and authors with professors that publish at those conference. So what is the most related conference to the conference ICDM, given who they publish with and where are the other conferences that htey publish at. This kind of biparitie grpah is like a user-item graph. What is the most related produce for this particular user.

So lets look at this as a bipartite graph - and ask how related are two items or how related are towo users. Given that the ysers have already purchased in the past - what can I recommedn to them, based on what they have common with other users. So, are items A and A' more related than B and B'. 

We can use shortest path or number of common neighbors to determine this. What kinf of graphs would obey this kind of intution. It turns out that random walk on graphs will obey this pageRank. PErsonalized PageRank, ranks proximity of nodes to the teleport nodes S. So I start a random walk at node A and then whenever I teleport I go back to A. This will give me all the nodes that are most similar to A. 

One way to implement this is to take the teleport set and compute the pagerank vector using power iteration. but its quicker to just do a simple random walk

So we would pick the nodes with the higest visit counts. So, this results in a very simple recommender system that works very well in practice. 

The teleport set can also consist of multiple starting poiints

Normal pagerank:
teleportation vector is uniform

personalized pagerank: teleport to a topic specific set of pages. 
nodes can have different probabilities of surfer landing there

random walk with restarts:
- topic specific pagerank where teleport is always to the same node. In this case, we dont need power iteration we can just use random walk and its very fast and easy.
  
