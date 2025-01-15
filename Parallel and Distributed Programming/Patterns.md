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

------------------------------------------------------------------

**Heap and Stack Allocation:** Memory allocations are important.

------------------------------------------------------------------

**Closure and Objects:**
Closures are function objects that can be constructed and managed like data. Lambda functions.
Objects are a language construct that associate data with the code to act on and manage that data.


## Parallel Data Management Patterns
**Pack:** Eliminates unused or irrelevant elements in a collection. Input elements marked with true and false.

Application: Image processing, Collision detection.

------------------------------------------------------------------

**Pipeline:** Tasks are sectioned into separate stages and each stage of the process can be done at the same time by different functions. It follows the producer-consumer relationship. One stage's output is another's input. Each stage processes data independently and simultaneously. Like a car factory works.

Application: Video Encoding and Decoding, Real-Time Data Processing

------------------------------------------------------------------

**Geometric Decomposition:** Divide data into overlapping or non overlapping sub-collections.

Application: Decomposition for stencil computations.

------------------------------------------------------------------

**Partition Pattern:** Special case for Geometric Decomposition which only have non overlapping partitions. Common in matrix operations and image processing.

------------------------------------------------------------------

**Gather:** Collects elements according to an input array. Gathering the one required. I think this is almost like pack but with a predetermined input array.

Applications: Sparse matrix operations, ray tracing.

**Scatter:** Instead of selecting some part of the collection from an array it just re-arranges according to the input array.

Application: Writing results of simulations, Updating sparse matrices.

## Advanced Patterns
**Super scalar Sequence:** Sequences can happen in parallel as long as data dependencies are met.

**Futures:** Non-hierarchical task management. First task is initiated then a future object is returned immediately. The other processes will continue then when the actual value is needed the real value will be replaced in place of the placeholder.

**Speculative Selection:** Executes all branches of conditional statement regardless of the necessity in parallel then discards the ones that are not necessary. 

**Workpile:** A generalization of Map pattern where Tasks can be added while execution. Tasks will be dealt with and after that tasks may create another task and that task is going to be included in the stack.

Challenges are efficient task scheduling.
Avoiding bottlenecks in task generation.

**Search:** find elements in a collection that match a certain criteria. Criteria could range from simple character match to complex arithmetic constraints.

**Segmentation:** Divides a collection into multiple segments for independent processing. Each segment can be treated as a separate collection.
`[1, 2, 3, 4, 5, 6]`
`[1-2], [3-4], [5-6]`
`[3, 7, 5]` (Sum of each segment)

Application: Multi-thread data processing, Data-Parallel Algorithms in Distributed Systems.

**Expand:** Creates new data elements or collections based on an existing one, number of output is not known, it depends on input. Like Generative AI. Generating neighboring states in simulations.

Problems could be managing memory for newly generated data.

**Category Reduction:** Groups data into categories and applies a reduction operation within each category. 

first grouping then applying reduction in the categories. So segmented reduction malet new.


## Programming Model Support For Patterns
All 3 of them support 3 models: Map pattern, Workpile and Reduction.


## The Map Pattern Implementation
#### Key characteristics of the Map pattern.
1. **Independent tasks:** Each element is isolated.
2. **No side effects:** The function applied should not alter any shared state.
3. **Flexible Scheduling:** Can be implemented in various programming frameworks.

##### Serial Execution (Python)
```
def saxpy(a, x, y):
	for i in range(len(x)):
		y[i] = a * x[i] + y[i]
```

##### Parallel With TBB (C++)
```
#include <tbb/parallel_for.h>
#include <vector>

void saxpy_tbb(float a, std::vector<fload>& x, std::vector<float>& y) {
	tbb::parallel_for(size_t(0), x.size(), [&](size_t i) {
		y[i] = a * x[i] + y[i]
	})
}
```

##### Parallel With OpenMP (C/C++)
```
#include <omp.h>
#include <stdio.h>

void saxpy_openmp(float a, float* x, float* y, int n) {
	#pragma omp parallel for
	for (int = 0; i < n; i++) {
		y[i] = a * x[i] + y[i]
	}
}
```

##### Parallel With OpenCl (Open Kernel Code)
```
__kernel void saxpy_kernel(float a, __global const float* x, __global float* y) {
	int i = get_global_id(0);
	y[i] = a * x[i] + y[i]
}
```

