# Hoisting in Javascript
Things to Keep in Mind When Handling Elements Declared with the `var` Keyword in Javascript.

<img src="hoist.webp" >

## About
Hoisting is a behavior of the JavaScript interpreter that **moves declarations of variables, functions, and classes to the top of their scope before code execution.** This allows us to reference these elements before they are declared.

However, it is important to note that **only the declarations are hoisted, not the initializations.** This means that if you try to use a variable or function before it is declared, you will get the value `undefined`.

```js
function fn() {
  console.log(message); // undefined

  var message = 'Hello!';
}

fn();
```
If you use `let` instead of `var`, you will get a `ReferenceError`.

## Breakdown

**1. Affected Elements**: Functions, variables, and classes declared with var.  
**2. Scope**: Hoisting applies within the function scope or the global scope where the element is declared.  
**3. Behavior**: The declarations are conceptually treated as if they were moved to the top of their scope.
   
## Important note
Hoisting only affects **declarations**, not the **initialization of variables**. Variables declared with var are still initialized with `undefined` before their assigned value.

While hoisting can be convenient at times, it can also lead to unexpected behavior if not understood properly. **It's generally recommended to avoid relying on hoisting**.


## Q&A
If you've understood, then you should be able to answer the following questions.

1. **What is the hoisting in Javascript?**  
   JavaScript moves up declarations of variables, functions, and classes declared with `var` to the top of their scope. 
   This allows up to reference these elements before they are declared.
   
2. **(T/F) Both the declarations and the initialization are hoisted.**  
  False, only the declarations are hoisted, not the initialization. if the hoisted element is referenced before it is declared, you will get `undefined`.

3. **What does the following code output?**
    ```js
    function fn() {
      console.log(message);

      var message = 'Hello!';
    }

    fn();
    ```

    Answer: undefined 
    
4. **Break down the behavior of hoisting, focusing on affected elements, scope, and behavior.**  
    **1. Affected Elements**: Functions, variables, and classes declared with var.  
    **2. Scope**: Hoisting applies within the function scope or the global scope where the element is declared.  
    **3. Behavior**: The declarations are conceptually treated as if they were moved to the top of their scope.

5. **Why is it generally recommended to avoid relying on hoisting?**  
  Hoisting can be convenient sometimes, but it can make code difficult to predict leading to bugs if not understood properly.

6. **What does the following code output, and Why?**  
   ```js
    var foo = 5;
    function bar() {
        if (foo === undefined) {
            var foo = 10;
        }
        console.log(foo);
    }
    bar();
   ```

   Answer: 10  
    1. In the given code snippet, foo is declared and assigned a value of 5 in the global scope.
    2. Inside the bar function, foo is redeclared and initialized to 10 within an if statement.
    3. Due to hoisting, the declaration of foo within the bar function is effectively moved to the top of the function scope.
    4. At the start of the bar function's execution, foo is undefined due to hoisting, which affects how the if statement's condition is evaluated.
    5. As a result, foo within the function scope refers to the locally declared foo rather than the global foo.
    6. Consequently, the console.log(foo) statement inside the function prints 10, which is the value of the locally declared foo within the if block of the `bar` function.
     
  7. **How do `let` and `const` differ from `var` in terms of hoisting?**  
  Unlike variables declared with `var`, which are hoisted and initialized with `undefined` at the top of their scope, variables declared with **`let` and `const` are also hoisted but not initialized.** This means they enter a "temporal dead zone" from the start of the block until the declaration is encountered. Accessing a `let` or `const` variable before its declaration results in a `ReferenceError`.