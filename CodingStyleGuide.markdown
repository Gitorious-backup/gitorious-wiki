Gitorious Coding Style Guide
=============================

As Gitorious is a codebase of decent size (by Ruby standards at
least), we try to adhere to a few guidelines to keep the codebase
satisfying to look at. This document outlines the main points we try
to follow.

The one thing to take with you from this document is that there's some
level of flexibility in the "rules" outlined below, but most important
thing is that your code _should look good_ and be easily _readable_
and _understandable_ for every one else. The Gitorious codebase has
its dark corners, possibly filled with dragons, but we try to improve
things as we see them, "leave the source in a better state than you
found it" is solid advice.

Pay attention to what you commit; always review pending changes with
`git diff --staged` and look for things that violates the outlines
below (git is being helpful and highlights some things, such as
trailing whitespace).

In general, try to follow the style of the existing code, and pay
attention to how the code you're writing _actually looks_, not just
how it works.


General Guidelines
--------------------

* Lines should stay below 80 chars or so.
* No tabs.
* Don't get too clever.
* No trailing whitespace


Ruby
--------------------

Generally the points in Christan Neukirchens
[RUBY-STYLE](http://github.com/chneukirchen/styleguide/raw/master/RUBY-STYLE)
apply, mainly:


Javascript
--------------------

* 4 spaces, no tabs
* 80 chars or less line length
* Local variables must use the `var` keyword
* Use (one) blank line to group statements together where suited
* Always use a single space after a keyword, and before a curly brace.
* Curly braces goes on the same line

    // Wrong
    function foo (arg){
    // Wrong
    function foo(arg)
    {
    // Correct
    function foo(arg) {

The same applies for conditionals:

    // Wrong
    if (arg){
    // Wrong
    if(arg)
    {
    // Correct
    if foo(arg) {
        ...
    } else {
        ...
    }

One possible exception to the above is if the conditionals argument is
wrapped on multiple lines, the brace can be placed on a new line to
ease readability of the conditional body:

     if ((foo && barIsJustAWordUsedforDemonstrations) ||
         kittensAreFluffy && doesNotSayWoof)
     {
         ...
     }

However, long boolean expressions should be avoided in the first place.


CSS
--------------------

* four space indent
* no tabs
* multiple selectors on seperate lines, unless singleworded

    /* wrong */
    \#foo #bar div.foo, #baz p#quux span.foo {
        ...
    }
    /* Correct */
    \#foo #bar div.foo,
    \#baz p#quux span.foo {
        ...
    }
* Opening brace on the same line as the selector


HTML
--------------------

* 2 spaces indendation
* No tabs
* Prefer to indent deep nesting on a newline+indent, so the structure
  is easier to follow
