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
One of my favorite expressions is the aphorism 

In order to build this featuree, a function that solves an expression with PEMDAS, we have to define what an expression will look like and what PEMDAS is. For this article we will only consider a rather small subset of possible mathematical expressions. Specifically, an expression will be an array of alternating numbers and binary operators. For our game we did not include parentheses in our problems, we will consider their addition ina future article. So an example of an expression would be [1, '+', 5] and we would expect this to equal 6 when solved. Now PEMDAS is a convention of written mathematics and it is not the end all. However it is a convention that can be challenging to learn so we want to help people learn it in our game. For our purposes, PEMDAS will mean the following:

Pemdas Diagram
Pemdas actually has only 4 steps
Parentheses Evaluate terms im parentheses first (we didn't do this inour project)
Exponents Exponents, denoted in javascript with the binary operator **. Under our current definition of an expression, we could have something like [2, '**', 3] and we would expect that to equal 8.
Multiplication and Division We will evaluate multiplication and division from left to right, simultaneously. While javascript uses '*' as the multiplication operator, in our code we used 'x' because that is closer to how arithmetic is taught. In our code, an expression using multiplication and division might look like [54, '/', 9, 'x', 7] which we would evalute from left to right, first getting 54 / 9 = 6 and then 6 x 7 = 42. Notice that if we did multiplication and then division we would instead end up with 54 / 42 because we did the multiplication first, leaving us with the fraction 9/7.
Addition and Subtraction We then evaluate addition and subtraction left to right just like multiplication and division. We can also see the same potential for misunderstanding with the expression [20, '-', 5, '+' 32]. Done our way, this will result in 20 - 5 = 15 and then 15 + 32 = 47 whereas if we did addition then subtraction we'd end up with 20 - 37 = -17
So we can see that evaluating an expression with require a tiered approach. First we will look for parentheses, if we don't find them, we will look for an exponent, then multiplication or division, then addition or subtraction. It is my feeling that implementing parentheses is best done recursively after completing a solver that works on the later tiers so let's do that first.

## Solution
We will evaluate the expression in three tiers, first exponentiation, then multiplication and division, then addition and subtraction. Let's declare an array called evalTiers where each index contains all the operands we will look for at each tier. For us this will look something like

```javascript
const evalTiers = [
   '**',
   ['x', '/'],
   ['+', '-']
];
```

Now we can start to build our function evalExpression (I'm using the spread operator ... here to avoid changing the input array)

```javascript
function evalExpression(expressionArray) {
   const expression = [...expressionArray];
   for(const tierOperands of evalTiers) {
      // do something
   }
}   
```
            
What's the something though? We have to look through the expression to see if the operation in the current tier is represented. I did this with the findIndex array method as follows:

```let nextOperatorIndex = expression.findIndex((o) => tierOperands.includes(o));```
            
This will return the index of the first item in the expression array that is included in the array of operands in the current tier or return -1 if none are found. We want to continue evaluation on the array until there are no more operands in our current tier. Therefore we can implement a while loop where we do something to the expression array while our nextOperatorIndex is not 0. That would look like this:

```javascript
function evalExpression(expressionArray) {
   const expression = [...expressionArray];
   for(const tierOperands of evalTiers) {
      let nextOperatorIndex = expression.findIndex((o) => tierOperands.includes(o));
      while (nextOperatorIndex !== -1) {
         //do something
         nextOperatorIndex = expression.findIndex((o) => tierOperands.includes(o));
      }
   }
}   
```
            
So in our while loop we have to do something with nextOperatorIndex. We want to evaluate whatever operation that nextOperatorIndex is pointing us to. Let's look at a small example. If we had the array [1, '+', 3, 'x', 7] and nextoperatorIndex is currently 3, what should we do? We need to look at the operator at index 3, see if it tells us to multiply, divide, add, etc. and then do that to the things at index 2 and 4, and then replaces all 3 things with the result. So in this case we replace the [3, 'x', 7] portion of the array with 21 leaving us with [1, '+', 21]. How do we do this with code? First we can write a helper function to handle figuring out what operation to do and then doing it.

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
        return 'Error, should be a valid operator';
    }
}
```
            
We can then use this helper function to replace the array elements from index = nextOperatorIndex - 1, to nextOperatorIndex + 1 with the result of the operation like so:

```javascript
function evalExpression(expressionArray) {
   const expression = [...expressionArray];
   for(const tierOperands of evalTiers) {
      let nextOperatorIndex = expression.findIndex((o) => tierOperands.includes(o));
      while (nextOperatorIndex !== -1) {
         const operator = expression[nextOperatorIndex];
         const operandOne = expression[nextOperatorIndex - 1];
         const operandTwo = expression[nextOperatorIndex + 1];
         const result = evalOperation(operandOne, operator, operandTwo);
         expression.splice(nextOperatorIndex - 1, 3, result);
         nextOperatorIndex = expression.findIndex((o) => tierOperands.includes(o));
      }
   }
   return expression[0];
}    
```     
            
And with that, we're done!
### Parentheses
Parentheses are a little bit tricker than everything else. However, they are a great opportunity to work on recursive problem solving. What we can do is, check to see if our expression has parentheses. If it does, we can take whatever is in the parentheses and call solveExpression on whatever lies between the parentheses. As these subcalls happen, they will eventually get to an expression that has no parentheses, solve that one, and then recurse out of there. So what does this look like? Something like this:

```javascript
function evalExpression(expressionArray) {
   const expression = [...expressionArray];
   let nextParenthesesIndex = expression.findIndex((o) => o === '(' );
   while (nextParenthesesIndex !== -1) {
      // do something
   }
   for(const tierOperands of evalTiers) {
      let nextOperatorIndex = expression.findIndex((o) => tierOperands.includes(o));
      while (nextOperatorIndex !== -1) {
         const operator = expression[nextOperatorIndex];
         const operandOne = expression[nextOperatorIndex - 1];
         const operandTwo = expression[nextOperatorIndex + 1];
         const result = evalOperation(operandOne, operator, operandTwo);
         expression.splice(nextOperatorIndex - 1, 3, result);
         nextOperatorIndex = expression.findIndex((o) => tierOperands.includes(o));
      }
   }
   return expression[0];
}
```         
               
But how do we find the closing parentheses? We can declare a variable called parentheses depth that will keep track of how deep into parentheses we go. We have to do this so we dont think the first closing paren in an expression like (2*(2+3) + 2) is the one we are looking for.

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
               console.log(expression);
            }
         }
      }
      nextParenthesesIndex = expression.findIndex((o) => o === '(' );
   }
   for(const tierOperands of evalTiers) {
      let nextOperatorIndex = expression.findIndex((o) => tierOperands.includes(o));
      while (nextOperatorIndex !== -1) {
         const operator = expression[nextOperatorIndex];
         const operandOne = expression[nextOperatorIndex - 1];
         const operandTwo = expression[nextOperatorIndex + 1];
         const result = evalOperation(operandOne, operator, operandTwo);
         expression.splice(nextOperatorIndex - 1, 3, result);
         nextOperatorIndex = expression.findIndex((o) => tierOperands.includes(o));
      }
   }
   return expression[0];
} 
```        
               
## Closing Thoughts
I really like any time I get to use recursion in a reasonable way so I enjoyed figuring out how to deal with parentheses. I like this project because I feel like I came away witha deeper understanding of how expressions are evaluated. What a wild ride! You can check out the game calculationster here: 
