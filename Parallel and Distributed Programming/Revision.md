What are patterns? Patterns are codification of best practices in software engineering. Task parallelism and data access combinations that solve a specific problem in algorithm design.

Task parallelism and data access combination to solve a specific problem in algorithm design. 
In Software Engineering it is the codification of best practices.

Why do we use it?
Universal: Can be used regardless of hardware and language
Simplicity: It is clear and structured for designing algorithms
Determinism: Outputs can be replicated for efficient debugging and testing


Nesting patterns: Enables hierarchical and recursive composition of patterns. Nesting levels are not allowed in most programming models nowadays. They are directly mapped to hardware like CUDA, OpenMP, OpenCL, C++ but in TBB and Cilk plus it is allowed (arbitrary nesting).

**Serial Control Flow Patterns**
Sequence, Selection, Iteration, Recursion

Sequence: Enforces serial order of execution where where an element is dependent on the one before it.

A = f(T)
B = g(A)
C = h(B)

It can also be parallelized in some cases

A = f(T)
B = g(T)
C = h(B, A)

Selection: Is choosing which path the execution is going to go according to conditions. ```if(a) {b;} else {c;}```

Iteration: Is looping over an action until a condition is met not to loop anymore. 
```while(c) a;```

Recursion: This is when a function calls itself. Whether it is directly or indirectly enabling nested tasks. If the function call happens at the end it crates a tail recursion which is a special form of recursion that can be converted into iteration.

**Parallel Control Flow Patterns**
Fork-join, Map, Stencil, Reduction, Scan, Recurrence

fork-join: It is separating elements into different parts then combining them after processing simultaneously. E.g: Quick Sort and Merge Sort

Map Pattern: A single function will be implemented for each element separately. Each process is independent of the others. E.g: Image processing, Gamma correction, Color Space Conversion, ray tracing

Stencil Pattern: It is like Map pattern but a single function has access to the neighboring element too. It is useful in cases where the context of the former element is needed. E.g: Image filtering (convolution), Motion estimation in video encoding, fluid flow simulation.

Reduction: Works by merging all the elements together as a single value. Uses multiplication or addition (associative combiner). E.g: Summing elements of a collection, Dot product of matrix multiplication, used in iterative solutions in linear equations.

Scan: Works by having an array of all the reduction of the elements before for each element. E.g: Integration, Financial Modeling, Random Number Generation.

**Serial Data Management Patterns**
Random Read and Write, Stack and Heap Allocation, Closure and Objects

Random Read and Write: It is reading from a specific part of memory and the problem could be Aliasing

Stack and Heap Allocation: A data structure to store data in a LIFO structure and dynamic memory allocation for heap

Closure and Object: Closure is a function of an object that has procedures in it. and Object is a combination of functions and data, properties and methods.

**Parallel Data Management Patterns**
Pack, Pipeline, Geometric Decomposition, Partition, Gather, Scatter

**Pack**: Eliminates unused or unnecessary elements form a collection
**Pipeline**: The process is separated into sections where each section does a single process of the whole process. Like a car factory would do manufacturing cars.
**Geometric Decomposition**: Overlapping or non-overlapping sections to store data
**Partition**: This is a special type off Geometric Decomposition in which only Non-overlapping sections are created. This is used for stencil control flows.
**Gather**: Selects some elements from a collection according to an input array.
**Scatter**: Rearranges elements in a collection according to an input array. Hence scattering.