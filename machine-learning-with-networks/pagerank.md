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

What did they find:
-	The figure below sorts the starting nodes by the number of nodes that BFS visits when starting from that node
-	If you look at the nodes on the left colored blue, they are able to reach a very small number of nodes before stopping
-	If you look at the nodes on the left colored blue, they are able to reach a very large number of nodes before stopping
-	Using these plots we can determine the number of nodes in the IN and OUT component of the bowtie shaped web graph shown below. 
- SCC corresponds to the largest strongly connected component in the graph.
- The IN component corresponds to nodes which have out-links to SCC but no in-links from SCC. The OUT component corresponds to nodes which have in-links from SCC but no out-links to SCC. 
- Nodes with both in and out links to the SCC fall in SCC.
- Tendrils correspond to edges going out from IN or those going into OUT. TUBES are connections from IN to OUT that bypass SCC.
-	Disconnected components are not connected to SCC at all
