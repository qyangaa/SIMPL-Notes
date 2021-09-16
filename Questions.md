+ Why do we have `MALLOC` here `SaM Examples/test.sam`

+ + ```
    // postfix.sam
    
    PUSHIMM 1
    PUSHIMM 2
    SUB
    PUSHIMM 3
    SUB
    PUSHIMM 4
    MALLOC 
    STOP
    ```

+ Can we have labels of same name?
+ Is there any requirements/ guideline on error reporting? How would that be graded?
+ `lvl0-good-medium-6.lo` is actually lvl1 (has `while` `if` `else`)
+ How to handle STRREP? There is no str concat commands and PUSHIMMSTR doesn’t garantee continugous memory. Or handle in compile time?

+ Instructions

