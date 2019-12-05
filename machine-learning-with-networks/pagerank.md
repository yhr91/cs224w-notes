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

$$\rm ENC$$ maps node $$u$$ and $$v$$ to low-dimensional vector $$\mathbf{z_u}$$ and $$\mathbf{z_v}$$:
![node embeddings](../assets/img/node_embeddings.png?style=centerme)

$$
\mathcal{L} =\sum_{u\in V} \sum_{v\in N_R (u)} -\rm \ log (P(v|\mathbf{z_u}))
$$

![node2vec](../assets/img/node2vec.png?style=centerme)
 
