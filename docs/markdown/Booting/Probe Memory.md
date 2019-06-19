# Primer: Probe Memory

The first thing to do in booting is almost always checking the available memory. Why? Because without a complete map of the memory, it is impossible to decide where to load our kernel. So in this section, we are going to write a program in assembly to probe usable memory.
<!-- TODO: a name for this section -->
## Placeholder

### Why code in assembly?

It is certainly a pain to code in assembly for most people. There are some arguments that coding in assembly results in the program run faster than those coding in higher level language like C. That argument is not so much valid today as modern compliers are sophisticated enough to generate highly optimized program. The performance difference between a human written assembly and machine generated assembly is subtle and it would requires a highly skilled assembly programmer to outperform modern complier.
Then, why do we need to code in assembly for this section? The reason is C complier like gnu gcc always generates code for protected mode machine and we are working in real mode. So till someone write a complier for real mode, it would be necessary to write assembly. 
