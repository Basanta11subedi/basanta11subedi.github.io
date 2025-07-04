---
title: "Learning Clarity"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

# **Clarity Basics**

**An interactive introduction to the Clarity language.**

Clarity features LISP-like syntax. That means that you will see a lot of parentheses. Inside these parentheses you will find symbols, phrases, values, and more parentheses. People that are new to LISP-like languages may feel intimidated as it looks very foreign compared to both natural languages as many other programming languages.

A way to conceptualize Clarity code is to think of lists inside lists. (The technical term for these kinds of expressions is “S-expression“.) Expressions do not differentiate between data and code, that makes things a bit easier! Here is an example of a Clarity expression (hint: click the *play* button or copy and paste it into the REPL):

Did you guess correctly what the expression evaluates to? Even though it might look a little strange for those who are not used to *Polish notation* in math, it is easy to make a reasonable assumption. The **+** symbol defines an operation (in this case addition), and the **4** and the **5** are inputs. The end result: **9**.

Such expressions always follow the same basic pattern: an opening round brace **(**, followed by a function name, optionally followed by a number of input parameters, and closed by a closing round brace **)**. The different parts inside the braces are separated by whitespace.

The **+** symbol in the example above really has no special significance. It is just a function name. Here is another example:

Is it starting to make sense? **concat** is the function name and it is provided with two strings as inputs. “Concat” is short for *concatenate*. Thus: it glues two strings together to form the classic **"Hello World!"**.

You might be able to intuit how more complex expressions go together at this point. Take the following calculation for example: **4 × (15 + 10)**. In Clarity, it looks like this:

The above is still pretty legible, but it is obvious that expressions with multiple levels of nesting become hard to read. Whitespace is only used to delineate symbols. It means that we can use tabs, spaces, and newlines to make Clarity code more readable.

As you start writing more Clarity code, you will discover which formatting is most comfortable for you. Looking at how other developers format their code also helps.

Finally, it is possible to add freeform commentary to your code. Never be sparse with your comments, especially if you are working on a project with multiple people! Comments are prefixed by a double semi-colon (**;;**) and can appear on their own line or after an expression. They are also useful to temporarily deactivate some code during the development process.

Remember that Clarity contracts are committed to the chain as-is. Comments therefore increase the contract size and anything *creative* you put in there will be viewable by anyone inspecting the code.