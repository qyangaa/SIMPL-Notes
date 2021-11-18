# Higher Order Functions

### Re-Thinking Functions

- A very simple C function:
  <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110080324228.png" alt="image-20211110080324228" style="zoom:50%;" />
- What can we do with this function?
  - Invoke it: $x=f(x+y, x-y)$;
- What kind of "thing" is $f$ ?
  - It is a variable, of type int $\times \mathrm{int} \rightarrow \mathrm{int}$.
- What is the "value" of $f$ ?
  - The reasonable answer is that its value is the machine code for $f$, i.e., a pointer to the first instruction of that code sequence.

### If Functions Have Values ...

- If $f$ has a value, can it be assigned to another variable of the correct type?
  - Sure. That's exactly what function pointers accomplish.
  - int $\left({ }^{*} g p\right)$ (int, int); $g p=f ;$
- If $£$ has a value, can it be passed in as an argument to another function?
  <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110080419944.png" alt="image-20211110080419944" style="zoom:50%;" />
- If $f$ has a value, can it be returned as the result of calling another function?
  - Hmm. Logically, it should be, but what would that mean?
  - Let's assume for the moment that we can do this.

### If Functions Are Values...

- With simple value types (like int), we can work directly with values without having to bind them to names (i.e., named storage locations). Thus:
- int $x, y, z ;$
  $z=(x+y) *(x-y) ;$
  rather than
  - int $x, y, z ;$
  int t1 $=x+y ;$ int $t 2=x-y ;$
  $z=t 1^{*} t 2 ;$
- Can we do the same thing with function types?
  - Yes. This is what is known as an anonymous function, or lambda.
  - Here is an example in Python.
  <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110080517310.png" alt="image-20211110080517310" style="zoom:50%;" /><img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110083045257.png" alt="image-20211110083045257" style="zoom:50%;" />

### TwO Ways of Writing Functions

- Our current view is that all input arguments are presented at once, and the function evaluates its body and returns a value based on these values.
  - Definition: $f=\lambda(x, y) \cdot(x+y)$
  - Invocation: $f(1,2)$ returns $3 .$
- Is is meaningful to present the input arguments one at a time?
  - Definition: $f=\lambda x \cdot(\lambda y \cdot(x+y))$
  - Invocation:
    - $f 12$ returns $3 .$
    - What does $f 1$ return?
    - The reasonable answer is that is should return $\lambda y$. $(1+y)$, which is itself a function.
- This process of argument-by-argument elaboration of the defined multi-parameter function is called partial evaluation.

### Higher-Order Functions (HOFs)

- This generalized notion of treating functions as first-class values leads to the idea of higher-order functions.
- Specifically, a higher-order function:
  - Can appear as an R-value in an assignment statement.
  - Can appear as an L-value in an assignment statement.
  - Can be an input argument to a function call.
  - Can be the output result of a function,
  - Can be partially evaluated.
- The lambda notation is an important syntactic shortcut for defining such functions easily.
  - Not needed semantically.
  - But trying to support full-fledged higher-order functions without some syntactic support gets cumbersome really quickly. E.g., see anonymous inner classes in Java.

### Ruby example

<img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110080947759.png" alt="image-20211110080947759" style="zoom:50%;" />

#### Defining and Calling Functions

static int $x ;$ 

int $f($ int $y)$ return $x+y ;\}$

- What values does $f$ need to access in order to do its job?
  - The values of the variables $x$ and $y$. This is called the environment.
  - Where are these variables located?
    - $x$ is an uninitialized global variable, so it is in a known location in .bss.
    - The location of parameter $y$ is specified by the linkage convention.
    - The locations of any local variables of $f$ are also known, as offsets within the activation record of $f$.
  - Thus, the code generator can compile the function definition.
  - Also, no matter where $f$ is called from, even from another module, the function call will have access to all the values it needs, and will be able to compute the correct value.
- This simple two-level static scope structure allows us to organize the activation records in a call stack.
  - Will this organization work with higher-order functions?

### Activation Records for HOFs

def paidMore (amount)
	return lambda \{|e| e.salary > amount\}
end
medPaid = paidMore (50)
highPaid = paidMore(150)

- What values does medPa id need to access in order to do its job?
  - The value of the parameter e is not a problem, since it is presented at the call site.
  - The value of the variable amount is a problem, since it was the parameter that was passed in to paidMore as part of the partial evaluation process that generated medPaid.
    - By the time medPaid gets called, the activation record of paidMore containing that value has vanished from the call stack.
    - And we can't squirrel away that value in a fixed location, because we have multiple calls to paidMore with different values for amount.

### The Root Problem

- Let's rewrite these functions in lambda notation.

  paidMore $=\lambda$ amount. $(\lambda$ e. $(e$. salary $>$ amount $))$

  medPaid = paidMore(50)

  highPaid = paidMore(150)
- The functions medPaid and highPaid, which are generated by partially evaluating paidMore at runtime, share the same code body but need to bind amount differently.
  - These two values were available as parameters in the two calls of paidMore.
  - We therefore need to retain these values of amount so that they are available when medPaid and highPaid are invoked.
- In other words, the environments of medPaid and highPaid need to be split into two parts:
  - the part that was present when they were defined (amount), and
  - the part that will be present every time they are called $(e)$.

### Closures

- In other words, the environments for HOFs need to be split into two parts in general:
  - the part that was present when they were defined (the def-env); and
  - the part that will be present every time they are called (the callenv, aka the activation record).
- The notion of a HOF's "value" must therefore be extended from just the code pointer to the code pointer plus the part of the environment that was present when the HOF was defined but won't be available when it is called.
  - This combination (code, def-env) is called a closure.

<img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110090046050.png" alt="image-20211110090046050" style="zoom:50%;" />

### Implementing Closures

- Two general options
  - Allocate activation records on the heap rather than on the stack.
    - In this case, environments don't disappear in a LIFO manner, but must instead be garbage-collected.
  - Save the values of the necessary variables in a closure data structure.
    - In this case, the activation records can still be stack-allocated, but the partial evaluation process needs to copy over the necessary values into the closure so that the returned HOF will have its full environment available when it is called.
- Design choices
  - Should closures be flat or linked?
  - Should closure entries be bound to values or to locations?

# JIT Compilers

### Just-In-Time (JIT) Compilation

- When compiling LiveOak to SaM, we generate bytecode,
  - This gives us a portable distribution format to any machine that (has a SaM execution environment.
  - But it potentially makes runtime performance slow.
- When compiling LiveOak to $x 86$, we generate native code.
  - This gives up portability across machines: this binary makes sense only on $\times 86-64 /$ Linux systems.
  - But it makes runtime performance significantly faster.
- Can we get the best of both worlds?
  - Portability across hardware architectures.
  - Fast execution.
- This is the problem that JIT compilation (aka run-time compilation, binary translation, or dynamic compilation) attempts to solve.

### JIT Compilers vs. Conventional Compilers

- A JIT compiler converts methods from the portable representation to the native representation at the time of invocation, i.e., "just in time" for fast execution.
  - Essentially performs a translation from bytecode instructions to native host instructions.
  - Differs from a conventional compiler in that it doesn't have a frontend to parse a $\mathrm{HLL}$ and perform syntax checking.
  - The bytecode program is assumed to be syntactically correct (or is verified by the loader).
  - The bytecode instructions effective constitute an intermediate representation.
- Since the goal of JIT compilation is improved execution speed, optimizations are central.
  - Need to control run-time compilation overhead as many static optimizations are time-consuming (e.g., register allocation).
  - Can also stage optimizations depending on profile data (from runtime).

### Typical Dynamic Optimization Framework

<img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110081530692.png" alt="image-20211110081530692" style="zoom:50%;" />

### Profiling

- Critical component of overall architecture.
  - Can be collected by the interpreter, or be provided by the compiled code as it executes.
- A variety of instrumentation and sampling techniques can be used.
  - Often, profiling is done at the method level rather than at the basic block level.
- Common profile data
  - Code
    - Method usage (node counts in call graph).
    - Method usage refined by callers (edge counts in call graph).
    - Branch statistics (edge counts in control-flow graph).
  - Data
    - Data and pointer information.
- Different kinds of data drive different optimizations.

### Optimization #1: Selective Compilation

- Idea
  - Compile (or re-compile at a higher level at optimization) methods when they cross certain thresholds of usage.
- Trigger
  - Node count of the method in the call graph reaches a threshold (or some more complex cost-benefit analysis condition).
- Cost
  - Time lost in compiling/optimizing the method.
- Benefit
  - Time gained in future invocations of the method.
- Risk
  - No (or few) future invocations of the method.



### Optimization #2: Method Inlining

- Idea
  - Call to method $Q$ is replaced in the caller $P$ by the body of $Q$.
  - <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110092001754.png" alt="image-20211110092001754" style="zoom:20%;" />
- Trigger
  - Edge count of $(P, Q)$ in the call graph reaches a threshold.
- Cost
  - Time lost in inlining $Q$ and compiling the combined method.
- Benefit
  - Time gained in future invocations of the method $Q$ because of removal of linkage overheads.
  - Opportunity for more global optimizations in combined method.
- Risk
  - Space expansion of combined method may adversely affect instruction cache locality.
  - Edge count may be aggregation over multiple call sites.
    <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110092133918.png" alt="image-20211110092133918" style="zoom:33%;" />

### Optimization #3: Code Re-Layout

- Idea
  - Re-organize the layout of basic blocks so that the hot traces are as straight as possible (less jumps).
- Trigger
  - Edge counts on control-flow graph indicate that the most common paths are "crooked" (spatially distant).
- Cost
  - Finer-grained profiling is time-consuming.
  - Need to maintain CFG at runtime.
  - Re-organization may require reversing evaluation of conditions.
- Benefit
  - Improved locality in instruction cache.
- Risk
  - Future branch statistics may change from the profile data.



### Optimization #4: Method Specialization

- Idea
  - Create multiple specialized versions of a polymorphic method (virtual) for common special cases to improve running time.
- Trigger
  - Data type information on method parameters indicate predominance of certain types.
- Cost
  - Finer-grained profiling is time-consuming.
  - Compiling specialized versions is a time overhead.
  - Need to keep multiple code versions and dispatch among them.
- Benefit
  - Avoiding the general case in a majority of the method calls.
- Risk
  - Nothing obvious.



### The Jikes Research Virtual Machine

- A flexible open testbed to prototype virtual machine technologies and experiment with a large variety of design alternatives.
  - Originally started as the Jalapeño JVM in IBM Research.
  - Currently maintained at https://www.jikesrvm.org.
- A distinguishing characteristic of Jikes RVM is that it is implemented in the Java ${ }^{T \mathrm{M}}$ programming language and is self-hosted i.e., its Java code runs on itself without requiring a second virtual machine.
- We will use its architecture as a case study of JIT compilers.
  - Source: "Adaptive Optimization in the Jalapeño JVM", Matthew Arnold et al., Proceedings of OOPSLA '00, October 2000, Minneapolis, MN, USA, pages 47-65.

### Jikes RVM: Compile-Only Strategy

- Compiles all methods to native code before they are executed.
  - Baseline compiler: Translates bytecodes directly into native code by simulating Java's operand stack. No register allocation. (274.14 , (B/ms compilation rate)
  - Optimizing compiler: Translated bytecodes into an IR, then performs a variety of optimizations. Uses linear scan register allocation.
  - Multiple levels of optimization.
    - Level 0: A set of optimizations performed on-the-fly during translation from bytecode to $\mathbb{R}$. (8.77 B/ms, 31.3× slowdown from baseline)
    - Level 1: Level 0 + additional local optimizations $+$ some inlining. (3.59 B/ms, 76.4× slowdown from baseline, 2.4× from Level 0)
    - Level 2: Level $1+$ global optimizations. (2.07 B/ms, 132.4× slowdown, $1.7 \times$ from Level 1 )

### Jikes RVM: Compilation Scenarios

<img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110081935267.png" alt="image-20211110081935267" style="zoom: 67%;" />

### Jikes RVM: Adaptive Optimization System

<img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110082005390.png" alt="image-20211110082005390" style="zoom: 67%;" />

### Jikes RVM: Feedback-Directed Inlining

![image-20211110082039496](/Users/ayang/Library/Application Support/typora-user-images/image-20211110082039496.png)



# Bootstraping Compilers

### How Do We Compile The Compiler?

- This sounds much like:
"Which came first, the chicken or the egg?"
- If we had written our LiveOak compiler manually in x86 assembly language, we wouldn't have a problem. But ...
  - The LiveOak compiler is a Java program.
  - Java programs are run in the JVM, which is basically a large piece of $\mathrm{C}$ code.
  - C code is compiled to $x 86$ machine code by the C compiler.
  - The C compiler is itself a C program.
- Wait! Aren't we going into an infinite loop here?
- No: we will stop the process at a well-defined point and bootstrap our way up from there.

### What Are Compilers, Anyway?

- They are just programs, but they are programs of a very special kind.
  - They take as input (strings in) a source language $S$.
  - They are written in an implementation language $I$.
  - They produce as output (strings in) a target language $T$.
- Let's show this graphically as a T-diagram.
  - Or, symbolically as $\mathcal{C}(S, I, T)$.
  - <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110075329706.png" alt="image-20211110075329706" style="zoom:50%;" />
- We will use $M$ to denote machine language.
  - The only program that can run on a hardware machine is one whose implementation language is $M$.

### A Simpler Problem First

- Suppose we had written our LiveOak compiler in C rather than in Java. Then its representation is
<img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110075414771.png" alt="image-20211110075414771" style="zoom:50%;" />

- Since the C compiler is available as a binary, its representation is
  <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110075424043.png" alt="image-20211110075424043" style="zoom:50%;" />

- So we can compose these and get

<img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110095142706.png" alt="image-20211110095142706" style="zoom:33%;" />

### Variation 1: A Cross-Compiler

- LiveOak compiler for $x 86$ written in C on an M1-based Mac:
  - <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110075511220.png" alt="image-20211110075511220" style="zoom:50%;" />
- C compiler (executable) on the M1-based Mac:
  <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110075519063.png" alt="image-20211110075519063" style="zoom:50%;" />
- So we have:
  <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110075526758.png" alt="image-20211110075526758" style="zoom:50%;" />

### Variation 2: Two-Step Bootstrapping

+ C compiler for x86 written in C on an M1-based Mac:
  <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110075617800.png" alt="image-20211110075617800" style="zoom:50%;" />
+ C compiler (executable) on the M1-based Mac:
  <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110075625835.png" alt="image-20211110075625835" style="zoom:50%;" />
+ So we have (step 1):
  <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110075634101.png" alt="image-20211110075634101" style="zoom:50%;" />
+ And then (step 2):
  <img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110075648426.png" alt="image-20211110075648426" style="zoom:50%;" />

### Why Does Bootstrapping Work?

- Fundamentally, because "data" and "code" are the same.
  - They are just different interpretations applied to bit-strings.
- The compiler is special in the way that it manipulates this data-code duality.
  - When the compiler generates code, it produces and combines bitstrings that become its output data.
  - But hardware interprets these same bit-strings as instructions when executing the generated program.
- This is a very deep property of the von Neumann architecture that is also the source of many security holes.
- We will study this concept with a concrete example taken from Ken Thompson's 1983 Turing Award Lecture Reflections on Trusting Trust.

### Starting Point

- We have a native compiler $c c_{1}$ for a subset of $C$ that recognizes only some of the ASCII escape sequences for non-printing characters.
  - We want to extend our language definition to accept the escape $^{\prime} \backslash \mathrm{V}^{\prime}$ for vertical tab.
  - The routine that recognizes ASCII escape sequences and converts them to characters is in the lexer.

<img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110100413100.png" alt="image-20211110100413100" style="zoom:50%;" />

### Attempt #1

<img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110075931803.png" alt="image-20211110075931803" style="zoom:50%;" />

- This doesn't work.
- It doesn't even compile with our existing native compiler $c c_{1}$.
- Why? existing cc1 (before this change) doesn’t know ‘\v’ yet (\v is not in the language of the code written in)

### Attempt #2

<img src="/Users/ayang/Library/Application Support/typora-user-images/image-20211110075956138.png" alt="image-20211110075956138" style="zoom:50%;" />

- This works just fine. cc1 knows ‘11’
- Why?
  - We now have a new native compiler $c c_{2}$ that compiles programs written in this "extended" $\mathrm{C}$ language.

### One Final Plot Twist

- The routine convert 2 is written in the extended $\mathrm{C}$ language.
  - So we can compile the compiler source with $c c_{2}$ but with convert2 instead of convert $3 .$
  - This time, the compilation succeeds.
  - We can now forever discard convert3.
- In Thompson's words:
This is a deep concept. The original source code of convert is amazing in that it "knows" in a completely portable way what character code is compiled for a new line in any character set. This act of knowing then allows it to recompile itself, thus perpetuating the knowledge. When we wanted to add the knowledge of the new escape sequence, we had to "train" the compiler. We had to do this once and **once it "learned" the concept, we could use the selfreferencing definition**.

