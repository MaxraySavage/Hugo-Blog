---
title: "PEMDAS From Scratch"
date: 2021-09-14T15:48:44-04:00
# weight: 1
# aliases: ["/first"]
tags: 
   - javascript
author: "Maxray Savage"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Coding an arithmetic expression evaluator in Javascript"

disableHLJS: false # to disable highlightjs
disableShare: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/MaxraySavage/Hugo-Blog/content/posts/PEMDAS-from-scratch.md"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

## Intro
A few months ago I made an edutainment game for The Odin Project's first game jame. My team had decided to build a game for practicing arithmetic expressions. I took on the task of generating the expressions as appropriate for each level of the game. It turned out to be rather challenging to generate random arithmetic expressions that were within certain difficulty constraints. For example, I didn't want the expression to evaluate to a negative number at any point along the evaluation path because we didn't want to require knowledge of negative numbers. In order to ensure this, I had to write my own script to evaluate expressions step by step so I could tell if my generated expressions conformed to my difficulty constraints. This meant I had to encode PEMDAS from scratch. It seems like every few months there's a viral tweet about some arithmetic expression that looks like the solution should be one thing but if strictly evaluated with PEMDAS evaluates to something else. I really enjoyed digging into the mechanics of solving an expression so I am writing this blog post to share.

## Let's Define the Problem
I've found that when you attempt to solve a problem with code you can either spend a few extra minutes diefining your terms really well in the beginning or get totally lost halfway through whatever your project is. So let's take a moment to think about what we're doing here. 

### What are we making?
A function written in javascript. The function should take an arithmetic expression as in input and return the result of evaluating the expression.

### What is an arithmetic expression?
We have some choices here. We could represent an arithmetic expression as a string but for the purposes of this post, I will represent expressions with arrays so for example, `3 + 2` will be an array with 3 elements like this: `[3, '+', 2]`.

We'll also include parentheses, so we could have something like `(3 + 2) * 7` which would look like `['(', 3, '+', 2, ')', '*', 7]`.

So our input will be an array of this form. We will trust that our inputs are well formed for the purposes of this article. 

### How do you evaluate an expression?
There's a relatively ubiquitous algorithm for evaluating arithmetic expressions called PEMDAS. This algorithm tends to be pretty tricky as evidenced by the biannual viral tweet where the internet argues about the right way to evaluate some expression or another. PEMDAS is not only an algorithm, it is also an acronym! It stands for 

- Parentheses
- Exponentiation
- Multiplication
- Division
- Addition
- Subtraction 

The thing is, PEMDAS actually only has 4 tiers of evaluation, not the 6 you might expect. 

Let's consult this handy diagram: 

![PEMDAS diagram](/pemdas-diagram.png)

So the multiplication and division happen in the same tier and the addition and subtraction happen in the same tier. Operators in the same tier are evaluated from right to left.

In pseudocode, the PEMDAS algorithm looks like

```
function evaluateExpression (arithmetic expression):
   while expression contains parentheses:
      replace the expression within parentheses with its evaluated form
   while the the expression contains binary operators
      execute the binary operators in the correct order
```

* **Parentheses**: 
   - Evaluate subexpressions in parentheses 
   - Replace the subexpression in parentheses with the result of evaluation
   - Examples:
      - `(2 + 3) * 4` would become `5 * 4`
      - We must evaluate the deepest nested pair of parentheses first so `(2 + (15 / 5)) * 4` would first become `(2 + 3) * 4` and then `5 * 4`
* **Exponents**: 
   - Exponents are the first tier of operators to evaluate
   - Exponents are denoted in javascript with the binary operator `**`
   - Examples:
      - `12 + 3 ** 2` becomes `12 + 9`
      - Evaluated right to left (which is weird!) so `2 ** 1 ** 3` becomes `2 ** 1` becomes `2` which is NOT what would happen if you went left to right. That would be `2 ** 1 ** 3` then `2 ** 3` finally becoming `8`
* **Multiplication and Division**: 
   - Multiplication and division are the second tier of operators to evaluate
   - In javascript, multiplication is represented by `*` and division is represented by `/`
   - We will evaluate multiplication and division from left to right, simultaneously
   - Examples:
      - `6 + 12 / 4 * 2` becomes `6 + 3 * 2` becomes `6 + 6`
* **Addition and Subtraction**: 
   - Addition and subtraction are the second tier of operators to evaluate
   - In javascript, addition is represented by `+` and subtraction is represented by `-`
   - Addition and subtraction are evaluated from left to right, simultaneously
   - Examples:
      - `18 - 7 + 4` becomes `11 + 4` becomes `15`

### Plan of attack
So we know what are inputs and outputs are and understand our general steps we need to take to make this happen. My sense is that parentheses will be best implemented using recursion after we get `EMDAS` done. So we'll start with solving expressions without parentheses and go from there.


## Solution
We will evaluate the expression in three tiers, first `exponentiation`, then `multiplication and division`, then `addition and subtraction`. We'll get to parentheses at the end. 

Let's declare an array called evaluationTiers where each index contains all the operands we will look for at each tier. For us this will look like:

```javascript
const evaluationTiers = [
   '**',
   ['x', '/'],
   ['+', '-']
];
```

Now we can start to build our function `evaluateExpression` (I'm using the spread operator `...` here to copy the input array into a new array object)
Remember that, for us, an expression is an array that looks like `[1, '+', 2]`

```javascript
function evalExpression(expressionArray) {
   const expression = [...expressionArray];
   for(const tier of evaluationTiers) {
      // do something
   }
}   
```
            
What's the something though? We have to look through the expression to see if the operation in the current tier is represented. I did this with the findIndex array method as follows:

```javascript 
let nextOperatorIndex = expression.findIndex((o) => tier.includes(o));
```
            
This will return the index of the first item in the expression array that is included in the array of operands in the current tier or return -1 if none are found. We want to continue evaluation on the array until there are no more operands in our current tier. Therefore we can implement a while loop where we do something to the expression array while our nextOperatorIndex is not -1. That would look like this:

```javascript
function evalExpression(expressionArray) {
   const expression = [...expressionArray];
   for(const tier of evalTiers) {
      let nextOperatorIndex = expression.findIndex((o) => tier.includes(o));
      while (nextOperatorIndex !== -1) {
         //do something
         nextOperatorIndex = expression.findIndex((o) => tier.includes(o));
      }
   }
}   
```
            
So in our while loop we have to do something with nextOperatorIndex. We want to evaluate whatever operation that nextOperatorIndex is pointing us to. Let's look at a small example. If we had the array [1, '+', 3, 'x', 7] and nextoperatorIndex is currently 3, what should we do? We need to:
- Look at the operator at index 3
- See if it tells us to multiply, divide, add, etc.
- Get the result of performing the operation on the numbers at index 2 and 4 
- Then replace the items at index 2, 3 and 4 with the result
So in this case we replace the `[3, 'x', 7]` portion of the array with 21 leaving us with `[1, '+', 21]`. How do we do this with code? First we can write a helper function to handle figuring out what operation to do and then doing it.

```javascript
function evalOperation(operandOne, operator, operandTwo) {
   switch (operator) {
      case '**':
        return operandOne ** operandTwo;
      
      case 'x':
        return operandOne * operandTwo;
  
      case '/':
        return operandOne / operandTwo;
  
      case '+':
        return operandOne + operandTwo;
  
      case '-':
        return operandOne - operandTwo;
  
      default:
        return 'Error, invalid operator';
    }
}
```
            
We can then use this helper function to replace the array elements from index = nextOperatorIndex - 1, to nextOperatorIndex + 1 with the result of the operation like so:

```javascript
function evalExpression(expressionArray) {
   const expression = [...expressionArray];
   for(const tier of evalTiers) {
      let nextOperatorIndex = expression.findIndex((o) => tier.includes(o));
      while (nextOperatorIndex !== -1) {
         const operator = expression[nextOperatorIndex];
         const operandOne = expression[nextOperatorIndex - 1];
         const operandTwo = expression[nextOperatorIndex + 1];
         const result = evalOperation(operandOne, operator, operandTwo);
         expression.splice(nextOperatorIndex - 1, 3, result);
         nextOperatorIndex = expression.findIndex((o) => tier.includes(o));
      }
   }
   return expression[0];
}    
```     

But we have one more step to do! We didn't implement the fact that exponents are evaluated in reverse!
So we can reverse the expression at the beginning of the exponentiation evaluation tier and then reverse it back at the end. But we need to make sure we exponentiate in the right direction so we need to reverse AGAIN when we evaluate the exponent. This is very ugly. There must be a better way! But for now, this works:

```javascript
function evalExpression(expressionArray) {
   const expression = [...expressionArray];
   for(const tier of evalTiers) {
      if (tier === '**') {
         expression.reverse();
      }
      let nextOperatorIndex = expression.findIndex((o) => tier.includes(o));
      while (nextOperatorIndex !== -1) {
         const operator = expression[nextOperatorIndex];
         const operandOne = expression[nextOperatorIndex - 1];
         const operandTwo = expression[nextOperatorIndex + 1];
         let result;
         if (tier === '**') {
            result = evalOperation(operandTwo, operator, operandOne);
         } else {
            result = evalOperation(...expression.slice(nextOperatorIndex - 1, 3));
         }
         expression.splice(nextOperatorIndex - 1, 3, result);
         nextOperatorIndex = expression.findIndex((o) => tier.includes(o));
      }
      if (tier === '**') {
         expression.reverse();
      }
   }
   return expression[0];
}    
```            

And with that, we're done!
## Adding Parentheses
Parentheses are a little bit tricker than everything else. However, they are a great opportunity to work on recursive problem solving. What we can do is, check to see if our expression has parentheses. If it does, we can take whatever is in the parentheses and call solveExpression on whatever lies between the parentheses. As these subcalls happen, they will eventually get to an expression that has no parentheses, solve that one, and then recurse out of there. So what does this look like? Something like this:

```javascript
function evalExpression(expressionArray) {
   const expression = [...expressionArray];
   let nextParenthesesIndex = expression.findIndex((o) => o === '(' );
   while (nextParenthesesIndex !== -1) {
      // do something
   }
   for(const tier of evalTiers) {
      let nextOperatorIndex = expression.findIndex((o) => tier.includes(o));
      while (nextOperatorIndex !== -1) {
         const operator = expression[nextOperatorIndex];
         const operandOne = expression[nextOperatorIndex - 1];
         const operandTwo = expression[nextOperatorIndex + 1];
         const result = evalOperation(operandOne, operator, operandTwo);
         expression.splice(nextOperatorIndex - 1, 3, result);
         nextOperatorIndex = expression.findIndex((o) => tier.includes(o));
      }
   }
   return expression[0];
}
```         
               
But how do we find the closing parentheses? We can declare a variable called parentheses depth that will keep track of how deep into parentheses we go. We have to do this so we dont think the first closing paren in an expression like `(2*(2+3) + 2` is the one we are looking for.

```javascript
function evalExpression(expressionArray) {
   const expression = [...expressionArray];
   let nextParenthesesIndex = expression.findIndex((o) => o === '(' );
   while (nextParenthesesIndex !== -1) {
      let parenthesesDepth = 1;
      for(let i = nextParenthesesIndex + 1; i < expression.length; i++) {
         if(expression[i] === '(') {
            parenthesesDepth += 1;
         } else if (expression[i] === ')') {
            parenthesesDepth -= 1;
            if (parenthesesDepth === 0) {
               let subexpressionLength = 1 + i - nextParenthesesIndex;
               let subexpression = expression.slice(nextParenthesesIndex + 1, i);
               let subexpressionResult = evalExpression(subexpression);
               expression.splice(nextParenthesesIndex, subexpressionLength, subexpressionResult);
            }
         }
      }
      nextParenthesesIndex = expression.findIndex((o) => o === '(' );
   }
   for(const tier of evalTiers) {
      let nextOperatorIndex = expression.findIndex((o) => tier.includes(o));
      while (nextOperatorIndex !== -1) {
         const operator = expression[nextOperatorIndex];
         const operandOne = expression[nextOperatorIndex - 1];
         const operandTwo = expression[nextOperatorIndex + 1];
         const result = evalOperation(operandOne, operator, operandTwo);
         expression.splice(nextOperatorIndex - 1, 3, result);
         nextOperatorIndex = expression.findIndex((o) => tier.includes(o));
      }
   }
   return expression[0];
} 
```        
               
## Closing Thoughts
I really like any time I get to use recursion in a reasonable way so I enjoyed figuring out how to deal with parentheses. I like this project because I feel like I came away witha deeper understanding of how expressions are evaluated. What a wild ride! You can check out the game calculationster here: 
