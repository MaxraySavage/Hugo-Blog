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
searchHidden: false
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
    URL: "https://github.com/MaxraySavage/Hugo-Blog/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

## Intro
A few months ago I made an edutainment game for The Odin Project's first game jam. My team and I had decided to build a game for practicing basic math. I took on the task of generating the problems as appropriate for each level of the game. It was hard! Generating random arithmetic expressions within certain difficulty constraints was *not* simple. 

For example, one difficulty constraint we had was that our audience should not have to even consider the existence of negative numbers. At first that seemed easy, just make sure the answer isn't a negative number, right? But what about something like `$5 - 2 \times 5 + 20$`. The answer is `$15$` but along the way you end up with `$-15 + 20$`. Bad!

In order to enforce our constraints, I wrote a script to evaluate expressions step by step so I could tell if my problems conformed to my difficulty constraints. Which meant I had to get very friendly with ***PEMDAS***.

I got interested in the idea of writing code to solve a math problem in the same order and with the same techniques as how a person would do it. This turned out to be pretty tricky but I had fun doing it so I decided to make it into my first *ever* blog post! 

## Define the Problem
When you attempt to solve a problem with code you can either spend a few extra minutes defining your terms really well in the beginning or get totally lost halfway through whatever your project is. So let's take a moment to think about what we're actually doing here. 

### What are we making?
A function written in javascript. The function should take an arithmetic expression as an input and return the result of evaluating the expression. It should resolve individual operations in the order dictated by **PEMDAS**.

I did some research on how other people have solved this problem but nothing really seemed to fit my requirements because I wanted to evaluate my expressions in the same way that a person would without getting into postfix notation. 

### What is an arithmetic expression?
So what really *is* an arithmetic expression? If you dig deeply enough you end up in philosophy. Instead let's allow for a subset of arithmetic expressions specifically
- there will be numbers
- there will be binary operators (operators that take two numbers and spit out a number)
- there will be parentheses
- the expression will be well formed (nothing like `$2 + (((+ *  - 4 23 4.3$`)

We need to decide on the expected format for our input. So, what is an arithmetic expression ***to us***. We have some choices here. We could represent an arithmetic expression as a string like `"2 + 7 x 4"`. There are benefits to this. For our game I ended up using this method mainly so that other people using my code to generate expressions wouldn't have to deal with formatting and could just print problems to the player. This led to every function I wrote having a preamble splitting the string into an array at the beginning and then rejoining everything at the end. For this article I decided to instead expect an expression to be an array. So for example the expression `$3 + 2$` will be an array with 3 elements like this: `[3, '+', 2]`. We can also include parentheses, so we might have something like `$(3 + 2) \times 7$` which would look like `['(', 3, '+', 2, ')', 'x', 7]`

So our input will be an array of this form. We will trust that our inputs are well formed for the purposes of this algorithm. So no missing parentheses, chains of addition signs or anything else. Not very realistic but maybe validating that could be another article for you to enjoy! Let's save some fun for later.

### How do you evaluate an expression?
There's a relatively ubiquitous algorithm for evaluating arithmetic expressions called PEMDAS. This algorithm tends to be pretty tricky as evidenced by the biannual viral tweet where the internet argues about the right way to evaluate some expression or another (usually the confusion actually comes from an omitted multiplication sign!). PEMDAS is not only an algorithm, it is also an acronym! It stands for 

- Parentheses
- Exponentiation
- Multiplication
- Division
- Addition
- Subtraction 

The thing is, PEMDAS actually only has 4 tiers of evaluation, not the 6 you might expect. 

Let's consult this handy diagram: 

![PEMDAS diagram](/images/pemdas-diagram.png)

So the multiplication and division happen in the same tier and the addition and subtraction happen in the same tier. Most of these operators are evaluated from left to right. However, exponents are tricky! Exponents are ***[right associative](https://en.wikipedia.org/wiki/Associative_property)*** which means that, within a sequence of exponents, we should evaluate the rightmost pair first. 

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
      - `$12 + 3^2$` in javascript looks like `12 + 3 ** 2`
      - Evaluated right to left (which feels weird!) so `2 ** 1 ** 3` becomes `$2^{1^3} \rightarrow 2^1 \rightarrow 2$` which is NOT what would happen if you went left to right. That would be `$2^{1^3} \rightarrow 2^3 \rightarrow 8$`
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


## Evaluating Operations
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
function evaluateAllOperations(expressionArray) {
   const expression = [...expressionArray];
   evaluationTiers.forEach((tier) => {
    // do something
  });
  return expression[0];
}   
```

### Find the Next Operator            
What do we do within the `forEach` callback ? 
We need to look through the expression to see if any operations in the current tier are represented and we need to figure out which of those operations we should do first. 

I did this with the `findIndex` array method because I could search with a custom callback function instead of looking for just one operator at a time. This function finds the index of the next operation that should be executed within a given tier.

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

The way we jump forward by two's for right associative operators may seem weird but we need to remember that, because of how we will handle parentheses later, this function will never actually be used on an expression with parentheses.

### Fleshing Out the Loop
So we know we can find the next operand. We can build a while loop to keep performing operations until all operations in the current tier are performed like so

```javascript
function evaluateAllOperations(expressionArray) {
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
Now, what do we do in our while loop? Well, we should probably do something with `nextOperatorIndex`. We want to evaluate whatever operation that `nextOperatorIndex` is pointing us to. Let's look at a small example. If we had the array `[1, '+', 3, 'x', 7]` and `nextOperatorIndex` is currently `3`, what should we do? We need to:
- Look at the operator at index `3`
- See if it tells us to multiply, divide, add, etc.
- Get the result of performing the operation on the numbers at index `2` and `4` 
- Then replace the items at index `2`, `3` and `4` with the result.

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

 Now we have to replace the operand and its operators with the result of performing the operation. For this we'll write a new function. I decided to have the function operate directly on the expression that it is passed rather than return a modified copy of the input array.

```javascript
function performOperation(expression, index) {
  const result = evalOperation(
    ...expression.slice(index - 1, index + 2)
  );
  expression.splice(index - 1, 3, result);
}
```
            
### Putting it Together
Putting together all the functions we have written we have our expression evaluator! We still can't work with parentheses but other than that we're doing great.
```javascript
function evaluateAllOperations(expressionArray) {
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

## Adding Parentheses
To me, dealing with parentheses felt little bit tricker than everything else. I did feel like they were a great opportunity to work on recursive problem solving. What we can do is, check to see if our expression has parentheses. If it does, we can take whatever is within the parentheses and call `evaluateExpression` on whatever lies between the parentheses. As these subcalls happen, they will eventually get to an expression that has no parentheses, solve that one, and then recurse out of there. So what does this look like? Something like this:

```javascript
function evaluateExpression(expressionArray) {
  const expression = [...expressionArray];
  let openingParenthesesIndex = expression.findIndex((o) => o === '(');
  while (openingParenthesesIndex !== -1) {
    // do something
    openingParenthesesIndex = expression.findIndex((o) => o === '(');
  }
  return evaluateAllOperations(expression);
}
```         
### Closing Parentheses               
But how do we find the closing parentheses? We can declare a variable called `parenthesesDepth` that will keep track of how deep into parentheses we go. We have to do this so we dont think the first closing parentheses in an expression like `$(2*(2+3) + 2)$` is the closing parentheses we are looking for and try and treat `$2*(2+3$` as a subexpression. If a parentheses isn't found we return `-1` to mimic the behavior of `findIndex`.

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
  return -1;
}
```  

We'll also need a little utility function to actually create the array representing our subexpression. We can use our `findClosingParentheses` function to make writing our utility function somewhat trivial. Notice that because we are adding one to `openingParenthesesIndex` and the `slice` array method excludes the item located at the ending index, we will not be including the subexpression's surrounding parentheses.  

```javascript
function getSubexpression(expression, openingParenthesesIndex) {
  const closingParenthesesIndex = findClosingParenthesesIndex(
    expression,
    openingParenthesesIndex,
  );
  const subexpression = expression.slice(
    openingParenthesesIndex + 1,
    closingParenthesesIndex,
  );
  return subexpression;
}
```
I really like pulling stuff like this out into its own function because, when you use long descriptive variable names (something else I enjoy) the code can start to look very crowded. 
###   
We can now effectively grab our subexpressions out of the main expression! Sometimes good recursion feels like cheating. We can evaluate our subexpression by calling our `evaluateExpression`. This will eventually recurse to a layer with no parentheses. In that layer, the function will finally get to its second half where it actually performs operations. Once it evaluates that subexpression on the final layer, we will replace the subexpression with the result which we do with `splice`. And here I present the result of our hard labor, the final `evaluateExpression` function!

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
  return evaluateAllOperations(expression);
}
```

## Closing Thoughts
I feel like writing this forced me to refactor my code to be easier to present in this blog as well as  write more helper functions so that the main function would be less bloated looking.

I really like any time I get to use recursion in a reasonable way so I enjoyed figuring out how to deal with parentheses. I like this project because I feel like I came away with a deeper understanding of what actually goes into evaluating an expression. What a wild ride! Thanks for reading! You can check out the game Calculationster that inspired all this work here: {{<calculationster>}}
