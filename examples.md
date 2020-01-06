# Example ASG files

This page contains a few small examples of how to use the ASG solver.

## Run Mode

The simplest way to use the ASG solver is to run an existing Answer Set
Grammar (i.e. to use the solver to calculate the set of strings which
are in the language of the grammar). In this mode, all the input file
only needs to contain an ASG.

The simplest type of ASG is a Context Free Grammar (CFG) (with no
context sensitive annotations expressed in ASP). For example, consider
the following CFG:

```
start -> as bs cs { }

as -> "a" as { }
as -> { }

bs -> "b" bs { }
bs -> { }

cs -> "c" cs { }
cs -> { }

```

The empty pairs of curly braces are where the ASP annotations would
usually go, but in this example, there are no ASP annotations in the
grammar. This is the standard CFG `a^ib^jc^k` (i.e. any number of `a`'s,
followed by any number of `b`'s, followed by any number of `c`'s.

This ASG is available
[here](https://github.com/spike-imperial/FastLAS/tree/master/data/aibjck.asg).
We can run the ASG solver on this input file using the command:

```
ASG --mode=run --depth=4 aibjck.asg
```

The depth denotes how deep the parse trees of the grammar can be (as
this grammar is infinite, unless we bound the maximum depth, there are
infinitely many strings in its language, so the solver would not
terminate). If the ASG solver is correctly installed, it should output
the following set of strings:

```

a
aa
b
bb
ab
abb
aab
aabb
c
bc
cc
bcc
ac
aac
acc
aacc
abc
abcc
aabc
aabcc
bbc
abbc
aabbc
bbcc
abbcc
aabbcc
```

Note that the empty line is not a mistake. The empty line is output
because this grammar accepts the empty string.

## Learn Mode

TODO
