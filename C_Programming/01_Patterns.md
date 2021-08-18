# An Introduction to Patterns

> How do you know what to write in C? I have a rough understanding of what my program should do, how can I teach it to the computer?

This is a question that I have been asked a lot in the last few weeks. For a while I didn't have a proper answer to it, but now I think that I do.

## Thinking in Patterns
	
The key behind being able to tell the computer what to do is to not think of a C program as a bunch of instructions that are being executed line-by-line in a top-to-bottom manner, but rather as a collection of so-called _Patterns_. A pattern is a single, standalone piece of C code - often consisting of only a few (though sometimes more) lines - that serves exactly one purpose. When you're writing a program, your task is to select the correct patterns for your program, put them in the right order and connect them in a meaningful fashion.

Think of it as a "fill in the blanks"-text ("Lückentext" in German), where you have a basic template that serves a particular purpose. Filling in the blanks with other patterns gives you the ability to further customize its behavior.

The idea behind patterns is to provide you with simple, easy-to-replace "blocks" of code that can be treated as units on their own, giving you the opportunity to change or improve them without accidentally damaging other parts of your code in the process, therefore avoiding errors.

