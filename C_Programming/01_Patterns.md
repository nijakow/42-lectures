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
 - Section 2 protects your code from segmentation faults, invalid memory reads, dangerous parameter values or any other state where your following sections could crash. _Most patterns in this section are [Guard Patterns](#guard-patterns)._
 - Section 3 initializes all of your variables with a meaningful value by using [Initialization Patterns](#initialization-patterns). In some cases it creates new data structures by calling other functions (e.g. `malloc`). _Be aware that in order to avoid memory leaks, everything you're creating in this section (and all following sections) will either have to be destroyed in section 5 or passed back to the caller of the function._
 - Section 4 is where the fun begins. In this section you are only limited by your imagination. _However, it is strongly recommended that you insert **only one** [High-Level Pattern](#high-level-patterns) here to avoid confusion._
 - Section 5 has the task of freeing all data structures that you have created in the sections before. _The only exception to this are memory blocks that are going to be `return`-ed to the caller of our function._

Sections 1, 2 and 3 have the purpose of setting everything up so that section 4 can run. Section 4 then executes a set of patterns that do the work that `ft_bzero` is responsible for, and section 5 terminates the function accordingly.

As stated previously, you'll find these five sections in almost every function that you're going to encounter.

## The Ultimate List of Patterns

### Guard Patterns

#### The NULL Guard

The most frequently occuring guard pattern is the so-called _NULL Guard_ pattern. A _NULL Guard_ checks whether a variable (`ptr` in this case) is equal to `NULL` - and if it is, it terminates the function, by giving back an _error indicator_ as the return value (often `NULL`, `'\0'`, `0` or nothing at all, depending on the return type):

```c
if (ptr == NULL)
    return;
```

```c  
if (ptr == NULL)
    return NULL;
```

```c
if (ptr == NULL)
    return '\0';
```

```c  
if (ptr == NULL)
    return 0;
```

In some cases, the _NULL Guard_ also cleans up and destroys active data structures (just like the fifth section of a function), e.g.:

```c
if (ptr == NULL)
{
    free(some_other_pointer);
    return;
}
```

##### Pitfalls:
Sometimes there are cases of _NULL Guard_ patterns looking like this:

```c
if (!ptr)
    return NULL;
```

```c  
if (ptr == 0)
    return NULL;
```

> Writing _NULL Guard_ patterns that way is _dangerous_ since both versions of the code above check whether `ptr` is equal to `0` (meaning the numerical value zero, not the "pointer to nothing" referred to by `NULL`). Due to the fact that `NULL` is not the same thing as as the number zero - although on some platforms it might be represented by the same value - the resulting code will _not be portable_. Therefore, compiling it on a different machine might break the guard, leading to undefined and unwanted behavior.


### Initialization Patterns

#### The Default Init

Most variables are going to be initialized with their default value, depending on their type.

Integers:

```c
i = 0;
```

Characters:

```c  
c = '\0';
```

Pointers:

```c  
ptr = NULL;
```

Strings (`char*`):

```c
str = "Some static string";
```

Arrays are initialized using the [Array Initialization Pattern](#array-initialization).

#### Array Initialization
There are multiple ways to initialize arrays. The generalized form to initialize an array of an atomic type (e.g. `char`, `short`, `int`, `long`, ...) is by using a [BZero Pattern](#bzero-pattern).

#### BZero Pattern
In order to initialize a region of memory (as specified by its address and size) with zeroes, we use the _BZero Pattern_. This pattern is closely related to the standard library functions `bzero` and `memset`.

Initializing an array with the declaration `int array[n];` using the _BZero Pattern_ looks like this:

```c
i = 0;
while (i < n)
{
    array[i] = 0;
    i = i + 1;
}
```

A block of pointers or characters can be initialized by using `NULL` or `'\0'` as the according default value.

> It's often easier to write `bzero(array, 10 * sizeof(int));` or `memset(array, 0, 10 * sizeof(int));` instead of typing out the entire loop. Be aware though that the definition of `NULL` and the interna of floating point numbers might differ on multiple systems. Therefore, using `bzero` or `memset` for pointer arrays and arrays of type `float`, `double`, `struct` or `union` is _not recommended_.

### High-Level Patterns
