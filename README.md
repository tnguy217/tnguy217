
- 🌱 I’m currently learning on this Computer Scence, this is my last project and I really need help to find where I did wrong
This project is the essential part of a navigation system. It reads in a directed graph or an undirected
graph G with n vertices and m edges from a file specified by the command-line (with proper
arguments). It then takes corresponding actions for given instructions from stdin. Besides the
Stop instruction, valid instructions include
α: graph printing instruction (PrintADJ)
β: path computation instructions (SinglePair and SingleSource)
γ: length/path printing instructions (PrintLength and PrintPath)
Both SinglePair and SingleSource path computations should have worst-case time complexity
O(m log n). Path printing should have worst-case time complexity O(n). Length printing should
have worst-case time complexity O(1). Memory should be allocated when needed, and released
when it is no longer needed. Memory leaks should be avoided.
2 Modular Design
For modular design, you need to write the following modules:
• a module for various data structures, with header file named data structures.h
• a module for min-heap, with header file named heap.h and implementation file named
heap.cpp
• a module for stack, with header file named stack.h and implementation file named stack.cpp
• a module for graph algorithms, with header file named graph.h and implementation file
named graph.cpp
• a module for utilities, with header file named util.h and implementation file named util.cpp
• and the main program, with header file named main.h and implementation file named
main.cpp
1
Your project should only use basic C++ libraries to compile your implementation to produce the
executable PJ3. To ensure that,the following standard Makefile will be used to grade your project.
EXEC = PJ3
CC = g++
CFLAGS = -c -Wall
$(EXEC) :main.o util.o stack.o heap.o graph.o
$(CC) -o $(EXEC) main.o util.o heap.o graph.o stack.o
main.o :main.cpp main.h data_structures.h util.h stack.h heap.h graph.h
$(CC) $(CFLAGS) main.cpp
util.o :util.cpp util.h data_structures.h
$(CC) $(CFLAGS) util.cpp
stack.o :stack.cpp stack.h data_structures.h
$(CC) $(CFLAGS) stack.cpp
heap.o :heap.cpp heap.h data_structures.h
$(CC) $(CFLAGS) heap.cpp
graph.o :graph.cpp graph.h data_structures.h stack.h heap.h
$(CC) $(CFLAGS) graph.cpp
clean :
rm *.o $(EXEC)
Given that your project will be compile using the above Makefile, you have to name the files exactly
as specified.
3 Data Structures
This section provides some data structures that may give you a head start with the project. You
SHOULD NOT copy and paste these data structures. However, you can write similar ones after
studying them. It should be your own work.
3.1 VERTEX
You need to have a data structure for vertices in the graph. Our algorithms also use a color system
on vertices. The following is a part of the codes that I wrote. Note that the definition of COLOR in
the following is NOT provided.
typedef struct TAG_VERTEX{
int index;
2
COLOR color;
double key;
int pi;
int position;
}VERTEX;
typedef VERTEX *pVERTEX;
Here the field position is used to record the position of the vertex in the min-heap array.
You will find this very handy when DecreaseKey operations are needed. I use the key field (rather
than the d field) to reuse my codes for the min-heap.
Here is a sample usage of the data type VERTEX.
pVERTEX *V;
// V is a pointer to pointer to VERTEX
V = (pVERTEX *) calloc(n+1, sizeof(pVERTEX));
// Now you can make reference to V[1], but not to V[1]->index
for (int i=1; i<=n; i++){
V[i] = (VERTEX *) calloc(1, sizeof(VERTEX));
// Now you can make reference to V[1]->index
}
Please note that illegal memory access is the main cause of segmentation fault.
3.2 EDGE
You need to have a data structure for nodes on the adjacency lists of the graph. I wrote the following
in my implementation.
typedef struct TAG_NODE{
int index;
int u;
int v;
double w;
TAG_NODE *next;
}NODE;
typedef NODE *pNODE;
The usage of the NODE data type is similar to that of VERTEX.
3.3 MIN-HEAP
In Project 2, you have implemented the min-heap data structure, where the HEAP data structure
contains the fileds capacity (of type int), size (of type int), and H (of type ELEMENT **). The
data structure ELEMENT contains only one required field: key (of type double).
To partially re-use your implementation of Project 2, you can use the following:
3
typedef VERTEX ELEMENT;
typedef ELEMENT *pELEMENT;
Because VERTEX has a filed named key (of type double), your earlier implementations of min-heap
should work as usual. Note that you can make reference to the four other fields defined in VERTEX,
and in ELEMENT.
In this project, the elements of the heap array should be pointers to objects of type VERTEX. To
reuse codes with minimal modification, I used the following in my implementation.
typedef VERTEX ELEMENT;
typedef ELEMENT *pELEMENT;
typedef struct TAG_HEAP{
int capacity;
int size;
pELEMENT *H;
}HEAP;
typedef HEAP *pHEAP;
3.4 STACK
In order to print out the computed path from a source node s to a destination node t, we start from
the destination node to trace out the path using the predecessor field. This will trace out the path
in reverse order. In order to print the path in correct order, we need to use a stack. The elements of
the stack should contain information (or pointers to information) about the corresponding vertices
on the path.
4 Valid Executions
A valid execution of your project has the following form:
./PJ3 <InputFile> <GraphType> <Flag>
where
• PJ3 is the executable file of your project,
• <InputFile> should be the exact name of the input file,
• <GraphType> should be substituted by either DirectedGraph or UndirectedGraph,
• <Flag> is either 1 or 2.
Your program should check whether the execution is valid. If the execution is not valid, your
program should print out the following message to stderr and stop.
Usage: ./PJ3 <InputFile> <GraphType> <Flag>
Note that your program should not crash when the execution is not valid.
4
5 Flow of the Project
5.1 Read in the Graph and Build the Adjacency Lists
Upon a valid execution, your program should open the input file (specified by argv[1]) and read
in the graph. The format of the input file is the following. The first line contains two positive
integers, representing the number of vertices n and number of edges m, respectively. Each of the
next m lines has the following format:
edgeIndex u v w(u, v)
where u is the start vertex of edge (u, v), v is the end vertex of edge (u, v), w(u, v) is the weight
of edge (u, v), and edgeIndex is the index of edge (u, v). Your program should read in edgeIndex,
u, and v as int, and read in w(u, v) as double. For undirected graphs, each edge still has two
vertices but the order is not important.
After knowing the values of n and m, your program should dynamically allocate memory for an
array of n pointers to objects of type VERTEX. Let us call this array V. Then for each i = 1, 2, . . . , n,
V[i] is a pointer to an object for vertex i. Your program should dynamically allocate memory for
an array of n adjacency lists. Let us call this array ADJ. Then for each i = 1, 2, . . . , n, ADJ[i] is a
pointer to the adjacency list of vertex i.
While reading the graph from argv[1] edge by edge, the adjacency lists are populated accordingly.
Note that dynamic memory allocation is needed for each node (corresponding to an edge) on the
adjacency lists.
Let (u, v) be the edge newly read in from the file argv[1]. If the graph is directed and flag is 1,
insert the corresponding node for edge (u, v) at the front of ADJ[u]. If the graph is directed and flag
is 2, insert the corresponding node for edge (u, v) at the rear of ADJ[u]. If the graph is undirected
and flag is 1, insert a corresponding node for edge (u, v) at the front of ADJ[u], and insert a
corresponding node for edge (v, u) at the front of ADJ[v]. If the graph is undirected and flag is 2,
insert a corresponding node for edge (u, v) at the rear of ADJ[u], and insert a corresponding node
for edge (v, u) at the rear of ADJ[v]. After all m edges are read in, your program should close the
file argv[1].
5.2 Initialize the Priority Queue (min-heap) and the Stack
Your program should initialize a min-heap of capacity n and a stack of capacity n. Dynamic memory
allocations are required for the min-heap and the stack. The heap will be used in the execution
of Dijkstra’s path finding algorithm. The stack will be used to output a path from a given source
vertex to a given destination vertex.
5.3 Loop over the Instructions
Your program should expect the following instructions from stdin and act accordingly:
5
(a) Stop
On reading Stop, the program stops.
(b) PrintADJ
On reading the PrintADJ instruction, your program should do the following:
(b-i) Print the adjacency lists of the input graph to stdout. Refer to posted test cases for
output format.
(b-ii) Wait for the next instruction from stdin.
(c) SinglePair <source> <destination>
where <source> and <destination> are two integers in the set {1, 2, . . . , n}. This is one
of two path computation instructions. On reading the SinglePair instruction, your
program should do the following:
(c-i) Apply the variant of Dijkstra’s algorithm (as taught in class and stated in the lecture
slides) to compute a shortest path from <source> to <destination>. In particular, for
each i ∈ {1, 2, . . . , n}, the algorithm should compute the final values of V[i]->key and
V[i]->π. If <destination> is reachable from <source>, V[destination]->key is the
weight of a shortest path from <source> to <destination>, and V[destination]->π
is the predecessor of <destination> on the computed shortest path from <source> to
<destination>. If <destination> is not reachable from <source>, V[i]->π is nil,
and V[i]->key is DBL MAX (defined in the header file <cfloat>).
(c-ii) Wait for the next instruction from stdin.
(d) SingleSource <source>
where <source> is an integer in the set {1, 2, . . . , n}. This is the other path computa-
tion instruction. On reading the SingleSource instruction, your program should do the
following:
(d-i) Apply Dijkstra’s algorithm to compute shortest paths from <source> to all vertices that
are reachable from <source>. In particular, for each i ∈ {1, 2, . . . , n}, the algorithm
should compute the final values of V[i]->key and V[i]->π. If vertex i is reachable from
<source>, V[i]->key is the length of a shortest path from <source> to i, and V[i]->π
is the predecessor of i on the computed shortest path from <source> to i. If vertex i is
not reachable from <source>, V[i]->π is nil, and V[i]->key is DBL MAX.
(d-ii) Wait for the next instruction from stdin.
(e) PrintLength <s> <t>
where <s> and <t> are two integers in the set {1, 2, . . . , n}. This is the only length print-
ing instruction. This instruction is valid if and only if <s> is the same as <source> in the
most recent path computation instruction, and <t> is the same as <destination> in case the
most recent path computation instruction is a SinglePair instruction.
On reading a valid PrintLength instruction, your program should do the following:
6
(e-i) If your program has computed a shortest <s> to <t> path in the most recent path com-
putation, print the length of the computed path to stdout. Refer to posted test cases
for output format.
If your program has not computed a shortest <s> to <t> path in the most recent path
computation, print the following to stdout:
There is no path from <s> to <t>.
In the above, <s> and <t> should be replaced by their given values. Refer to posted test
cases for output format.
(e-ii) Wait for the next instruction from stdin.
(f) PrintPath <s> <t>
where <s> and <t> are two integers in the set {1, 2, . . . , n}. This is the only path printing
instruction. This instruction is valid if and only if <s> is the same as <source> in the most
recent path computation instruction, and <t> is the same as <destination> in case the most
recent path computation instruction is a SinglePair instruction.
On reading a valid PrintPath instruction, your program should do the following:
(f-i) If your program has computed a shortest <s> to <t> path in the most recent path compu-
tation, print the computed path to stdout. Refer to posted test cases for output format.
If your program has not computed a shortest <s> to <t> path in the most recent path
computation, print the following to stdout:
There is no path from <s> to <t>.
In the above, <s> and <t> should be replaced by their given values. Refer to posted test
cases for output format.
(f-ii) Wait for the next instruction from stdin.
(g) Invalid instruction
On reading an invalid instruction, your program should do the following:
(g-i) Write the following to stderr:
Invalid instruction.
(g-ii) Wait for the next instruction from stdin
<!---
tnguy217/tnguy217 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
