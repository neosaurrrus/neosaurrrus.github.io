---
layout: post
title:      "Smartening myself up with Markdown"
date:       2019-08-20 17:21:26 +0000
permalink:  smartening_myself_up_with_markdown
---


As I have been writing blogs I have been interested in optimising the process of transferring my output, **words and code** into the clearest manner. *It turns out that simply writing everything into OneNote and pasting into Medium gets a little messy*

## Starting out

My main code editor is Visual Studio Code. It handily allows you to preview the output using **Ctrl/Option + Shift + V**. The markdown guide they have is pretty nice actually so check it out:

[Markdown and VS Code](https://code.visualstudio.com/docs/languages/markdown)

The main reason I learnt markdown is to show my code more clearly, this is easy to do with backticks:

The escape character in Markdown is the backslash (\):

\*\* Double asterisks normally makes the text Bold unless you use backticks to escape them \*\*


## Paragraphs and Text

Its important to make sure we leave a clear line between our paragraphs else Markdown will just treat it as part of the same paragraph.
This is a Naughty Paragraph that follows on as I deliberately missed a line.

This is a good paragraph, it will get treats.

One asterisk or underscore will make something *italic*

Two asterisks or underscore will make something **bold**

Using Double Tilde will ~~strikethrough the text~~

## Headings

There are two types of heading, first we can use # to denote a H1, ## for H2 and so forth... It is clear enough to know to only apply at the start of a line.

### The other kind of heading (This is a H3 btw)

You can also do an underliney H1 or H2 by underlining with either = or - (equals or minus) as shown here:

Heading 1
=========
Heading 2
---------

## Code Blocks

Here is some code, I can just indent it:

    let robots = "Terminator"
    console.log(robots + " are cool")

However instead, using three backticks at the start and end will tell that it is code, saying 'js' will even try and highlight accordingly. It works for other languages too.

```js
let robots = "R2D2"
console.log(robots + " are very cool");
```

If you just want to include a code snippet inline with other text you can just use a backtick to `console.log("show your")` code

A special case is using backticks with 'diff'. This will allow you to show changes by putting plus and minusses at the start of each line. That probably doesnt make sense but here is an example:

```diff
-let favouritePet = "dogs";
+let favouritePet = "giraffes";
```

## Tables

Tables are possible buts its a pain. Not every markdown parser supports it. You have to use `|`  to define columns:

    | Country | Continent |
    | France | Europe |

## Images

This is a ! so ...

`! [alt-text]`(url)`

Will do the job nicely.



## Add to lists

While I am talking about lists, some places support checkboxes using square brackets. To fill a checkbox use an `x` inbetween like so:

* [ ] Feed cats
* [x] Feed bears

Not really list related but Github also allows use of `#` and `@`  to reference issues and people. 

# Using Markdown in Medium

To import my markdown into medium I use the really nice service [Markdown to Medium](markdowntomedium.com) which spits out markdown to be imported into Medium.

Not everything works as it should though, code blocks are given in gist url form and require hitting enter after them for medium to understand it needs to show the code.

Nested Listed often tend to break.

Overall though, the 5 mins it takes to sort out is far better than trying to spend time getting it correct using purely medium.

## Conclusion

I hope that gets you started with Markdown. To learn what I have just learnt in a better way I can recommend [Wes Bos' Mastering Markdown course](www.masteringmarkdown.com)

You cant be expected to learn everything so I would recommend having a cheatsheet bookmarked. Google your favourite.
