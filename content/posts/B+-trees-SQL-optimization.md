+++
title = 'How B+ Trees Optimize SQL Queries: A Primer'
date = 2024-06-11T12:54:54+05:30
draft = false
author = 'durwasa'
tags = ['programming']
+++


## Introduction

For someone who has taken a course in Computer Science, they have probably come across a B+ tree, often used in the context of databases for storing data. A B+ tree schematically looks like this:

```text
                   [   1003   ]
                  /     |     \
          [1001]  [1002]  [1004 1005]  [1007]
         /       |             |           |
    [Naruto] [Sasuke] [Sakura Hinata Kakashi] [Itachi]
```

In a B+ tree, the data always lies in the leaf nodes.

However, what are the advantages of using such a structure in a database? Considering that data always resides on disk, how are disk operations optimized? Where does the OS come into play in optimizing queries? Ultimately, running a SQL application involves executing a binary like say `./sql`, which runs in RAM. So, how does this binary interact with the disk, seek through it, return to RAM, perform calculations, and fetch more data from the disk? These questions prompted me to explore deeply what happens when you execute:

```sql
SELECT * FROM table WHERE id='122';
```

## Setup

Let's consider a database with the following structure:

```text
Student ID | Name     | Marks
------------------------------
   1001    | Naruto   | 85
   1002    | Sasuke   | 72
   1003    | Sakura   | 90
   1004    | Hinata   | 78
   1005    | Kakashi  | 88
   1006    | Shikamaru| 65
   1007    | Itachi   | 92
   1008    | Gaara    | 81
   1009    | Rock Lee | 70
   1010    | Neji     | 87
```

In the disk, the data is stored in a B+ tree as follows:

```ascii
                   [   1003   ]
                  /     |     \
          [1001]  [1002]  [1004 1005]  [1007]
         /   |   \           |           |
    [Naruto] [Sasuke] [Sakura Hinata Kakashi] [Itachi]
```

When creating the table, we inform the query engine which column is our indexed key. The index key is used to navigate through the B+ tree. Remember, the values are always in the leaf nodes, so we must traverse through the B+ tree.

## Traversal

So, what happens when the query is executed? The OS needs to know where to locate the root of the B+ tree. Usually, this information is kept in a System Catalog (Metadata). This catalog includes information about tables, columns, data types, constraints, and indexes. In our minimal example, the table type and the indexed `student_id` are stored in the catalog. The system catalog includes pointers or references to the index files.

Once the root node is located on the disk, it is loaded into RAM, and the execution starts for the engine.

```text
Root Node (in RAM):
Keys: [1004]
Pointers: [pointer_to_child_1, pointer_to_child_2]

Child Nodes (on disk):
Node 1 (1001, 1002, 1003)
Node 2 (1005, 1006, 1007)
```

Here's the refined section with added details, and data on why disk seek is more costly than RAM calculations.

## Sequence Diagram

When the RAM loads the B+ tree node, it decides which child node to access next, requiring a disk operation. This seems inefficient, doesn't it? Each decision necessitates a trip to the disk to fetch the next node, which is time-consuming. Any Computer Science graduate knows that RAM operations are much faster than disk operations. Even if one is not a CS graduate, it is intuitive—disk operations involve mechanical components moving the head to the correct memory address, introducing inertia.

Clearly, this approach is superior to a linear search where each Student ID is loaded into RAM, compared, discarded, and then the next block is loaded and searched. A linear search would involve one load to RAM and one comparison per node. Instead, the system brings the top `k` nodes of the hierarchy into RAM, leaving the leaf nodes on the disk. Instead of searching the entire leaf node, the system filters decisions at the RAM level using pointers to determine which subtree may contain the desired index.

For instance, in a large tree with 20+ levels:

```text
Level 1 (RAM): [Root Node]
Level 2 (RAM): [Internal Nodes]
Level 3 (RAM): [Internal Nodes]
Level 4-20 (Disk): [Internal Nodes] ... [Leaf Nodes]
```

This significantly reduces the time required for the search. Additional techniques like cache eviction and LRU (Least Recently Used) strategies can further optimize performance. However, the overarching idea remains the same.

### Fetching from RAM to Disk

Below is a representation of how data is fetched from RAM to disk:

```text
+------------------------------------+
|            RAM                     |
+------------------------------------+
| Root Node [1003]                   |
| Internal Node 1 [1001, 1002]       |
| Internal Node 2 [1004, 1005]       |
+------------------------------------+
          |             |
          v             v
+----------------+ +-----------------+
|      Disk      | |      Disk       |
+----------------+ +-----------------+
| Leaf Nodes     | | Leaf Nodes      |
| [Naruto, Sasuke] | [Sakura, Hinata]|
+----------------+ +-----------------+
```

### What if the column is not INDEXED

When a column is not indexed, the database must perform a linear search to locate the desired data. This involves scanning each row in the table sequentially, which is highly inefficient, especially for large datasets. In contrast, an indexed column allows the database to quickly locate data using a structured B+ tree, significantly reducing search time. The `WHERE` clause on an indexed column leverages the B+ tree's hierarchy to rapidly navigate to the relevant data, minimizing disk access and computational overhead. Therefore, queries on indexed columns execute much faster than those on non-indexed columns, highlighting the critical role of indexing in database performance optimization.

### Conclusion 
B+ trees are essential for optimizing SQL query performance through efficient data organization and indexing. By leveraging the speed of RAM for intermediate node processing and minimizing disk access to leaf nodes, B+ trees significantly enhance query execution times. This hierarchical approach reduces costly disk seeks, capitalizing on the rapid access speeds of RAM for decision-making. Techniques like cache eviction and LRU further improve efficiency by keeping frequently accessed nodes readily available. This deep dive into B+ tree mechanics underscores the importance of indexing in modern databases, demonstrating how foundational computer science concepts can solve practical problems, ensuring databases remain robust, efficient, and responsive.
