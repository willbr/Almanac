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

# printf %.* dynamic precision or field width


## dynamic precision

```c
int prec = 4;  // precision
double num = 3.14159265;

printf("%.*f\n", prec, num);  // prints: 3.1416
```

## field width

```c
const char *str = "Hello, World!";
int width = 5;

// Print only the first 'width' characters from the string
printf("First %d characters: '%.*s'\n", width, width, str);

return 0;
```

output:

    First 5 characters: 'Hello'

# comma operator

    (expression1, expression2)

* expression1 is evaluated
* expression2 is evaluated
* expression 2 is the result

```c
int a=1, b=2;
b = (a=2, 10);
```

the result is:

```c
a == 2
b == 10
```

The comma operator has lower precedence that most operators, so it needs parentheses.

```c
int x = (a = 5, b = 3);
```

## for loops

```c
    for (int i = 0, j = 10; i < j; i++, j--) {
        printf("i = %d, j = %d\n", i, j);
    }
```

# Cosmopolitan Libc

https://justine.lol/cosmopolitan/index.html

Cosmopolitan Libc makes C a build-anywhere run-anywhere language, like Java, except it doesn't need an interpreter or virtual machine. Instead, it reconfigures stock GCC and Clang to output a POSIX-approved polyglot format that runs natively on Linux + Mac + Windows + FreeBSD + OpenBSD + NetBSD + BIOS on AMD64 and ARM64 with the best possible performance. 

# References

* <https://wiki.xxiivv.com/site/ansi_c.html>
* <https://ftrv.se/3>
* <https://aiju.de/misc/c-style>

