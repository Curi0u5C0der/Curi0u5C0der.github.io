---
layout: default
title:  Graph Datastructure
---

# Graph

Graph is a data structure which is non-linear and consists of vertices and edges. The vertices in a graph is also known as the nodes, nodes are the components that represents the objects, and the relationship between the objects are formed using the egdes which connects the vertices.

![Graph with Vertices and Edges](/images/graph.png "Graph")

For example, in a social network, people can be represented as nodes, and their friendships as edges connecting them. In a road network, intersections can be nodes, and roads can be edges connecting them.

## Types of Graphs

There are three basic types of graphs representations,
- Directed
- Undirected
- Weighted

### Directed Graph

In a directed graph, edges have a direction and represent a one-way relationship between nodes.

![Directed Graph](/images/directed.png "Directed Graph")

### Undirected Graph

In an undirected graph, edges have no direction and represent a two-way relationship between nodes.

![Undirected Graph](/images/undirected.png "Undirected Graph")

### Weighted Graph

Both in directed and undirected graphs, weights can be associated with edges, which represent the strength of the relationship between nodes. Weighted graphs are used in a variety of applications, such as in transportation networks, 
where the weight can represent the distance between two locations.

![Weighted Graph](/images/weighted.png "Weighted Graph")

## Applications of Graph

Graph is a powerful datastructure that can be used to solve a variety of problems, listed below are few useful applications of graph data strcture:

- **Social Networks:** Social Networks can be modlled using a graph database, with each user represented using vertices and the relationship between the users can be denoted using edges connecting the vertices, and graph algorithms can be used to analyze these networks, for example to identify influential users or to recommend new connections.

- **Routing algorithms:** In Routing algorithms such as transportation systems and networks, graphs are commonly used data structure, graphs can be used to solve varities of problems in complex networks such as finding the shortest path between two locations using graph algorithms such as Dijkstra's algorithm.

- **Data visualization:** Graph data structure can be used to visually represent a complex network of genes and their interactions in a biological system.

- **Game AI:** Graphs are often used in game development to represent the game world and to create AI that can navigate the world and make decisions based on the graph structure.

- **Recommendation systems:** Graphs data structrue can be used to build recommendation systems, that suggest content to users based on their preferences and the preferences of similar users.

- **Search engines:** Search engines such as Google use graphs to model the web, with each webpage represented by a node and hyperlinks between pages represented by edges. Complex graph algorithms like page rank algorithm can be used to rank pages based on their relevance to a search query.

- **Machine learning:** Graphs are used in machine learning applications such as neural networks and deep learning to represent data and relationships between data points.

Overall, the graph data structure is a powerful tool with a wide range of useful applications in many fields.