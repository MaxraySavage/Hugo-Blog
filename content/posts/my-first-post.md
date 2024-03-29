---
title: "Let's Learn about Markdown!"
date: 2020-12-29T13:54:10-05:00
draft: true
toc: false
images:
tags:
  - info
  - learning
  - the joy of discovery
  - Hugo
  - Markdown
logoText: "cat markdown-text.md"
---

###### {{<param title>}}
## Headers
You can represent a header with a # sign. The more #'s, the smaller the header.
###### ###### There
##### ##### are
#### #### six
### ### different
## ## sizes
# # for
headers. These are good for big titles and starting new sections in your article! Using this will encourage you to build good outlines for your writing.

## Italics
Italizcs let your reader know that they should read the text in an italian accent. This is called "italicizing." To italicize text, wrap it in underscores, like so _ _That's a spicy meatball!_ _

## Bold
Wrapping text in double asterisks will embolden your language to jump off the page. Be careful with formatting or you may end up with a dis***aster** on your hands.

## Lists
YOu can list your thoughts in order or not.

### A numbered list
An ordered list is as easy as starting each line with a number followed by a period. So, for example, you would follow these steps:
1. Press enter to start a new line in your markdown document
2. Type the number 1
3. Press the period key 
34. Press space
5. Type the rest of the line
6. Check your numbering
7. Repeat, incrementing the number at the start of each new line 
1. Though you can start each line with 1. for some reason
1. **How bold**

### A bullet point list
Start each line with a little dash '-' and your face will go from looking like '-' to 'o' as your jaw drops at the sheer power of the markdown syntax. 
To recap:
- 😑
- 😰

## Putting lines through stuff
Sometimes it can be important to put a line through your text when you want to let people know that you ~~never~~ too make mistakes and say the ~~right~~ wrong thing.

## Hyperlinks
If you'd like to log your readers out of their gmail account you can make a link that says something like, please [click here!](https://mail.google.com/mail/logout) but the link will actually go to "https://mail.google.com/mail/logout"

Do this by wrapping your link text in brackets like so [[A Link]] and then follow the end bracket with a parenthetical containing the destination URL. 
Like so: [[Log me out of gmal](https://mail.google.com/mail/logout)](https://mail.google.com/mail/logout)

## Images
> ![alt text](Image URL "Hover text")

## Tables 
```
| Tables  | Don't   | Seem     |
| ------- |:-------:| -------- |
| very    | fun     |          |
| Seems   | very    | annoying |
```

| Tables  | Don't   | Seem     |
| ------- |:-------:| -------- |
| very    | fun     |          |
| Seems   | very    | annoying |

## Quotes
``` > Quotes, creates a quote ```
> Quotes


## Code
Let's see how ``` in-line code ``` looks.
What if I make it in-line ```python python``` code?



```python {linenos=table,hl_lines=[2],linenostart=19}
x = 12
x = x%5

def squared(num):
    return num*num

y = squared(x)
```

```python
this_block = "doesn't have a language set for it"
```

{{< highlight python "linenos=table,hl_lines=3,linenostart=199" >}}
x = 12
x = x%5

def squared(num):
    return num*num

y = squared(x)
{{< / highlight >}}

```html
<body>
  <h1> Hello! </h1>
</body>
```

```bash
ls -al .
cat /var/log | grep login
```

## Gists

You can embed Github Gists but you get the WHOLE THING. Careful.
{{< gist MaxraySavage a23b413c5ae112e92ffb>}}


## Youtube Embed
{{< youtube id="-1wirGsdK4U" title="Blawan - What You Do With What You Have" >}}

## Footnotes
Will this[^1] make a footnote?

[^1]: Is this the first footnote?

What does the big[^bignote] footnote do?

[^bignote]: How much can I put in this footnote? Quite a bit!
    ```python
    for each footnote in footnotes:
      print(f"{footnote} ")
    ```
  