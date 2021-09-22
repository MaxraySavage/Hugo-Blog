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
mathjax: true
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
I've found that when you attempt to solve a problem with code you can either spend a few extra minutes defining your terms really well in the beginning or get totally lost halfway through whatever your project is. So let's take a moment to think about what we're doing here. 

### What are we making?
A function written in javascript. The function should take an arithmetic expression as an input and return the result of evaluating the expression. I did some research on how other people have solved this problem but nothing really seemd to fit because I wanted to evaluate my expressions in the same way that a person would. That way I could maintain my requirements that we never get anything but nonnegative integers at any point along the evaluation path.

### What is an arithmetic expression?
We have some choices here. We could represent an arithmetic expression as a string but for the purposes of this post, I will represent expressions with arrays so for example, `$3 + 2$` will be an array with 3 elements like this: `[3, '+', 2]`.

We'll also include parentheses, so we could have something like `$(3 + 2) * 7$` which would look like 
```
['(', 3, '+', 2, ')', '*', 7]
```

So our input will be an array of this form. We will trust that our inputs are well formed for the purposes of this algorithm. 

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

So the multiplication and division happen in the same tier and the addition and subtraction happen in the same tier. Most of these operators are evaluated from left to right. However, exponents are tricky! They are ***[right associative](https://en.wikipedia.org/wiki/Associative_property)*** which means that, within a sequence of exponents, we should evaluate the rightmost pair first. 

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
      - `$(2 + 3) * 4$` would become `$5 * 4$`
      - We must evaluate the deepest nested pair of parentheses first so `$(2 + (15 / 5)) * 4$` would first become `$(2 + 3) * 4$` and then `$5 * 4$`
* **Exponents**: 
   - Exponents are the first tier of operators to evaluate
   - Exponents are denoted in javascript with the binary operator `**`
   - Examples:
      - `$12 + 3^2$` becomes `$12 + 9$`
      - `$12 + 3^2$` in javascript looks like `12 + 3**2`
      - Evaluated right to left (which is weird!) so `2**1**3` becomes `$2^{1^3} \rightarrow 2^1 \rightarrow 2$` which is NOT what would happen if you went left to right. That would be `$2^{1^3} \rightarrow 2^3 \rightarrow 8$`
* **Multiplication and Division**: 
   - Multiplication and division are the second tier of operators to evaluate
   - In javascript, multiplication is represented by `*` and division is represented by `/`
   - We will evaluate multiplication and division from left to right, simultaneously
   - Examples:
      - `$6 + 12 / 4 \times 2 \rightarrow 6 + 3 \times 2 \rightarrow 6 + 6$`
* **Addition and Subtraction**: 
   - Addition and subtraction are the third tier of operators to evaluate
   - In javascript, addition is represented by `+` and subtraction is represented by `-`
   - Addition and subtraction are evaluated from left to right, simultaneously
   - Examples:
      - `$18 - 7 + 4 \rightarrow 11 + 4 \rightarrow 15$`

### Plan of attack
So we know what are inputs and outputs are and understand our general steps we need to take to make this happen. My sense is that parentheses will be best implemented using recursion after we get `EMDAS` done. So we'll start with solving expressions without parentheses and go from there.


## Solution
We will evaluate the expression in three tiers, first `exponentiation`, then `multiplication and division`, then `addition and subtraction`. We'll get to parentheses at the end. 

### Evaluation Tiers
Let's declare an array called evaluationTiers where each index contains all the operands we will look for at each tier. For us this will look like:

```javascript
const evaluationTiers = [
  { operators: ['**'], leftAssociative: false },
  { operators: ['*', '/'], leftAssociative: true },
  { operators: ['+', '-'], leftAssociative: true },
];
```

### Function Skeleton
Now we can start to build our function `evaluateExpression` (I'm using the spread operator `...` here to copy the input array into a new array object)
Remember that, for us, an expression is an array that looks like `[1, '+', 2]`

```javascript
function evalExpression(expressionArray) {
   const expression = [...expressionArray];
   evaluationTiers.forEach((tier) => {
    // do something
  });
  return expression[0];
}   
```

### Finding the Next operator            
WHat do we do within the `forEach` callback ? We have to look through the expression to see if the operation in the current tier is represented. I did this with the findIndex array method as follows, also handling right associativity:

```javascript 
function getNextOperatorIndex(expression, tier) {
  let nextOperatorIndex = expression.findIndex((o) => tier.operators.includes(o));
  // If the operator is left associative or not found at all, we're done
  if (tier.leftAssociative || nextOperatorIndex === -1) {
    return nextOperatorIndex;
  }
  /*
    If the operator is right associative
    we must see if that operator is the first of a sequence of operators within the tier
    If so, we jump forward until we get to the last operator in the sequence
    For example with '2 ** 3 ** 1 + 1 ** 2' we should get the second **
  */
  while (nextOperatorIndex + 2 < expression.length
    && tier.operators.includes(expression[nextOperatorIndex + 2])) {
    nextOperatorIndex += 2;
  }
  return nextOperatorIndex;
}
```

### Fleshing out the Loop
So we know we can find the next operand. We can build a while loop to keep performing operations until all operations in the current tier are performed like so

```javascript
function evaluateExpression(expressionArray) {
  const expression = [...expressionArray];
  evaluationTiers.forEach((tier) => {
    let nextOperatorIndex = getNextOperatorIndex(expression, tier);
    while (nextOperatorIndex !== -1) {
      // Do something
      nextOperatorIndex = getNextOperatorIndex(expression, tier);
    }
  });
  return expression[0];
}   
```

### Evaluating Individual Operations            
Now, what do we do in our while loop? We should probably do something with nextOperatorIndex. We want to evaluate whatever operation that nextOperatorIndex is pointing us to. Let's look at a small example. If we had the array `[1, '+', 3, 'x', 7]` and `nextOperatorIndex` is currently `3`, what should we do? We need to:
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

    case '*':
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

 Now we have to replace the operand and its operators with the result of performing the operation. For this we'll write a new function. I decided to have the function operate directly on the expression it is passed rather than return a modified copy of the input array though that approach would also be possible. We can then use this helper function to replace the array elements from index = nextOperatorIndex - 1, to nextOperatorIndex + 1 with the result of the operation like so: 

```javascript
function performOperation(expression, index) {
  const result = evalOperation(
    ...expression.slice(index - 1, index + 2)
  );
  expression.splice(index - 1, 3, result);
}
```
            
### All togther now
Putting together all the functions we have written we have our expression evaluator! We still can't work with parentheses but other than that we;'re doing great.
```javascript
function evaluateExpression(expressionArray) {
   const expression = [...expressionArray];
   evaluationTiers.forEach((tier) => {
    let nextOperatorIndex = getNextOperatorIndex(expression, tier);
    while (nextOperatorIndex !== -1) {
      performOperation(expression, nextOperatorIndex)
      nextOperatorIndex = getNextOperatorIndex(expression, tier);
    }
  });
   return expression[0];
}    
```               

And this will work for any expressions that don't contain parentheses!
## Adding Parentheses
Parentheses are a little bit tricker than everything else. However, they are a great opportunity to work on recursive problem solving. What we can do is, check to see if our expression has parentheses. If it does, we can take whatever is in the parentheses and call solveExpression on whatever lies between the parentheses. As these subcalls happen, they will eventually get to an expression that has no parentheses, solve that one, and then recurse out of there. So what does this look like? Something like this:

### Building out another loop
```javascript
function evaluateExpression(expressionArray) {
  const expression = [...expressionArray];
  let openingParenthesesIndex = expression.findIndex((o) => o === '(');
  while (openingParenthesesIndex !== -1) {
    // do something
    openingParenthesesIndex = expression.findIndex((o) => o === '(');
  }
  evaluationTiers.forEach((tier) => {
    let nextOperatorIndex = getNextOperatorIndex(expression, tier);
    while (nextOperatorIndex !== -1) {
      const result = evalOperation(
        ...expression.slice(nextOperatorIndex - 1, nextOperatorIndex + 2)
      );
      expression.splice(nextOperatorIndex - 1, 3, result);
      nextOperatorIndex = getNextOperatorIndex(expression, tier);
    }
  });
  return expression[0];
}
```         
### Where is the closing parentheses?               
But how do we find the closing parentheses? We can declare a variable called parentheses depth that will keep track of how deep into parentheses we go. We have to do this so we dont think the first closing paren in an expression like `$(2*(2+3) + 2)$` is the one we are looking for and try and treat `$2*(2+3$` as a subexpression.

```javascript
function findClosingParenthesesIndex(expression, openingParenthesesIndex) {
  let parenthesesDepth = 1;
  for (let i = openingParenthesesIndex + 1; i < expression.length; i += 1) {
    if (expression[i] === '(') {
      parenthesesDepth += 1;
    } else if (expression[i] === ')') {
      parenthesesDepth -= 1;
      if (parenthesesDepth === 0) {
        return i;
      }
    }
  }
  return 'Error: no closing parentheses found';
}
```        
### The whole thing  
```javascript
function evaluateExpression(expressionArray) {
  const expression = [...expressionArray];
  let openingParenthesesIndex = expression.findIndex((o) => o === '(');
  while (openingParenthesesIndex !== -1) {
    const subexpression = getSubexpression(expression, openingParenthesesIndex);
    // recurse into subexpression
    const subexpressionResult = evaluateExpression(subexpression);
    // we need to add two to length
    // because subexpression doesn't include its bounding parentheses
    expression.splice(
      openingParenthesesIndex,
      subexpression.length + 2,
      subexpressionResult,
    );
    openingParenthesesIndex = expression.findIndex((o) => o === '(');
  }
  evaluationTiers.forEach((tier) => {
    let nextOperatorIndex = getNextOperatorIndex(expression, tier);
    while (nextOperatorIndex !== -1) {
      performOperation(expression, nextOperatorIndex);
      nextOperatorIndex = getNextOperatorIndex(expression, tier);
    }
  });
  return expression[0];
}
```

## Formatting a String into an Expression Array
Just for fun, I wrote a method to take in a string and format it like an expression array we've been using. I like a chance to practice using regex, call me crazy.

```javascript
function parseExpression(expressionString) {
  const expressionArray = Array.from(
    expressionString.matchAll(
      /-?[0-9]+\.?[0-9]*|\*{2}|\*{1}|\/|\+|-|\(|\)/g
    )
  ).map((v) => v[0]);
  return expressionArray.map((term) => {
    if (Number.isNaN(parseFloat(term))) {
      return term;
    }
    return parseFloat(term);
  });
}
```

## Closing Thoughts
I feel like writing this forced me to refactor my code to be easir to present in this blog and write more helper functions so that the main function would be less bloated looking.

I really like any time I get to use recursion in a reasonable way so I enjoyed figuring out how to deal with parentheses. I like this project because I feel like I came away witha deeper understanding of how expressions are evaluated. What a wild ride! You can check out the game calculationster here: 
