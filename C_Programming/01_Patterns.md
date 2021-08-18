# An Introduction to Patterns

> How do you know what to write in C? I have a rough understanding of what my program should do, how can I teach it to the computer?

This is a question that I have been asked a lot in the last few weeks. For a while I didn't have a proper answer to it, but now I think that I do.

## Thinking with Patterns
	
The key behind being able to tell the computer what to do is to not think of a C program as a bunch of instructions that are being executed line-by-line in a top-to-bottom manner, but rather as a collection of so-called _Patterns_. A pattern is a single, standalone piece of C code - often consisting of only a few (though sometimes more) lines - that serves exactly one purpose. When you're writing a program, your task is to select the correct patterns for your program, put them in the right order and connect them in a meaningful fashion.

Think of it as a "fill in the blanks"-text ("Lückentext" in German), where you have a basic template that serves a particular purpose. Filling in the blanks with other patterns gives you the ability to further customize its behavior.

The idea behind patterns is to provide you with simple, easy-to-replace "blocks" of code that can be treated as units on their own, giving you the opportunity to change or improve them without accidentally damaging other parts of your code in the process, therefore avoiding errors.

## The Layout of a Function

Most functions that you will encounter tend to share some similarities. Let's call them the _Master Pattern_, as this will always be the pattern you'll start with when you're confronted with the task of creating a new function from scratch.

Let's have a look at the `ft_bzero` function:

```c
void ft_bzero(void* s, size_t n)
{
    char*   bytes;
    size_t  i;
	
    if (s == NULL)
        return;
    bytes = (char*) s;
    i = 0;
    while (i < n)
    {
        bytes[i] = 0;
        i = i + 1;
    }
    return;	/* NOTE: This line is optional */
}
```

This function takes a pointer to a chunk of memory together with its size, and fills every byte within that region with the value `0`.

Let's now subdivide this function into five sections to see how it works:

```c
void ft_bzero(void* s, size_t n)
{
    /* 1st: Variable Declarations */
    char*   bytes;
    size_t  i;
    
    /* 2nd: Guards against Segmentation Faults */
    if (s == NULL)
        return;
    
    /* 3rd: Initialization of Variables */
    bytes = (char*) s;
    
    /* 4th: The Body of your Function (in this case, a while loop) */
    i = 0;
    while (i < n)
    {
        bytes[i] = 0;
        i = i + 1;
    }
    
    /* 5th: Wrapping Things Up */
    return;
}
```

As you can see, every section in this function has a specific purpose.

 - Section 1 lists all variables that are going to be used within all following sections of the function. Just like the list of ingredients in a cooking recipe, it comes before anything else. _If you want to add a variable to your function, do it here._
 - Section 2 protects your code from segmentation faults, invalid memory reads, dangerous parameter values or any other state where your following sections could crash. _You'll find mostly `if` statements, `NULL` checks and `return` statements here._
 - Section 3 initializes all of your variables with a meaningful value. In some cases it creates new data structures by calling other functions (e.g. `malloc`). _Be aware that in order to avoid memory leaks, everything you're creating in this section (and all following sections) will either have to be destroyed in section 5 or passed back to the caller of the function._
 - Section 4 is where the fun begins. In this section you are only limited by your imagination. _However, it is strongly recommended that you insert **only one** high-level pattern here to avoid confusion._
 - Section 5 has the task of freeing all data structures that you have created in the sections before. _The only exception to this are memory blocks that are going to be `return`-ed to the caller of our function._

Sections 1, 2 and 3 have the purpose of setting everything up so that section 4 can run. Section 4 then executes a set of patterns that do the work that `ft_bzero` is responsible for, and section 5 terminates the function accordingly.

As stated previously, you'll find these five sections in almost every function that you're going to encounter.

