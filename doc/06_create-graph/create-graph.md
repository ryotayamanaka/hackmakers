# Create the Graph

## Introduction

Now, the tables are created and populated with data. Let's create a graph representation of them.

Estimated time: 5 minutes

### Objectives

Learn how to create a graph from relational data sources by:
- Starting a client (Python) that connects to the server
- Using PGQL Data Definition Language (DDL) (e.g. CREATE PROPERTY GRAPH) to instantiate a graph

### Prerequisites

- This lab assumes you have successfully completed the lab - Create and populate tables.

## **STEP 1:** Connect with Python client

SSH into the compute instance where you installed the graph server.

    ssh -i <private_key> opc@<public_ip_for_compute>

Then start a client shell instance that connects to the server

    opgpy --base_url https://localhost:7007 --user hackmakers
    
    enter password for user hackmakers (press Enter for no password): 

You should see the following if the client shell starts up successfully.

    Oracle Graph Server Shell 20.4.0
    >>>

## **STEP 2:** Create the graph

Set the create property graph statement.

    statement = '''
    CREATE PROPERTY GRAPH "Hackmakers"
      VERTEX TABLES (
        hackmakers.customer
          LABEL "Customer"
          PROPERTIES (cst_id, first_name, last_name)
      , hackmakers.account
          LABEL "Account"
          PROPERTIES (acc_id)
      )
      EDGE TABLES (
        hackmakers.transaction
          KEY (txn_id)
          SOURCE KEY(src_acc_id) REFERENCES account
          DESTINATION KEY(dst_acc_id) REFERENCES account
          LABEL "transfered_to"
      , hackmakers.account
          KEY (acc_id)
          SOURCE KEY(cst_id) REFERENCES customer
          DESTINATION KEY(acc_id) REFERENCES account
          LABEL "owns"
      )
    '''

Run the statement.

    session.prepare_pgql(statement).execute()

If you need to recreate the graph, destroy the graph first and run the statement above again.

    graph.destroy()

## **STEP 3:** Check the newly created graph

Attach the graph to check that the graph was created. 

    graph = session.get_graph("Hackmakers")
    graph
    
    PgxGraph(name: Hackmakers, v: 180, e: 835, directed: True, memory(Mb): 0)

Run the query below, which find all transactions from the accounts owned by 'David'.

    graph.query_pgql("""
        SELECT c1.first_name AS src_first_name, c1.last_name AS src_last_name, t.amount, t.datetime, 
               c2.first_name AS dst_first_name, c2.last_name AS dst_last_name
        FROM MATCH (c1)-[:owns]->(a1)-[t]->(a2)<-[:owns]-(c2)
        WHERE c1.first_name = 'David'
        ORDER BY src_last_name, t.datetime
    """).print()

You may now proceed to the next lab.
