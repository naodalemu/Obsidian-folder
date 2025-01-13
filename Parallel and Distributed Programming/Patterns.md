### **What are patterns?**
Patterns codify best practices in software engineering. In parallel programming, they help design algorithms by addressing task distribution and data access.
- Codifying best practice in SE
- task parallelization and data access standardization in parallel programming.

### **Why Do we use patterns**
- **Universality**: Independent of hardware or software(language)
- **Simplicity**: Provide a clear instruction for designing algorithms
- **Determinism**: Ensure repeatable results for easier testing and debugging because the problem of parallelism is determinism.


## **Nesting Patterns**
### **What are nesting patterns**
Nesting allows combining patterns hierarchically and recursively.
Nesting involves hierarchical and recursive composition of other patterns in parallel programming. It helps developers to build complex algorithms from simpler components.

Some languages support small amount of nesting and sometimes it is even directly mapped to hardwar.

TBB and Cilk allow flexible nesting and while efficiently mapping parallelism to physical hardware.
Systems that do not support nesting are CUDA, OpenCL, C++, AMP, OpenMP
CUDA,
OpenCL,
OpenMP,
C++,

## **Structured Serial Control Flow Patterns**
### Sequence
A sequence enforces the order of task execution. One thing is dependent on the result of the previous task. Tasks are executed in a strict linear order.
T = f(A)
S = g(T)
B = h(S)

Let's try another one that can use parallel execution at some places.
T = f(A)
S = g(A)
B = h(S, T)
### Selection
Chooses one of the two branches, only one branch gets executed ensuring mutual exclusivity. It is a control structure. Serial execution evaluates the condition and then chooses one of the branches.
```
if (c) {a;}
else {b;}
```

### Iteration
Iteration repeats a task while a condition holds true. Loops are a common implementation for this pattern. for and while loops are used for this.

### Recursion
Recursion occurs when a function calls itself directly or indirectly. 
Tail Recursion is a special kind of recursion where the function call is at the end of the function so it can be considered as iteration.
Examples:
- Factorial Calculation
- Fibonacci Sequence
- Divide and Conquer Algorithms (Quick sort and Merge sort)


## **Parallel Control Patterns**
**fork-Join:** Splits the workflow into parallel and later combines it into one.
- Used in OpenMP and Cilk plus
- Divide and Conqure (Quick Sort and Merge Sort)
	- Quick Sort: Divide the array into parts then sort it and join them together
	- Merge Sort: Recursively separate the array, sort the halves and merge them in parallel.
- Graph algorithms, Dynamic programming

**Map Pattern:** 
**Stencil Pattern:** 
**Reduction:** 
**Scan:** 
**Recurrence:** 
