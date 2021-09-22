# Register Allocation

## Background

### Framing The Register Allocation Problem

- Problem
  - GPR access is faster than memory access
  - large number of program variables and temporary values, small number of GPRs.
- Key idea
  - Generate "abstract" assembly code **assuming an unbounded number of  virtual registers**.
  - Perform **register allocation i.e., map the virtual registers to the fixed finite number of GPRs**, 
    - satisfying semantic correctness, architectural idiosyncrasies, and linkage protocol constraints.
- Key questions
  - Scope: Local, global, or interprocedural (across function calls, complicated)
  - When: Static or dynamic
- Keymetrics
  - Size of generated code, execution speed of generated code, speed of allocation.



### Common Issues

- Machine idiosyncrasies
  - [Aliasing] Assigning a value to one register can affect the value of another , eg. rax, eax, ax, al
  - [Register configurations] Register pairs, e.g., full-width multiplication.
  - [Miscellaneous] Destructive operations, condition flags.
- Pre-coloring
  - Forcing some variables to be assigned to particular registers, e.g., arguments and return values in procedure linkage on $\times 86$.
- Problem complexity
  - **Global register allocation is NP-complete**, by a reduction from the standard NP-complete problem [GT4] of graph k-colorability ( "Given a graph $G=(V, E)$ and a natural number $k$ such that $2<k \leq|V|$, determine whether or not there is an $k$-coloring of $G^{\prime \prime}$ ).
  - Not a significant problem in practice, but worst-case scenarios can be constructed.



### Design Space Dimensions (for global register allocation)

- [Ref: "Register Allocation Deconstructed", D. R. Koes and S. C. Goldstein, Proc. $12^{\text {th }}$ International Workshop on Software \& Compilers for Embedded Systems (SCOPES), pp. 21-30. 2009.]
- Assignment
  - The action of assigning a register to a variable.
  - E.g., integrated optimal, graph heuristic (for static), linear scan heuristic (fast, dynamic, JIT compilers).
- Spilling
  - The action of storing a variable into memory instead of registers.
  - E.g., integrated optimal, separate optimal, separate heuristic.
- Move Insertion
  - The action of inserting register-register moves, i.e., making a variable live in different registers during its lifetime.
  - E.g., Integrated optimal, integrated optimal ignoring uncoalescable, separate optimal, separate aggressive, none.
- Coalescing
  - The action of limiting the number of moves between registers, thus limiting the total number of instructions.
  - E.g., Full, limited, none.



## Local Register Allocation

Improved Local Register Allocation
- Scope
  - Evaluating an arithmetic expression.
  - No reordering of AST using commutative or associative properties.
  - No common subexpression elimination.
- Example: $(A-B)+((C+D)+(E * F))$.
  - How many temporaries does it take to generate 3-address code for this expression? 11 -> 6 -> 3

  - [Q: the lecture mentioned algorithm, what algorithm is that?]

    ![image-20210922091847021](Week5_Register_Allocation.assets/image-20210922091847021.png)

### Sethi-Ullman Numbering Algorithm

- [Ref: "The Generation of Optimal Code for Arithmetic Expressions", R. Sethi and J. D. Ullman, Journal of the ACM 17(4), pp. 715-728. October 1970.]

- Two-pass algorithm.
  - Pass 1: Recursively determine $T_{E}$, the minimum number of temporaries required to evaluate the given expression $E$.
    - Recursive definition of $T_{E}$ :
    - 

  $$
  \begin{array}{cc}
  E \rightarrow \text { id } & T_{E}=1 \\
  E \rightarrow \text { unop } E_{1} & T_{E}=T_{E_{1}} \\
  E \rightarrow E_{1} \text { binop } E_{2} & T_{E}=\left\{\begin{array}{cc}
  \max \left(T_{E_{1}}, T_{E_{2}}\right), \quad \text { if } T_{E_{1}} \neq T_{E_{2}} \\
  1+T_{E_{1}}, & \text { otherwise }
  \end{array}\right.
  \end{array}
  $$

  - Pass 2: Recursively generate code for $E$, being supplied with a list of temporary names (i.e., registers) of length $\geq T_{E}$.
    - <img src="Week5_Register_Allocation.assets/image-20210922092518218.png" alt="image-20210922092518218" style="zoom:50%;" />

- **Doesn't change the amount of computation.**

- Will in general reduce register pressure, resulting in **fewer GPR spills**.

## Control Flow Graphs and Liveness Analysis

### Motivation

- global register allocation requires information about the liveness of names.
  - not explicit in the program.
  - Must be calculated statically (i.e., at compile-time).
  - Must correctly characterize all dynamic (run-time) paths.
- Control flow makes it hard to extract this information.
  - Branches and loops in programs.
  - Different branches may be taken in different executions.
  - Different numbers of loop iterations may be executed in different executions.

### Control Flow Graphs (CFG)

#### Definition

- a graph representation of the computation and control flow in the program.
  - Provides a framework for static analysis of program control-flow.
- CFG nodes: **basic blocks**.
- CFG edges:  possible **flow of control** from the end of one basic block to the beginning of another basic block.
  - Edges are sometimes labeled with the Boolean value for which they are taken.
  - There may be multiple incoming/outgoing edges for a given basic block.
- A possible execution is a **consistent path** in the CFG.
  - There may be paths in the CFG that correspond to infeasible executions.
  - Happens if result from previous block can affect result of the next block

#### Examples

<img src="Week5_Register_Allocation.assets/image-20210922094250301.png" alt="image-20210922094250301" style="zoom:50%;" />

H/ LLIR: High / low level intermediate language

<img src="Week5_Register_Allocation.assets/image-20210922094415264.png" alt="image-20210922094415264" style="zoom:50%;" />

#### Usage

- Extract information including live values at compile time (statically) 
- Process
  - Define 2 program points in the CFG. Defined for instructions and basic blocks
    <img src="Week5_Register_Allocation.assets/image-20210922094714545.png" alt="image-20210922094714545" style="zoom:50%;" />
    - Within a basic block, the program point after an instruction is the same as the program point before its successor instruction
      <img src="Week5_Register_Allocation.assets/image-20210922095109088.png" alt="image-20210922095109088" style="zoom:50%;" />
  - Extract flow of information between these program points.
    - Effect of instruction execution
    - Effect of control flow

### Live variables as an Example

#### Definition

- A variable $v$ is live at a point $p$ in a CFG if **there is a path from $p$ to a use of $v$, and that path does not contain a (re-)definition of $v$.**
  - A statement is a definition of a variable $v$ if it may write to $v$.
  - A statement is a use of variable $v$ if it may read from $v$.
- Computing liveness
  - Write down a system of equations that define live variable sets at each point in the CFG.
  - Solve the system iteratively.
- <img src="Week5_Register_Allocation.assets/image-20210922095358955.png" alt="image-20210922095358955" style="zoom:50%;" />

+ Compute $\text{use}(I)$ and $\operatorname{def}(I)$ sets for an instruction $I$ based on its structure.
  - $I \rightarrow x=y$ binop $z: \operatorname{use}(I)=\{y, z\} ; \operatorname{def}(I)=\{x\}$.
  - $I \rightarrow x=$ unop $y:$ use $(I)=\{y\} ; \operatorname{def}(I)=\{x\}$.
  - $I \rightarrow x=y: \operatorname{use}(I)=\{y\} ; \operatorname{def}(I)=\{x\}$.
  - $I \rightarrow x=f\left(y_{1}, \ldots, y_{n}\right): \operatorname{use}(I)=\left\{y_{1}, \ldots, y_{n}\right\} ; \operatorname{def}(I)=\{x\}$.
  - $I \rightarrow$ if $(x): \operatorname{use}(I)=\{x\} ; \operatorname{def}(I)=\emptyset$
  - $I \rightarrow$ return $x:$ use $(I)=\{x\} ; \operatorname{def}(I)=\emptyset$
+ For instruction $I$, let:
  - $\operatorname{In}(I)=$ the set of live variables at the program point before $I$.
  - Out $(I)=$ the set of live variables at the program point after $I$.
+ For basic block $B$, let:
  - $\operatorname{In}(B)=$ the set of live variables at the program point before $B$.
  - $\operatorname{Out}(B)=$ the set of live variables at the program point after $B$.

#### Computation

##### Out(I) -> In(I): + use - def

- <img src="Week5_Register_Allocation.assets/image-20210922100012763.png" alt="image-20210922100012763" style="zoom:50%;" />
- General rule

$$
\operatorname{In}(I)=(\operatorname{Out}(I) \backslash \operatorname{def}(I)) \cup \operatorname{use}(I)
$$
- Backward flow of information.
  - Given $\operatorname{Out}(B)$, can compute $\operatorname{In}(B)$.
    <img src="Week5_Register_Allocation.assets/image-20210922100039163.png" alt="image-20210922100039163" style="zoom:50%;" />

##### Control flow - Union successor

<img src="Week5_Register_Allocation.assets/image-20210922100319814.png" alt="image-20210922100319814" style="zoom:50%;" />

- General rule
$$
\operatorname{Out}(B)=\bigcup_{B^{\prime} \in \operatorname{succ}(B)} \operatorname{In}\left(B^{\prime}\right)
$$
- Backward flow of information.
  - Given all the $\operatorname{In}\left(B^{\prime}\right) \mathrm{s}$, can compute $\operatorname{Out}(B)$.



##### Iteration

- Initialize all live variable sets to $\emptyset$.
- select an instruction $I$ or basic block $B$ and update $\ln (I)$ or $\operatorname{Out}(B)$ accordingly.
- Stop when we reach a fixed point (i.e., sets don't change any more, i.e., all equations have been satisfied).

<img src="Week5_Register_Allocation.assets/image-20210922100746361.png" alt="image-20210922100746361" style="zoom:50%;" />

## Global register allocation

### Graph Coloring Algorithm

+ CFG definitions
  + CFG $G=(V, E)$, 
  + every node (i.e., basic block) $b \in V$, 
  + Out $(b)$, the set of names live out of $b$.

+ Global register allocation:
  + assign temporary variables to a fixed set of registers.

+ Requirements
  + Two simultaneously live variables cannot be allocated to the same register (to interfere)
  + Two names $m$ and $n$ interfere if **either**:
    + **Both** names are initially live (e.g., function arguments), or 
    +  $\exists b \in V:\{m, n\} \subseteq \operatorname{Out}(b)$.
+ Interference Graph: YAG
  + Given the CFG $G=(V, E)$, its interference graph $I_{G}=\left(V^{\prime}, E^{\prime}\right)$ is defined as follows.
    - nodes - variable names: $V^{\prime}$ 
    - edge: if names interfere $(m, n) \in E^{\prime}$ (live at same time, need to be on different register)
+ Graph coloring algorithm
  + Definitions:
    + color: registers
    + Illegal: if two interfering nodes have same register(color)
  + Example
    <img src="Week5_Register_Allocation.assets/image-20210922102124110.png" alt="image-20210922102124110" style="zoom:50%;" />
  + Finding optimal register allocation to avoid move instructions is NP complete