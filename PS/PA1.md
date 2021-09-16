# Instructions

P3. ⟨*Program*⟩ → ⟨*MethodDecl*⟩

```
PUSHIMM 0 // [Q: what is this used for?]
LINK
JSR main
POPFBR
STOP
[MethodDecl]
```

P5: <MethodDecl> -> <Body>

```
main:
[Body]
main_end:
STOREOFF -1	// return to caller
ADDSP -n_vars // remove all variables
JUMPIND //[Q: does it jump to ]
```



P6. ⟨*Body*⟩ → (⟨*VarDecl*⟩)* ⟨*Block*⟩

```
(numVariables = varDecl())
ADDSP numVariables
[Block]
DUP // dup return value in callee stack [Q: is this right?]
JUMP main_end
```

P9. ⟨*VarDecl*⟩ → ⟨*Type*⟩ ⟨*Identifier*⟩ ( , ⟨*Identifier*⟩)* ;

```
(add to symbol table: {identifier: name, type, offset})
(return number_of_identifiers)
```

P10. ⟨*Block*⟩  →  { (⟨*Stmt*⟩)+ }

```
[Stmt]*
```

P15. ⟨*Stmt*⟩ →⟨Var*⟩ = ⟨*Expr⟩ ;

```
(symbol = getVar())
[Expr]
STOREOFF symbol.offset // offset = i_var + 1, i.e. for variable 1, offset = 2, offset1 is used to store PC
```

P16. ⟨*Stmt*⟩ → ;

```
(skip token)
```

P22. ⟨*Expr*⟩ →( ⟨*Expr*⟩ ?  ⟨*Expr*⟩ : ⟨*Expr*⟩ )

```
[Expr1]
JUMPC if_n // n used to count symbols to avoid name conflict
[ExprElse]
JUMPC next_n
if_n:
[ExprIf]
next_n
```

P23. ⟨*Expr*⟩ →( ⟨*Expr*⟩ ⟨*Binop*⟩ ⟨*Expr*⟩ )

```
[Expr1]
[Expr2]
[Binop]
```



P24. ⟨*Expr*⟩ →( ⟨*Unop*⟩ ⟨*Expr*⟩ )

```
[Expr]
[Binop]
```

P23. ⟨*Expr*⟩ →( ⟨*Expr*⟩ )

```
[Expr]
```



P26. ⟨*Expr*⟩ →⟨*Var*⟩ [Q: is this right?]

```
(symbol = getVar())
PUSHOFF symbol.offset
```



P27. ⟨*Expr*⟩ →⟨*Literal*⟩ 

```
[Literal]
```



P28.  ⟨*Binop*⟩ → […]

```
switch symbol.type:
	case ADD:
		return ADD
	...
```

P29. ⟨*Unop*⟩ → […]

```
// similar to <Binop>
```



P35. ⟨*Literal*⟩ →⟨*Num*⟩ [Q: do we need to have an extra layer just to match the grammar or return `PUSHIMM val` here directly?]

```
[Num]
```



P41. ⟨*Var*⟩ →⟨*Identifier*⟩ 

```
(return symbolTable[identifier])
```



P42. <Num>

```
PUSHIMM val
```

P43. <String>

```
PUSHIMMSTR str
```

P44. <Identifier> [Q: how to handle this?]

```

```





# Examples





```
// test extra parenthesis, string, string operation
String a,b;

{
a = (((("hello world"))));
b = (a * 5);
}
```

```
PUSHIMM 0		[0]
LINK				[0, 1(fbr)]
JSR main		[0, 1(fbr), 4(pc_s)] -> main
POPFBR			[heap2]
STOP				[heap2]
main:		
ADDSP 2			[0, 1(fbr), 4(pc_s), _, _]
PUSHIMMSTR "hello world" [0, 1(fbr), 4(pc_s), _, _, heap1]
STOREOFF 2	[0, 1(fbr), 4(pc_s), heap1, _]
PUSHOFF 2		[0, 1(fbr), 4(pc_s), heap1, _, heap1]
PUSHIMM 5		[0, 1(fbr), 4(pc_s), heap1, _, heap1, 5]
STRREP			[0, 1(fbr), 4(pc_s), heap1, _, heap2]
STOREOFF 3	[0, 1(fbr), 4(pc_s), heap1, heap2]
DUP					[0, 1(fbr), 4(pc_s), heap1, heap2, heap2]
JUMP main_end -> main_end
main_end:
STOREOFF -1	[heap2, 1(fbr), 4(pc_s), heap1, heap2]
ADDSP -2		[heap2, 1(fbr), 4(pc_s)]
JUMPIND 		[heap2, 1(fbr)] -> 4(pc_s)
```





```
// logic.sam

PUSHIMM 4		[4]
PUSHIMM 5		[4, 5]
EQUAL				[0]
PUSHIMM 4		[0, 4]
PUSHIMM 5		[0, 4, 5]
LESS				[0, 1]
OR					[1]
NOT					[0]
STOP
```



# Test Cases

lvl0-good-easy-9.lo [Q: what does the resulting stack look like?]

```
int a;
{
a = ((5) + 6);
}
```

```
PUSHIMM 0
LINK
JSR main
POPFBR
STOP
main:
ADDSP 1
PUSHIMM 5
PUSHIMM 6
ADD
STOREOFF 2
DUP
JUMP main_end
main_end:
STOREOFF -1
ADDSP -1
JUMPIND
```

lvl0-good-medium-4.lo

```
// test boolean operation
bool a, b, c;

{
a = (true); // 1
b = (false); // 0
c = (((!a) & b) | (!b)); // 1
}
```

```
PUSHIMM 0
LINK
JSR main
POPFBR
STOP
main:
ADDSP 3
PUSHIMM 1
STOREOFF 2
PUSHIMM 0
STOREOFF 3
PUSHOFF 2
NOT
PUSHOFF 3
AND
PUSHOFF 3
NOT
OR
STOREOFF 4
DUP
JUMP main_end
main_end:
STOREOFF -1
ADDSP -3
JUMPIND
```

