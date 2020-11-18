# Graph Query and Analysis in Python

## Introduction

This example shows how integrating multiple datasets and using a graph facilitate additional analytics and can lead to new insights. We will use three small datasets for illustrative purposes. The first contains accounts and account owners. The second is purchases by the people who own those accounts. The third is transactions between these accounts.

The combined dataset is then used to perform the following common graph query and analyses: pattern matching, detection of cycles, finding important nodes, community detection.

Estimated Lab Time: 10 minutes

### Objectives

- Learn how to query and analyze graphs with the Graph Server and Client.

### Prerequisites

- This lab assumes you have successfully completed all the previous Labs and have the Python client up and running.

## **STEP 1:** Get Graph

Attach the graph which is already loaded into the Graph Server.

    graph = session.get_graph("Hackmakers")
    graph

## **STEP 2:** Pattern Matching

PGQL Query is convenient for detecting specific patterns.

Find accounts that had an inbound and an outbound transfer, of over 500, on the same day. The PGQL query for this is:

    graph.query_pgql("""
      SELECT a.account_no
           , a.balance
           , t1.amount AS t1_amount
           , t2.amount AS t2_amount
           , t1.date
      FROM MATCH (a) <-[t1:transfer]- (a1)
         , MATCH (a) -[t2:transfer]-> (a2)
      WHERE t1.date = t2.date
        AND t1.amount > 500
        AND t2.amount > 500
        AND a.balance < 300
    """).print()

## **STEP 3:** Detection of Cycles

Next we use PGQL to find a series of transfers that start and end at the same account, such as A to B to A, or A to B to C to A.

The first query could be expressed as:

    graph.query_pgql("""
    """).print()

The second query just adds one more transfer to the pattern (list) and could be expressed as:

    graph.query_pgql("""
    """).print()

## **STEP 4:** Influential Accounts

Filter customers from the graph. (cf. [Filter Expressions](https://docs.oracle.com/cd/E56133_01/latest/prog-guides/filter.html))

    graph2 = graph.filter(pgx.EdgeFilter("edge.label()='transfer'"))
    graph2

Run [PageRank Algorithm](https://docs.oracle.com/cd/E56133_01/latest/reference/analytics/algorithms/pagerank.html). PageRank Algorithm assigns a numeric weight to each vertex, measuring its relative importance within the graph.

    result = analyst.pagerank(graph2);
    result

Show the result.

    graph2.query_pgql("""
      SELECT a.acc_id, a.pagerank
      MATCH (a)
      ORDER BY a.pagerank DESC
    """).print()

## **STEP 5:** Community Detection

Let's find which subsets of accounts form communities. That is, there are more transfers among accounts in the same subset than there are between those and accounts in another subset. We'll use the built-in weakly / strongly connected components algorithm.

The first step is to create a subgraph that only has the accounts and the transfers among them. This is done by creating and applying an edge filter (for edges with the lable "TRANSFER') to the graph.

    graph2 = graph.filter(pgx.EdgeFilter("edge.label()='transfer'"))
    graph2

[Weakly Connected Component](https://docs.oracle.com/cd/E56133_01/latest/reference/analytics/algorithms/wcc.html) (WCC) algorithm detects only one partition.

    graph2.destroy_vertex_property_if_exists("wcc")
    analyst.wcc(graph2)

The partition value is stored in a property named `wcc`

    graph2.query_pgql("""
      SELECT a.wcc AS component_id, COUNT(*) AS count
      FROM MATCH (a)
      GROUP BY a.wcc
      ORDER BY a.wcc
    """).print()

In this case, all six accounts form one partition by the WCC algorithm.

Run a strongly connected components algorithm, SCC Kosaraju, instead.

[Strongly Connected Component](https://docs.oracle.com/cd/E56133_01/latest/reference//analytics/algorithms/scc.html) (SCC) algorithm detects three partitions.

    graph2.destroy_vertex_property_if_exists("scc_kosaraju")
    analyst.scc_kosaraju(graph2)

List partitions and number of vertices in each

    graph2.query_pgql("""
      SELECT a.scc_kosaraju AS component_id, COUNT(a.account_no) AS count
      FROM MATCH (a)
      GROUP BY a.scc_kosaraju
      ORDER BY a.scc_kosaraju
    """).print()

## Acknowledgements ##

* **Original Author** -  Jayant Sharma, Product Manager, Spatial and Graph
* **Contributors** - Arabella Yao, Product Manager Intern, Database Management, and Jenny Tsai.
