# 2. Structure of language

## Definitions

+ Compiling: *implementation technique* that involves translating a source language to some other—usually lower-level—form.
+ Compiler: translate to other form but doesn’t executes. eg. GCC and Clang for C
+ Interpreter: translate and executes immediately. eg. older Ruby

## Stages

+ [frontend](https://en.wikipedia.org/wiki/Compilers#Frontend): 
  + [translates](https://en.wikipedia.org/wiki/Translator_(computing)) a computer programming [source code](https://en.wikipedia.org/wiki/Source_code) into an [intermediate representation](https://en.wikipedia.org/wiki/Intermediate_representation). (lexer+ parser)
  + specific to source language
+ intermediate representation (IR)
+ backend: 
  + works with the intermediate representation to produce code in a computer output language. The backend usually [optimizes](https://en.wikipedia.org/wiki/Program_optimization) to produce code that runs faster. (optimization) 
  + specific to final machine architecture
+ Some designs, such as [GCC](https://en.wikipedia.org/wiki/GNU_Compiler_Collection), offer choices between multiple frontends (parsing different source [languages](https://en.wikipedia.org/wiki/Programming_language)) or backends (generating code for different target [processors](https://en.wikipedia.org/wiki/Central_processing_unit)).[[3\]](https://en.wikipedia.org/wiki/Front_end_and_back_end#cite_note-3)

### Front end

+ Scanning: A **scanner** (or **lexer**) takes in the linear stream of characters and chunks them together into a series of something more akin to “words” (token). 
+ Parsing: A **parser** takes the flat sequence of tokens and builds a tree structure (parse tree/ abstract syntax tree, **AST**) that mirrors the nested nature of the grammar, report **syntax errors**.
+ Static analysis
  + Binding/ resolution: For each **identifier**, we find out where that name is defined (scope) and wire the two together. Type check if language is statically typed. Report **type error**.
  + Sematic insights are stored in following ways:
    + back to attributes on the syntax tree itself
    + symbol table
    + transform tree (next step)

### Back end

+ Optimization:
  + constant folding: do calculations of constants
  + Lua and CPython generate relatively unoptimized code, and focus most of their performance effort on the runtime
+ Code generation: to bytecode for virtual machine, so that it can be portble between different chips
+ Virtual machine: eg. write VM in C, then language can run on any platform with C compiler

+ Runtime: with services for the running program, such as memory management, garbage collector, object represenation tracking
  + memory management
    + reference counting: simple, but limited
    + tracing **garbage collection (GC)**: most languages

### Alternative routes

+ Single path compiler: Pascal, C
+ Tree-walk interpreters: begin execution right after parsing into AST. early Ruby
+ Transpiler: source-to-source compiler/ transcompiler (to another language that is already supported by the machine). JavaScript, many languages that transpile to C in UNIX
+ JIT (just-in-time): compile when program is loaded. tends to be the fastest way to implement dynamically typed languages





# 3. Language

## Components

+ **Data types**: booleans, numbers, strings, nil/ null
+ **Expressions**: produce a value
  + Arithmetic
    + Binary (infix): `a + b`
    + uniary (prefix): `-a`
  + comparison  and equality
  + Logical
  + precedence and grouping: `()`
+ **Statements**: produce an effect. `print`
+ **Variables**
+ **Control flow**:  conditional, loop
+ **Functions**: argument, parameter
+ **Closures**: *first class*,  real values that you can get a reference to, store in variables, pass around. eg. Functions
  + `inner()` has to “hold on” to references to any surrounding variables that it uses so that they stay around even after the outer function has returned. We call functions that do this **closures**. 
+ **Classes**: 
  + class beased languages: Instances (objects), classes, inheritance
  + prototype-based languages: only objects, objects inherit from other objects



## Grammars

For context free grammar …

+ head: name
+ Body: describe what the grammar rule generates, a list of symbols
  + Terminal: literal value (`if` or `1234`)
  + nonterminal: named reference to another rule in the grammar

### Bachus-Naur form (BNF)

```
expression     → literal
               | unary
               | binary
               | grouping ;

literal        → NUMBER | STRING | "true" | "false" | "nil" ;
grouping       → "(" expression ")" ;
unary          → ( "-" | "!" ) expression ;
binary         → expression operator expression ;
operator       → "==" | "!=" | "<" | "<=" | ">" | ">="
               | "+"  | "-"  | "*" | "/" ;
```



+ Syntax tree: defined syntax types for parser and interpreter to communicate

