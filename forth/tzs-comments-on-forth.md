URL: https://news.ycombinator.com/item?id=13082825
	
# Comment 1

A fun and fairly simple project, with a surprisingly high ratio of usefullness to effort, is to write an interpreter for a concatenative language. Concatenative languages, like FORTH, can do a lot with very limited resources, making them good candidates for embedded systems.

If you want to play around with making your own concatenative language, it is actually surprisingly simple. Here is an overview of a step-by-step approach that can take you from a simple calculator to a full language with some optimization that would actually be quite reasonable to use in an embedded system.

So let's start with the calculator. We are going to have a data stack, and all operations will operate on the stack. We make a "dictionary" whose entries are "words" (basically names of functions). For each word in the dictionary, the dictionary contains a pointer to the function implementing that word.

We'll need six functions for the calculator: add, sub, mul, div, clr, and print. The words for these will be `+`, `-`, `x`, `/`, `clr`, and `print`. So our dictionary looks like this in C:

```c
struct DictEntry {
    char * word;
    int (*func)(void);
} dict[6] = {
    {"+", add}, {"-", sub}, {"x", mul},
    {"/", div}, {"clr", clr}, {"print", print}
};
```

We need a main loop, which will be something like this (pseudocode):

    while true
        token = NextToken()
        if token is in dictionary
            call function from that dict entry
        else if token is a number
            push that number onto the data stack

Write NextToken, making it read from your terminal and parse into whitespace separated strings, implement add, sub, mul, div, clr, and print, with print printing the top item on the data stack on your terminal, and you've got yourself an RPN calculator. Type `2 3 + 4 5 + x print` and you'll get 45.

OK, that's fine, but we want something we can program. To get to that, we first extend the dictionary a bit. We add a flag to each entry allowing us to mark the entry as either a C code entry or an interpreted entry, and we add a pointer to an array of integers, and we add a count telling the length of that array of integers. When an entry is marked as C code, it means that the function implementing it is written in C, and the "func" field in the dictionary points to the implementing function. When an entry is marked as interpreted, it means that the pointer to an array of integers points to a list of dictionary offsets, and the function is implemented by invoking the functions of the referenced dictionary entries, in order.

A dictionary entry now looks something like this:

```c
struct DictEntry {
    char * word;
    bool c_flag;
    void (*func)(void);
    int * def;
    int deflen;
}
```

# Comment 2

Now the main loop has to look something like this:

    while true
        token = NextToken()
        if token is in dictionary
            if dict entry is marked as C code
                call function from that dict entry
            else
                InterpWords(def, deflen)
        else if token is a number
            push that number onto the data stack

and InterpWords looks something like this

    void InterpWords(int * def, int deflen)
        while (deflen-- > 0)
            InterpOneWord(&dict[*def++])

    void InterpOneWord(dict * dp)
        if (dp->c_flag)
            (*dp->func)()
        else
            InterpWords(dp->def, dp->deflen)

With this, we can add new built-in functions defined in terms of already defined functions. Before we do any of that, though, we should add some more stack manipulation functions. Add at least `dup`, `drop`, and `swap`, which, respectively, push a copy of the current top of stack onto the stack, pop an entry from the stack, and exchange the top two items on the stack.

For reference, after adding those, the dictionary offsets of the 9 words we've defined are:

    0 +
    1 -
    2 x
    3 /
    4 clr
    5 print
    6 dup
    7 drop
    8 swap

and they are all marked as C code. Now let us add a built-in function `sq`, which squares the top item on the stack. To do that, add a new dictionary entry initialized like this:

```c
    {"sq", false, 0, sq_def, 2}
```

where sq\_def looks like this:

```c
    int sq_def[] = {6, 2}
```

That's marked as being interpreted rather the C code, so when the main loop finds it it will call InterpWords, passing it sqdef and 2, and InterpWords will end up invoking dup and x, with the result being the top of the stack is squared.

OK, that's all fine and dandy, but we want to be able to define functions in our language from our language. So far, we can define functions in our language, like sq, but we have to do it by editing a C data structure and recompiling.


# Comment 3

To do this, we'll need some changes to both InterpWords and/or InterpWord, and to the main loop. The basic idea is that we add a "compiling" mode. When we are in compiling mode the main loop does not execute the tokens it gets. Instead it just makes a list of the dictionary offsets of the words it would have executed. So in compiling mode, if you type `dup x`, the main loop doesn't actually call dup and x, it just makes the list {6, 2}. That is used to make the function we are compiling. We'll have to find some way to tell the interpreter what we want to call the new function. The interpreter can than make a new dictionary entry for it, referencing that list of dictionary offsets.

The above only covers words in compiling mode...what do we do when the main loop sees a number in compiling mode? That's where we need to extend InterpWords. We need to change the list of dictionary offsets it receives to be able to also specify numbers. Maybe something like this (taking advantage of the fact that dictionary offsets are always non-negative):

    void InterpWords(int * def, int deflen)
        while (deflen-- > 0)
            if (*def >= 0)
                InterpOneWord(&dict[*def++])
            else if (*def == -1)
                push def[1] onto data stack
                def += 2; deflen--;
            else error

A -1 in the word offset stream now means that the next item in the stream is a number to push onto the stack rather than a word offset. So now we can support functions like `dup x 10 +` that squares the top of the stack and adds 10, which would compile to `{6, 2, -1, 10, 0}`

We still aren't there, though, because we don't yet have a way to enter or leave compiling mode. That's easy. We'll add a new built-in function, `:`. We'll write this in C. What `:` does is read the next token, see if it is a legal word name and not already in use. If it is (legal) and isn't (in use), it creates a new dictionary entry for it, allocates space for the definition, and flips on the global compiling flag.

Great! Now we can start defining new words from in our language. Just one problem...how do we terminate compilation? The main loop is compiling everything, so how do we get it to execute the command to stop compiling and finish making the dictionary entry?

For that, we need yet another flag in the dictionary: the immediate flag.

```c
    struct DictEntry {
        char * word;
        bool c_flag;
        void (*func)(void);
        int * def;
        int deflen;
        bool immediate;
    }
```

Change the main loop so that when it sees a word with the immediate flag set, it executes it even if it is in compiling mode. With that, we can add a word `;`, with the immediate flag set, that turns off compiling mode and finalizes the compilation of the word that was being compiled. When we get these changes in, then we would be able to define `sq` from within the language instead of making it a built-in like we did earlier: `: sq dup x ;`

Of course, we probably want our language to support some kind of if/else construct and some kind of looping construct.

I bet you can see how this will be done. Most of the work is done in InterpWords. Much like we used -1 to indicate that the next entry is a number for the stack, we'll use -2 to indicate a goto. The entry after -2 will be the offset to go to. (Should it be an absolute offset or relative? Your choice). We'll use -3 to indicate a conditional goto. It works like -2, except that it only goes if the top of the stack is non-zero, and it pops the top of the stack. If the top of the stack is zero, it pops it, and then skips over the offset.

Here's how we could use this to implement a do...while kind of loop. First think of how we want it to look in our programs. It might be something like this `do <body of loop> <loop condition> while`. So we want the compiled code to look like this:

    <dict offset list for body of loop>
    <dict offset list for loop condition>
    -3
    offset of body of loop above

To make `do` and `while` work, we need to flag their dictionary entries as immediate words. So when we are compiling and hit `do` the code for `do` is executed. For now, let's make this C code. What the `do` code does is simply save the offset of where the next word will be compiled to. What the `while` code does is put the -3 in the compiled code, and then put the offset that was saved by the `do` into the compiled code.

You can do similar things for if/else constructs, for loops, and any other control flow you want: make the low level operations you need in InterpWords, and then make immediate words for use during compilation that gather and use the necessary information to write that control flow.

Now let's talk a bit about optimization. One thing you can do is some inlining. Consider this function `: cube dup sq x ;`. When compiling this, we could notice that `sq` is not C code, and put its definition inline, so we end up compiling cube as if it were written like this: `: cube dup dup x x ;`.

(BTW, remember earlier when I said it was up to you whether you use absolute or relative offsets in the low level control flow? Relative will make inlining easier if the inlined function has any branches).

Now if you are going to implement entirely in C or similar, that's about the end of the simple optimizations (at least as far as my feeble mind can handle). However, if you are willing to work in assembly language some, you can get much fancier.

You can make the lowest level built-ins, such as add, mul, dup, and similar assembly language. (In the following I'm going to continue to say C function, for consistency with the earlier stuff, but what it now means is a function that is not defined in our language). We can change the way non-C functions are defined from being a list of dictionary offsets to a pointer to actual code. So for `: sq dup x ;` instead of compiling that as a pointer to the list {6, 2}, we compile it as a pointer to a list of call instructions, followed by a return. For sq, that would look like this:

    call dup
    call mul
    ret
    

Cube would look like this (without optimization)

    call dup
    call sq
    call mul
    ret
    

and with sq inlined at the high level, it would become

    call dup
    call dup
    call mul
    call mul
    ret

For built-ins that are position independent assembly code, we can do another level of inlining and replace calls to them with their body.

Depending on the addressing modes your processor supports, this could end up with cube being as simple as four instructions (not counting the return), with all the calls eliminated!

You can go farther and do things like cache the top few stack items in registers, and add peephole optimizers that go over the compiled code to eliminate redundant operations.

The really cool thing is how easy bootstrapping this kind of thing is. You only need a small number of built-in assembly functions to give you enough to write the rest of the language in itself, and these systems tend to be small and perform reasonably well. These kind of languages can be great in embedded systems, and indeed that is where the granddaddy of this kind of language, FORTH, got its start.

