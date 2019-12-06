---
layout: post
title: PageRank
---

In this section, we study PageRank which is a method for ranking webpages by importance using link structure in the web graph. This is a commonly used [algorithm](http://ilpubs.stanford.edu:8090/422/) for web search popularized by Google. Before discussing PageRank, let us first conceptualize the web as a graph and attempt to study its structure using the language of graph theory.

## The web as a graph

We can represent the entire web as a graph by letting the web pages be nodes and the hyperlinks connecting them be directed edges. We will also make a set of simplifying assumptions:

- Consider only static webpages
- Ignore dark matter on the web (i.e. inaccessible material, pages behind firewalls)
- All links are navigable. Transactional links (eg: like, buy, follow etc.) are not considered

Conceptualizing the web this way is similar to how some popular search engines view it. For instance, Google indexes web pages using crawlers that explore the web by following links in a breadth-first order. Some other examples of graphs that can be viewed this way are: the graph of citations between scientific research papers, references in an encyclopedia.

### What does the graph of the Web look like

In the year 2000, the founders of AltaVista ran [an experiment](https://www.sciencedirect.com/science/article/abs/pii/S1389128600000839) to explore what the Web looked like. They asked the question: given a node $$v$$: what are the nodes $$v$$ can reach? What other nodes can reach $$v$$?

This produced two sets of nodes:

$$
In(v) = \{ w| w \textrm{ can reach } v\}\\
Out(v) =\{ w| v \textrm{ can reach } w\}
$$

These sets can be ascertained by running a simple BFS. For example, in the following graph:

![BFS](../assets/img/BFS.png?style=centerme)

$$
In(A) = {A,B,C,E,G}\\
Out(A) = {A,B,C,D,F}
$$

### More on Directed graphs

There are two types of directed graphs:
- \textbf{Strongly connected graphs}: A graph where any node can reach any other node. 
- \textbf{Directed Acyclic Graph (DAG)}: A graph where there are no cycles and if $$u$$ can reach $$v$$, then $$v$$ cannot reach $$u$$

Any directed graph can be represented as a combination of these two types. To do so: 
1. <b>Identify the strongly connected components (SCCs) within a directed graph</b>: An SCC is a set of nodes $$\textbf{S}$$ in a graph $$\textbf{G}$$ that is strongly connected and that there is no larger set in $$\textbf{G}$$ containing $$\textbf{S}$$ which is also strongly connected. SCCs can be identified by running a BFS on all in-links into a given node and another BFS on all the out-links from the same node and then calculating the intersection over these two sets.
2. <b>Merge the SCCs into supernodes, creating a new graph G'</b>: Create an edge between the nodes of G' if there is an edge between corresponding SCCs in G.
G' is now a DAG

![SCC_DAG](../assets/img/SCC_DAG.png?style=centerme)

### Bowtie structure of the web graph

Broder et al. (!999) took a large snapshot of the web and tried to understand how the SCCs fit together as a DAG

-	Their findings are presented in the figure below. Here the starting nodes are sorted by the number of nodes that BFS visits when starting from that node

![BowTie_1](../assets/img/BowTie_1.png?style=centerme)

-	If you look at the nodes on the left colored blue, they are able to reach a very small number of nodes before stopping
-	If you look at the nodes on the left colored purple, they are able to reach a very large number of nodes before stopping
-	Using these plots we can determine the number of nodes in the IN and OUT component of the bowtie shaped web graph shown below. 

![BowTie_2](../assets/img/BowTie_2.png?style=centerme)

- SCC corresponds to the largest strongly connected component in the graph.
- The IN component corresponds to nodes which have out-links to SCC but no in-links from SCC. The OUT component corresponds to nodes which have in-links from SCC but no out-links to SCC. 
- Nodes with both in and out links to the SCC fall in SCC.
- Tendrils correspond to edges going out from IN or those going into OUT. TUBES are connections from IN to OUT that bypass SCC.
-	Disconnected components are not connected to SCC at all

## PageRank - Ranking nodes on the graph

Not all pages on the web are equal. When we run a query, we don’t want to find all the pages that contain query words. Instead, we want to learn how we rank the pages. How can we decide on the importance of pages based on the link structure of the web graph?

<b>The main idea here is that links are votes</b>

In-links into a page are counted as votes for that page which helps determine the importance of the page. In-links from important pages count more. Thus, this turns this into a recursive relationship: page A's importance depends on page B, whose importance depends on page C and so on.

Each link's vote is proportional to the importance of its source page. In the figure below, node j has links from i and k which each contribute importance values equal to $$\frac{r_i}{3}$$ and $$\frac{r_k}{4}$$. This is because there are three out-links from node $$i$$ and four out-links from node $$k$$. Similarly, node $$j$$'s importance is equally distributed between the three out-links that it shares with other nodes.

![LinkWeights](../assets/img/LinkWeights.png?style=centerme)

To summarize, a page is important if it is pointed to by other important pages. We can define the page rank $$r_j$$ for node $$j$$ as:

$$ 
\textbf{r_j} = \sum_{i\rightarrow j}\frac{\textbf{r_i}}{d_i} 
$$

where $$d_i$$ is the out degree of node i

### Matrix formulation
We can formulate this as N PageRank equations using N variables. We could use Gaussian elimination to solve this however however it would take a very long for a large number of pages such as those in the web graph. 

Instead, we can represent the graph as an adjacency matrix M that is column stochstic. This means that all the columns must add up to 1. So for node $$j$$ all the entries, in column $$j$$ will sum to 1. 

$$
\textrm{If } j \rightarrow i \textrm{ , then } \textbf{M}_{ij} = \frac{1}{d_j}\\
$$

$$r_i$$ is the importance score of page $$i$$

$$
\sum_i \textbf{r_i} = 1
$$

This will fulfill the requirement that the importance of any node must sum to 1. We now rewrite the PageRank vector as the following matrix equation:

$$
\textbf{r} = \textbf{M}.\textbf{r}
$$

In this case, the PageRank vector will be the eigen vector of the stochastic web matrix $$\textbf{M}$$ that corresponds to the eigen value of 1. 

### Random walk formulation

PageRank relations are very related to random walks. Imagine a web graph and a random surfer. The surfer surfs the graph randomly. At any time (t) he is at any page (i). Then the surfer will select any outgoing link, pick it at random and make a new step, and this process continues infinitely. Let p(t) be the probability that a surfer is at a given page I at time t. P(t) is a probability distribution over pages at a given time t.
So p(t+1) = M p(t)
The probability that at (t+1) the surfer is at page (j) will depend on the prob that the surfer was at pages(i) at time (t) and their out-degree. After some time the random walk will reach some steady state. It will converge. 
So, if we solve r=Mr, r is really just the probability distribution of where this surfer will be at time t. It’s modeling the stationary distribution of this random walker process on the graph and will converge to a distribution where the time doesn’t matter anymore, if you let it walk long enough.

### Power law iteration

In other words, r is the limit when we take a vector u and multiply with M long enough. This means that we can efficiently solve for $$\textbf{r}$$ using power iterations. 

Starting from any vector $$u$$, the limit $$\textbf{M}(\textbf{M}(… \textbf{M}(\textbf{M} \textbf{u})))$$ is the long-term distribution of the surfers. 

<b>Limiting distribution = principal eigenvector of $$\textbf{M}$$ = PageRank</b>

Initialize: $$ \textbf{r}^{(0)} = [\frac{1}{N},...,\frac{1}{N}]^T$$\\
Iterate: $$\textbf{r}^{(t+1)} = \textbf{M}.\textbf{r}^{(t)}$$\\
Stop when: $$|\textbf{r}^{(t+1)} - \textbf{r}^{(t)} | < \epsilon $$

We keep iterating until we converge based on epsilon. In practice, this tends to converge within 50-100 iterations. This is great because we don’t need to solve a system of equations. We’re just multiplying a vector with a matrix enough times

### Example

![PRExample1](../assets/img/PRExample1.png?style=centerme)
![PRExample2](../assets/img/PRExample2.png?style=centerme)
  
The $$\textbf{r}$$ vector you are left with is the page rank or page importances. So page $$y$$ has importance $$\frac{6}{15}$$, page $$a$$ has importance $$\frac{6}{15}$$ and page $$m$$ has importance $$\frac{3}{15}$$


### PageRank: Problems

1.	Dead ends: These are pages that you can get into but have no out-links. As a random surfer, it’s like coming to a cliff and having nowhere else to go. This will leak out importance. Mathematically, this would imply that matrix is no longer column stochastic. 
2.	Spider traps: These are pages with only self-edges as outgoing edges causing the surfer to get trapped. Eventually the spider trap will absorb all importance. Imagine I have this graph with a self-loop in B. Random surfer starts somewhere and when he navigates to b he gets stuck in b for the whole time. Power iteration will converge with b having all the importance but a has no importance.

How do we solve this: using random teleportation or random jump

Whenver a RW makes a step, the surfer has two options. The walker can come, flip a coin and with probability Beta, it will keep following the links and with (1- beta) it will teleport to a different webpage. Usually this is around 0.8 to 0.9. This can be visualized as jumping out from any node. 

In spider trap, teleport out in a finite number of steps

How to solve dead end: Follow random teleport links with total probability from 1.0. So if you come to a dead end always teleport. Where do you juimp – to any of the nodes with equal probability. To make the matrix column stochastic the solution is by always teleporting when there is nowhere else to go

Importance of j is B times taking one of the outlinks and 1-B probability of jumping anywhere else in the graph
So we now define a google matrix A which is related to the page rank matrix from before. So, even after adding teleportation we can use the eigenvector

### Computing PageRank: Sparse matrix formulation

The key step in computing page rank is the matrix-vector multiplication
 rnew = A rold
We want to be able to iterate this as many times as possible. If A, r_old r_new are small and can fit in memory then there is no problem. But if N = 1 billion pages and each entry is 4 bytes. Just for storing r_old and R-new, we would need 8GB of memory. Matrix A has N^2 = 10^{18} entries. This is a large number, close to 10millionGB of memory! 

We can rearrange the computation to look like this:

𝒓 = 𝜷𝑴 ⋅ 𝒓 + [𝟏 − 𝜷/𝑵]/𝑵

This is easier to compute because M is a sparse matrix, multiplying it with a scalar is still sparse, and then multiplying it with a vector is not as computationally intensive. After this, we simply add a constant which is the probability of the random walker directly jumping to r_new. So, now the amount of memory that we need goes down from N^2 to O(N). At every iteration some of the pagerank can leak out and by renormalizing M we can re-insert the leaked page rank.

Here is an example of how PageRank would work if applied to a graph:
Numbers sum to 1000, size of the node is proportional to the number. These are pagerank scores. Node B has veyr high importance because a lot of nodes point to it. These nodes still have importance without in-links because random jump can jump to them. Node C has only one-link but since its from B it also becomes very important. But this is less than B which has a ot of in-links goinginto it.

### PageRank Algorithm

Input: Graph G and parameter \Beta
Directed graph G (can have spider traps and dead ends)
Parameter \Beta

Output: PageRank vector r^{new}
Set: r_j^{old} = \frac{1}{N}:

Repeat until convergence: \sum_j|r_j^{new} - r_j^{old}| < \epsilon

\forall j: r_j^{'new} = \sum_{i\rightarrowj}\Beta \frac{r_i^{old}}{d_i}
	   r_j^{'new} = 0 if in-degree of j is 0

Now re-insert the leaked PageRank:
\forall j: r_j^{new} = r_j^{'new} + \frac{1-s}{N} where S = \sum_jr_j^{'new}
r^{old} = r^{new}
  
  
RandomWalk with restarts and Personalized PageRank

Imagine we have a bipartite graph consisting of users on one side and items on the other. We would like to ask how related two items are or how related two users are. Given that the users have already purchased in the past - what can we recommend to them, based on what they have common with other users.  

We can use different metrics to quality this such as shortest path or number of common neighbors to determine this. We have also seen that PageRank can be used to rank pages by importance. Personalized PageRank, ranks proximity of nodes to the teleport nodes S. And if we have a single node in S, then we call it random walk with restarts.

One way to implement this is to take the teleport set and compute the pagerank vector using power iteration. but its quicker to just do a simple random walk. So, the random walker starts at node A and then whenever it teleports it goes back to A. This will give us all the nodes that are most similar to A by identifying those with the highest visit counts. So, this results in a very simple recommender system that works very well in practice. 


### PageRank Summary:
Normal pagerank:
teleportation vector is uniform

personalized pagerank: teleport to a topic specific set of pages. 
nodes can have different probabilities of surfer landing there

random walk with restarts:
- topic specific pagerank where teleport is always to the same node. In this case, we dont need power iteration we can just use random walk and its very fast and easy.
  
