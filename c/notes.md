# C notes

# X Macro

Take a list of values and reuse them.

```c
#define LIST_OF_TYPES \
    X(char) \
    X(short) \
    X(int) \
    X(long)

```

```c
void
print_sizes(void) {
    #define X(varname) printf("sizeof(#varname) == %d\n", sizeof(varname))
    LIST_OF_TYPES
    #undef x
}
```
turns into
```c
void
print_sizes(void) {
    printf("sizeof(char) == %d\n", sizeof(char));
    printf("sizeof(short) == %d\n", sizeof(short));
    printf("sizeof(int) == %d\n", sizeof(int));
    printf("sizeof(long) == %d\n", sizeof(long));
}

```


## Refs
* <https://en.wikipedia.org/wiki/X_Macro>
* <https://www.drdobbs.com/the-new-c-x-macros/184401387>
* <https://digitalmars.com/articles/b51.html>
* <https://www.embedded.com/reduce-c-language-coding-errors-with-x-macros-part-1/>
* <https://en.wikibooks.org/wiki/C_Programming/Preprocessor_directives_and_macros#X-Macros>

# twitter

https://twitter.com/nothings/status/1770449205842329730

(BLM) Sean Barrett
@nothings
Protip:

In C, allocate arrays with malloc/calloc using sizeof() the variable, not the type:

   foo *arr = malloc(num_arr * sizeof(arr[0]));

not

   foo *arr = malloc(num_arr * sizeof(foo));

If you change the type of 'arr', the first is still correct, the second is wrong.

# References

* <https://wiki.xxiivv.com/site/ansi_c.html>
* <https://ftrv.se/3>
* <https://aiju.de/misc/c-style>

