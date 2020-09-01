---
title: Lua 
categories: lua
date: 2020-08-15 15:58:44
tags: [lua, learn]
---

Lua learning
<!--more-->

## Values and Types

Lua is a **dynamically typed** language. This means that variables do not have types; only values do. There are no type definitions in the language. All values carry their own type.

All values in Lua are **first-class** values. This means that all values can be stored in variables, passed as arguments to other functions, and returned as results.

There are eight basic types in Lua: nil, boolean, number, string, function, userdata, thread, and table.

1. nil  
    The type *nil* has one single value, **nil**, whose main property is to be different from any other value. It usually represents the absence of a useful value.  

2. boolean  
    The type *boolean* has two values, **false** and **true**. **Both nil and false make a condition false; any other value makes it true.**

3. number  
    The type *number* uses two internal representations, or two subtypes, one called **integer** and the other called **float**.

4. string  
    The type *string* represents **immutable** sequences of bytes. Lua is 8-bit clean: strings can contain any 8-bit value, including embedded zeros ('\0'). Lua is also **encoding-agnostic**.

5. function  
    Lua can call (and manipulate) functions written in Lua and functions written in C . Both are represented by the type function.

6. userdata  
    A *userdata* value represents a block of raw memory. There are two kinds of userdata: **full userdata**, which is an object with a block of memory managed by Lua, and **light userdata**, which is simply a C pointer value.

7. thread  
    The type *thread* represents independent threads of execution and it is used to implement **coroutines**. Lua threads are **not** related to operating-system threads.

8. table  
    The type *table* implements associative arrays, that is, arrays that can have as indices not only numbers, but any Lua value except nil and NaN.

Tables, functions, threads, and (full) userdata values are objects: variables do not actually contain these values, only references to them.

## Lexical Conventions

Lua is a free-form language. It **ignores** spaces (including new lines) and comments between lexical elements (tokens), except as delimiters between names and keywords.

Lua is a **case-sensitive** language.

### Literal String

    a = 'alo\n123"'
    a = "alo\n123\""               -- \xXX \ddd \u{XXX}
    a = [[                         -- this newline is not included in the string
        alo
        \n                         -- do not interpret any escape sequences
    ]]

### Numeric Constant

    3   345   0xff   0xBEBADA
    3.0     3.1416     314.16e-2     0.31416E1     34e1
    0x0.1E  0xA23p-4   0X1.921FB54442D18P+1

### Comment

    -- short comment
    --[[
        long comment
    ]]

## Variables

Variables are places that store values. There are three kinds of variables in Lua: *global variables*, *local variables*, and *table fields*.

Any variable name is assumed to be global unless explicitly declared as a local.

    a = 1           -- global variable
    local a = 1     -- local variable

Before the first assignment to a variable, its value is **nil**.

The syntax `var.Name` is just syntactic sugar for `var["Name"]`.

## Statements

### Blocks

A block is a list of statements, which are executed sequentially.

Lua has *empty statements* that allow you to separate statements with semicolons, start a block with a semicolon or write two semicolons in sequence.

    a = b + c
    (print or io.write)("done")
Since Lua ignores spaces (including new lines), it equals to:

    a = b + c(print or io.write)('done')
To avoid this ambiguity, it is a good practice to always precede with a semicolon statements that start with a parenthesis:

    ;(print or io.write)('done')

A block can be explicitly delimited to produce a single statement:

    do block end

Explicit blocks are useful to control the scope of variable declarations. Explicit blocks are also sometimes used to add a **return** statement in the middle of another block.

### Chunks

Lua handles a chunk as the body of an anonymous function with a variable number of arguments.

A chunk can be stored in a file or in a string inside the host program. To execute a chunk, Lua first loads it, precompiling the chunk's code into instructions for a virtual machine, and then Lua executes the compiled code with an interpreter for the virtual machine.

Chunks can also be precompiled into binary form; see program `luac` and function `string.dump` for details. Programs in source and compiled forms are interchangeable; Lua automatically detects the file type and acts accordingly (see `load`).

### Assignment

Lua allows multiple assignments. Before the assignment, the list of values is adjusted to the length of the list of variables.  

* If there are more values than needed, the excess values are thrown away.  
* If there are fewer values than needed, the list is extended with as many nil's as needed.  
* If the list of expressions ends with a function call, then all values returned by that call enter the list of values, before the adjustment (except when the call is enclosed in parentheses).

<!-- -->

    i = 3
    i, a[i] = i+1, 20   -- The assignment statement first evaluates all its expressions
                        --   and only then the assignments are performed.
    x, y, z = y, z, x   -- Exchanges values.

### Control Structures

* **while** *exp* **do** *block* **end**

* **repeat** *block* **until** *exp*  
  The condition can refer to local variables declared inside the loop block.

* **if** *exp* **then** *block* **elseif** *exp* **then** *block* **else** *block* **end**

* **return**  
  The return statement can only be written as the last statement of a block.  
  If it is really necessary to return in the middle of a block, then an explicit inner block can be used, as in the idiom **do return end**.

* **break**
  
* **~~continue~~**

* **goto**

        for i=1,10 do
            if i% 2 == 0 then goto continue end
            print(i)
            ::continue:: -- same scope
        end

* **for** *v = e1, e2, e3* **do** *block* **end**  
The **numerical for** is equivalent to the code:

        do
            local var, limit, step = tonumber(e1), tonumber(e2), tonumber(e3)
            if not (var and limit and step) then error() end
            var = var - step
            while true do
                var = var + step
                if (step >= 0 and var > limit) or (step < 0 and var < limit) then
                    break
                end
                local v = var
                block
            end
        end

  * All three control expressions are evaluated only once, before the loop starts. They must all result in numbers.
  * `var`, `limit`, and `step` are invisible variables. 
  * If the third expression (the step) is absent, then a step of 1 is used.
  * **The loop variable v is local to the loop body.**

* **for** *var_1, ···, var_n* **in** *explist* **do** *block* **end**  
The **generic for** is equivalent to the code:

       do
            local f, s, var = explist
            while true do
                local var_1, ···, var_n = f(s, var)
                if var_1 == nil then break end
                var = var_1
                block
            end
       end

  * `explist` is evaluated only once. Its results are an iterator function, a state, and an initial value for the first iterator variable.
  * `f`, `s`, and `var` are invisible variables.
  * he loop variables `var_i` are local to the loop

### Expressions

* Arithmetic Operators

  * **+**               : addition
  * **-**               : substraction
  * **\***              : multiplication
  * **/**               : float division
  * **//**              : floor division
  * **%**               : modulo
  * **^**               : exponentiation
  * **-**               : unary minus

  If both operands are integers, the operation is performed over integers and the result is an integer.  
  Otherwise, if both operands are numbers or strings that can be converted to numbers, then they are converted to floats.  
  Exponentiation and float division (/) always convert their operands to floats and the result is always a float.  

* Bitwise Operators

  * **&**               : bitwise AND
  * **|**               : bitwise OR
  * **~**               : bitwise exclusive OR
  * **>>**              : right shift
  * **<<**              : left shift
  * **~**               : unary bitwise NOT

  All bitwise operations convert its operands to integers.  
  Both right and left shifts fill the vacant bits with zeros.

* Relational Operators
  
  * **==**              : equality
  * **~=**              : inequality
  * **<**               : less than
  * **>**               : greater than
  * **<=**              : less or equal
  * **>=**              : greater or equal

  Equality (==) first compares the type of its operands. If the types are different, then the result is false.  
  Tables, userdata, and threads are compared by reference.  
  Equality comparisons do not convert strings to numbers or vice versa. Thus, "0"==0 evaluates to false, and t[0] and t["0"] denote different entries in a table.

* Logical Operators

  * **and**
  * **or**
  * **not**
  
  The conjunction operator **and** returns its first argument if this value is false or nil; otherwise, **and** returns its second argument.  
  The disjunction operator **or** returns its first argument if this value is different from nil and false; otherwise, **or** returns its second argument.  
  Both **and** and **or** use short-circuit evaluation; that is, the second operand is evaluated only if necessary.

* Concatenation

  * **..**              : string concatenation operator

  Operands are converted to strings.

* The Length Operator

  * **#**               : length operator

  The length of a string is its number of bytes.  
  The length operator applied on a table returns a border in that table.  

      #{1, 2, 3, 4, 5}          -- 5
      #{1, 2, 3, nil, 5}        -- 3
      #{nil, 2, 3, 4, 5}        -- 0

* Table Constructors

      a = {1, "2", 3, }
      a = {[1] = 1, x = 2, ["y"] = 3}
      a = { [f(1)] = g; "x", "y"; x = 1, f(x), [30] = 23; 45 }

  The order of the assignments in a constructor is undefined.  
  If the last field in the list has the form exp and the expression is a function call or a vararg expression, then all values returned by this expression enter the list consecutively.

* Function Calls
  