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
[here](https://github.com/spike-imperial/ASG/tree/master/data/aibjck.asg).
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

### a^nb^nc^n

We can constrain the previous grammar, turning it into a <em>Context
Sensitive Grammar (CSG)</em> by adding some ASP annotations. These
annotations mostly use a similar syntax and semantics to standard ASP
solvers, except that each production rule essentially has a copy of
each atom in the language of the program (and these can have different
truth values). The annotations in one production rule can refer to the
atoms in the child nodes of the production rules by using the notation
`atom@index`, where `atom` is an atom and `index` is the index of the
child node (i.e. the position in which it occurs in the body of the
production rule`. For example, consider the following grammar:

```
start -> as bs cs {
  :- size(X)@1, not size(X)@2.
  :- size(X)@1, not size(X)@3.
}

as -> "a" as {
  size(X+1) :- size(X)@2.
}
as -> {
  size(0).
}

bs -> "b" bs {
  size(X+1) :- size(X)@2.
}
bs -> {
  size(0).
}

cs -> "c" cs {
  size(X+1) :- size(X)@2.
}
cs -> {
  size(0).
}
```

The three base cases (production rules 3, 5 and 7) express that the size
of the current string is 0. The three recursive cases express that the
size of the current string is one more than the size of the previous
string (which is accessed by checking the value of size in the second
child node of the production rule. The first production rule states that
the final lists of `a`'s, `b`'s and `c`'s must all be the same size,
meaning that the ASG represents the well known CSG `a^nb^nc^n`.

This ASG is available
[here](https://github.com/spike-imperial/ASG/tree/master/data/anbncn.asg).
We can run the ASG solver on this input file using the command:

```
ASG --mode=run --depth=4 aibjck.asg
```

If the ASG solver is correctly installed, it should output
the following set of strings:

```

abc
aabbcc
```


## Learn Mode

TODO
