---
title: About Shell 2
date: 2016-07-29
tags:
- shell
---

The former article about shell is from the point of view of multi-tasking.  This one we expand from the view of resource abstraction --- files.

<!-- more -->

## Abstraction

In the hardware level the addressing of an external resource differs from their interfaces.  The result of the first abtraction attemptation is the unified addressing space.  The accessing of external resources now is interpreted as reading/writing from/to an address within a linear space.

Apparently a meaningless number is a bad name for human.  We will want a naming system that mark each piece of resource with memorable text.  Furthermore, the organizing requirement demanding the introduction of the "belongs to" relationship.  The combination of the two is the common general purpose file system implementation, a hierarchical name-tagging tree.

By convention the reference of any node in the tree can be represented by the path from root to itself.  Normally we pick a single character as the separator, say "%".  Then the path "root%one%leaf" will represent a node belongs to node "one" which belongs to root node.  The resource representatives are called "files" and the topology representatives are called "directories".  The meaning of "everything is a file" is indeed the reference of the addressing unification.

## Implementation

A simplified tree node data structure would look like this: it should contains the underlying linear address of the resource if it's a file, and pointers to its children if it's a directory.  It may also has a unique identifier within the system.

```c
typedef struct NodeT {
  char *name;
  int id;
  int address;
  NodeT *children;
} Node;
```

Here's an example of a file:

```c
Node file_node = Node { .name = "haha", .id = 23333, .address = 0x7869cd12, .children = null }
```

An example for a directory:

```c
Node dir_node = Node { .name = "lala", .id = 13684, .address = -1, .children = [23333, 15682, 33497] }
```

Nodes are traced via their ancestors and all the tracings are started from the root node. If we no longer want a node, the easiest way is to make it untraceable. Hence, deleting is basically sematically equivalent to unlinking.

Furthermore, the structure of file system is not necessarily be constrained to a tree. Multiple directories can share one child as long as they hold a child record with the same id number. This way we created hard linked files. Every child record of that id number is a hard link to others.

The structure is not constrained to a DAG, too. We can easily create a cycle in the file system. The only problem is they're meaningless.
