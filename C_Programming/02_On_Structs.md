# On Structs

The `struct` keyword is one of the most important keywords in the C programming language. It allows us to define data structures that we can "talk to", leading to less complex code and therefore to easily maintainable programs.

## An example

Lets imagine that we were working on a function that reads all characters from the standard input and puts them into a string that will be returned after all characters have been read. The only functions available to us are `char getc()`, which returns the next character from the standard input, and `int has_more()`, which returns either `0` or `1` depending on whether there is input or not.

Now, we might me tempted to use `strjoin()` or other functions to store the characters that we are reading:

```c
char *read_all()
{
    char *input;
    char *temp;
    char str[2];
    
    input = strdup("");  // Start with an empty string (can be free'd)
    while (has_more())
    {
        str[0] = getc();
        str[1] = '\0';
        temp = input;  // So that we can free the previous string after joining
        input = strjoin(input, str);
        free(temp);
    }
    return (input);
}
```

While this approach works, the memory handling code takes up about half of our function. We constantly create new strings by calling `strjoin(...)`, and we `free(...)` them later in order to release the memory back to the operating system. This has an impact on the readability of our code, and if we're confronted with more complex tasks something like this is probably going to cause memory leaks and other problems since we often forget to add calls to `free(...)` in the right places.

To keep the code readable we should try to find a way to around this. Therefore, it's often crucial to separate control from implementation. The trick is to come up with data structures that don't force us to deal with the memory handling ourselves.

The `struct` keyword was designed for exactly this purpose.

> Remember: Calls to `malloc` and other functions that deal with memory make it hard for the reader to get the gist of what our function is actually doing. Let subfunctions handle the memory allocations for you - concentrate on what's really important!

## How Structs work

> The only thing a computer understands is zeros and ones. Everything else is just an abstraction.

You might have heard this phrase before. Everything that the C language provides you with - integers, floating point numbers, pointers or functions - is nothing more than a bunch of ever-changing bit patterns on the hardware. So when you declare an int (e.g. by typing `int x;`), this just means that the CPU will reserve 4 bytes of memory that you can work with, and assign the label `x` to them:

```
+--------+--------+--------+--------+
|00000000|00000000|00000000|00000000|
+--------+--------+--------+--------+
^
int x;
```

When you declare a byte array, things look similar:

```
+--------+--------+--------+--------+
|00000000|00000000|00000000|00000000|
+--------+--------+--------+--------+
^
char bytes[4];
```

In short: The computer doesn't care what we do with the bytes we were given, it just gives us what we asked for. These four bytes could represent an integer, a four-byte string, a floating point number, and so on.

Now, the types `int`, `float`, `char` as well as arrays and pointers were already created for us by the designers of the C language, but these are only suggestions on what we can do with our bytes. Thankfully, they also added the `struct` feature to let us introduce our own ideas.

## The 'struct' Keyword

The `struct` keyword allows us to group zero, one or more already existing data types (`int`, `float`, `char*`, or even other structs) into a new data type that can be copied around, assigned a value, allocated, deleted and modified.

Think of it like building a box with drawers, each drawer containing one or more bytes:

```
+--------------------------------+
|               O                |   <-  Drawer 'a', containing a char
+--------------------------------+
|               O                |   <-  Drawer 'b', containing an int
+--------------------------------+
|               O                |   <-  Drawer 'c', containing a float
+--------------------------------+
|               O                |   <-  Drawer 'd', containing a char*
+--------------------------------+
|               O                |   <-  Drawer 'e', containing a smaller box
+--------------------------------+
```

As with every box that you can build in real life, there is a sheet with instructions on how the box can be built. In C, the code for doing this looks like this:

```c
struct Box
{
    char               a;
    int                b;
    float              c;
    char*              d;
    struct SmallerBox  e;
};
```

Now, this looks suspiciously like the "list of ingredients" that we write at the beginning of each function, right?

That's because they're actually related: Both lists tell the C compiler what to do with the bytes we were given, and how to access them. Again: It's up to us to decide what we want - we're not limited by the types we were given!

## Patterns!

As always, here's a bunch of a few commonly used patterns regarding structs, together with some functions that operate on them. I will leave the actual implementation of these functions to the reader :wink:

> Hint: If you don't know what these structs represent, just google the corresponding heading. There are some great visualizations of linked lists and ringbuffers online!

### Dynamically Resizable Strings

```c
struct String
{
    char*        data;
    unsigned int size;
};

void  String_Create(struct String*, char*);
void  String_Destroy(struct String*);
char* String_AsCString(struct String*);
void  String_AppendChar(struct String*, char);
void  String_AppendCString(struct String*, char*);
void  String_AppendString(struct String*, struct String*);
// ...
```

### Linked List Nodes

```c
struct LinkedListNode
{
    <type>                  value;  // Replace <type> with any type that you desire!
    struct LinkedListNode*  next;
};

struct LinkedListNode* LinkedListNode_New(<type>, struct LinkedListNode*);
void LinkedListNode_Delete(struct LinkedListNode*);
void LinkedListNode_InsertAfter(struct LinkedListNode*, struct LinkedListNode*);
// ...
```

### Ringbuffers

```c
struct Ringbuffer
{
    char          chars[SIZE];
    unsigned int  read_head;
    unsigned int  write_head;
};

void Ringbuffer_Create(struct Ringbuffer*);
void Ringbuffer_Destroy(struct Ringbuffer*);
void Ringbuffer_WriteChar(struct Ringbuffer*, char);
char Ringbuffer_ReadChar(struct Ringbuffer*);
int  Ringbuffer_GetFreeSpace(struct Ringbuffer*);
```

### 3D Vectors

```c
struct Vector3D
{
    float  x;
    float  y;
    float  z;
};

void  Vector3D_Create(struct Vector3D*);
void  Vector3D_Set(struct Vector3D*, float, float, float);
void  Vector3D_SetX(struct Vector3D*, float);
float Vector3D_GetX(struct Vector3D*);
void  Vector3D_SetY(struct Vector3D*, float);
float Vector3D_GetY(struct Vector3D*);
void  Vector3D_SetZ(struct Vector3D*, float);
float Vector3D_GetZ(struct Vector3D*);
void  Vector3D_AddVector(struct Vector3D*, struct Vector3D*);
void  Vector3D_AddConstant(struct Vector3D*, float);
void  Vector3D_CrossMultiply(struct Vector3D*, struct Vector3D*);
// ...
void  Matrix_MultiplyWithVector3D(struct Matrix*, struct Vector3D*);
```

### ... and more!

Feel free to come up with your own structs, you are only limited by your imagination!



## Back to our Function

With this knowledge we can now rewrite our little function like this:

```c
char *read_all()
{
    struct String  string;
    char*          c_str;
 
    /* Initialization: Create the String */
	String_Create(&string);
    
    /* Main part of the function: Append chars until done, then convert to char* */
    while (has_more())
    {
        String_AppendChar(&string, getc());
    }
    c_str = String_AsCString(&string);
    
    /* Destroy the string before returning */
    String_Destroy(&string);
    return (c_str);
}
```

As you can see, all of the memory allocation is now handled by the `String_` functions instead of our `read_all()`. We only have to create the string at the beginning and destroy it at the end. Resizing the actual byte array that stores the chars inside of the string is nothing we have to deal with anymore, also the intermediate strings created by `strjoin(...)` are nothing we have to touch. The code is much more readable now, and the complexity of handling allocations has been outsourced to other functions. Yay! Structs are awesome! 