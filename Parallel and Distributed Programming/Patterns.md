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

Some languages support small amount of nesting and sometimes it is even directly mapped to hardware.

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


## **Parallel Control Flow Patterns**
**fork-Join:** Splits the workflow into parallel and later combines it into one.
- Used in OpenMP and Cilk plus
- Divide and Conquer (Quick Sort and Merge Sort)
	- Quick Sort: Divide the array into parts then sort it and join them together
	- Merge Sort: Recursively separate the array, sort the halves and merge them in parallel.
- Graph algorithms, Dynamic programming

------------------------------------------------------------------

**Map Pattern:** Applies functions on each component independently. There is no connection between tasks. Embarrassingly parallel.
- Ideal for SIMD architectures.
- Applications: Image Processing (Color Space Conversion), Monte Carlo Sampling, Ray Tracing, image filtering.

------------------------------------------------------------------

**Stencil Pattern:** Generalization of map pattern with neighborhood access. A single function can access it's corresponding task and it's neighboring task. Makes it useful in places where the context of a data point is useful.

Difference between map and stencil:
- Map processes each task (element) separately.
- Stencil considers neighbors during operation

Example: Convolution, Median filtering, Motion estimation in video encoding, Isotropic Diffusion

------------------------------------------------------------------

**Reduction:** Combines all of the elements of a collection into a single value.
Used in, Numerical Applications, Dot product in matrix multiplication, 

combines everything like a + b + c + d + e

------------------------------------------------------------------

**Scan:** Computes all partial reductions of a collection. Produces an array where each element is a reduction of all the previous elements.

has all the elements before it as a reduction (with addition or multiplication)
[a + (a+b), (a+b+c), (a+b+c+d)]
[1 + 3 + 6 + 10 + 15 + 21 + 28]

Application: Integration, Financial Modeling, Random Number Generation, Data Packing

------------------------------------------------------------------

**Recurrence:** Generalization of iteration where loop iterations depend on previous computations. There must be serial ordering for computation.

Applications: Matrix Factorization, Image Processing, PDE Solvers (partial differential equations), Sequence Alignment.


## Serial Data Management Patterns
**Random Read and Write**: Uses pointers or indices for direct memory access. it involves reading from or writing to specific parts of memory location. Direct access to memory.

Challenges could be Aliasing (Which mean two pointers accessing the same memory location and it could cause synchronization problems if they do it at the same time.) Two or more memory locations referring to the same data. (Race Condition)

**Heap and Stack Allocation:** Memory allocations are important.
**Closure and Objects:**
Closures are function objects that can be constructed and managed like data. Lambda functions.