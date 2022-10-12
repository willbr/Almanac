https://news.ycombinator.com/item?id=33134663
 	
# timonoko 12 hours ago | prev | next [–]

This is always annoying to read as they wasted ")" to some shit.

"(" should be: Push next word to X-stack

")" should be: pop and execute top of X-stack.

So "(+ m n)" is equal to "m n +"

And "(IF (< A B) (THEN C ELSE D)))" == "A B < IF C ELSE D THEN" 

# timonoko 9 hours ago | root | parent | next [–]

X is separate execution stack. It is very short depending how many brackets you use. When I made such Forth some 50 years ago, the "pop and execute" was one cycle on Nova 1200, no overhead. And x-stack was one 8-bit page.

# timonoko 12 hours ago | parent | prev | next [–]

IF-sentence was totally wrong. Per se.

(IF (< A B)) (THEN C ELSE D) looks about right. This is about readability, all else stays the same. You can avoid the minute overhead of "(" by not using it on a critical situation. 

