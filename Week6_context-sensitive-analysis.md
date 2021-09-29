# Context sensitive analysis

## Introduction

### Semantic analysis

+ Lexical: character sequence -> token sequence
+ Syntactic: token sequence -> abstract syntax tree
+ Semantic analysis: meaningful? Type, declaration order, variable initialization, number of arguments …
+ Semantic analysis examples: 
  + [Pascal] Does the declaration of each identifier precede its uses?
  - [All] Does the call of a procedure match its type signature?
  - [Rust] Have all variables been initialized before they are used?
  - [C] Does a break statement have an appropriate enclosing construct?
  - [Ada] Does the same name occur both at the beginning and the end of a named loop or block?
  - [APL] What is the type and dimensionality of a name at a program point?

### Beyond Context-Free Language Features

- The following formal languages can be proven to not be context-free.
・ $L_{1}=\left\{w c w \mid w \in(a \mid b)^{*}\right\} .$ $\cdot L_{2}=\left\{a^{n} b^{m} c^{n} d^{m} \mid n \geq 1 \wedge m \geq 1\right\} .$

- These languages abstract semantic analysis problems.
  - $L_{1}$ : identifiers are declared before their use in a program.
  - $L_{2}$ :  number of formal parameters in the declaration of a procedure agrees with the numbe\} of actual arguments in a use of the procedure.
- Why not use context-sensitive grammars?
  - The problem of parsing a context-sensitive grammar is PSPACEcomplete.
  - Even a CSG would have difficulty (or outright lack the power) to effective encode typical semantic analysis problems.



# Type checking

## Introduction

+  Definition: The set of types in a programming language, along with the rules that use types to specify program behavior, are collectively called a type system.

### Purpose

+ Ensuring run-time safety.
  - Attempt to identify and catch as many ill-defined programs as possible before they execute an operation that causes a run-time error.
  - Infer a type for each expression. Check the types of operands to an operator with the language specification of that operator. Possibly coerce values to fit the specification.
+ Improving expressiveness.
  - Specify behavior more precisely than possible with context-free rules.
  - Operator overloading: what does + signify in most modern procedural languages?
+ Generating better code.
  - If the compiler can accurately determine the types of every expression statically, it can generate type-specific assembly code.
  - This avoids the overheads of maintaining run-time data tags (space) and disambiguating types at run-time (time).

### Terminology

+ Languages
  - Statically typed: Every expression can be type-checked at compile time.
  - Dynamically typed: Some expressions can be type-checked only at run time.
  - Untyped: Really, only has one type (e.g., BCPL).
  - Weakly typed: Has a poor type system.
  - Strongly typed: Every expression can be assigned an unambiguous type.
+ Type systems and implementations
  - Strongly checked: Perform enough checking to detect and prevent all run-time errors that result from misusing a type. (Java, not C)
  - Unchecked: The implementation assumes a well-formed program.
  - Statically checked: All type inference and checking is actually done at compile time.
+ Java couldn’t be statically typed and checked because the execution model doesn’t allow seeing all the code at once.
  - type inference must be perform as classes are loaded, and some run-time checking is performed.
  - lint enforces some rules that can help

### Judgments and Typing Rules

- Typing rules contain judgments of the form $E \Rightarrow e: T$, 
  -  $E$ is an environment containing, for example, the types of identifiers and functions.
  - e: expression
  -  "In the environment $E$, expression $e$ has type $T$”
- Break into $F$ , signature, and $G$, context
  - The signature shows the types of functions
  - The context shows the types of variables.
  - So we will write $F, G \Rightarrow e: T$.
- Typing rules have the form

$$
\frac{J_{1} \quad J_{2} \ldots J_{n}}{J} C(n \geq 0)
$$
+ Interpretation
  + J1… Jn: premises 
  + C: condition
  + J: conclusion
  + Read the rule as "From the judgments $J_{1}$ through $J_{n}$, if condition $C$ holds, conclude $J$ :"
  + Judgments expressed in formal language; condition in natural language.

### Examples of Typing Rules and Derivations

- Typing rules for arithmetic expressions
- Lemmas: $\frac{E \Rightarrow e_{1} \text { int } E \Rightarrow e_{2} \text { :int }}{E \Rightarrow e_{1}+e_{2} \text { :int }}, \frac{E \Rightarrow e_{1} \text { :int } E \Rightarrow e_{2} \text { :int }}{E \Rightarrow e_{1} * e_{2} \text { :int }}$.
  - E-> E + E : if e1, e2 are int, e1+e2 is int
  - E -> E * E : likewise
- Axioms: $\overline{E \Rightarrow x: T} \quad x: T \in E, \quad \overline{E \Rightarrow i: \text { int }}$  i is an integer literal.
  - E-> id: If environment have type binding x to T
  - E-> num: If i is integer
- Given these typing rules, how does one derive the judgment $x:$ int, $y$ :int $\Rightarrow x+12 * y$ : int?
  - 12 is int
  - y int
  - 12 * y int
  - x + 12 * y int

## Type systems

### Type expressions

+ Every language construct has a type associated with it. This type will be denoted by a **type expression**.
+ Examples
  - Basic types are type expressions.
  - Certain operators (such as tuples, records, arrays, functions, classes, inheritance, ...) can be applied to other type expressions to create new type expressions. Such operators are called **type constructors.**
  - We may also give **(user-defined) names** to type expressions (think typedef in $\mathrm{C}$ or names of classes in $\mathrm{C}++\mathrm{t}$. Such type names are also valid type expressions.
+ equivalence and inclusion
  - Equivalance: For type expressions $s$ and $t, s \equiv t$ means that these expressions "represent the same type".
  - Includesion: For type expressions $s$ and $t, s\prec t$ means that a construct of type $s$ can be used in a context requiring a construct of type $t$. t includes s

### Basic Example

- A type system modeled after C's type system.
- Basic types
  - Arithmetic types: char, int, uint, long, ulong, float, double.
  - The void type specifies an empty set of values.
  - The error type will be used to signal a type inference error. Not accessible for user, only for returning error during type checking
- Derived types
  - Arrays: If $T$ is a type expression and $N$ is an integer literal, then ARRAY $(N, T)$ is a type expression denoting the type of an array with elements of type $T$.
  - Products (tuples): If $T_{1}$ and $T_{2}$ are type expressions, then their Cartesian product $\mathrm{T}_{1} \times T_{2}$ is a type expression. The operator $\times$ is left-associative. T1xT2 != T2xT1 (order sensitive)
  - Records (struct): If $I_{1}, \ldots, I_{k}$ are distinct identifiers, and $T_{1}, \ldots, T_{k}$ are type expressions, then RECORD $\left(I_{1}: T_{1}, \ldots, I_{k}: T_{k}\right)$ is a type expression denoting a record type with $k$ named fields. Whether order sensitive is language dependent
  - Pointers (address): If $T$ is a type expression, then PTR $(T)$ is a type expression denoting the type "pointer to an entity of type $T^{\prime \prime}$.
  - Functions: If $D$ and $R$ are type expressions denoting a domain type and a range type, then $D \rightarrow R$ is a type expression denoting the type of a function mapping elements of $D$ to elements of $R . \rightarrow$ is right-associative.
- Usage
  - `+ : (int x int) -> int`



### Type Inference for Expressions and Statements

<img src="Untitled.assets/image-20210929110654509.png" alt="image-20210929110654509" style="zoom:50%;" />

- How do we type-check programs in this language?
  - What effect do declarations have on judgments?
  - What are the typing rules for expressions?
- Initial environment
  - $F=\{$ mod: int $\times$ int $\rightarrow$ int $\}$
  - $G=\emptyset$