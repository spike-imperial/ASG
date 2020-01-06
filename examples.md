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

The other mode of the ASG solver is to take a partially complete ASG and
learn some extra ASG annotations, given a set of (positive and negative)
examples of strings.

In this case, the input file consists of three main components, which
are described below.

### Initial Grammar

The initial grammar can be any ASG. In this case, we will start with the
first grammar in the previous section (`a^ib^jc^k`) and try to learn the
annotations in the second grammar (`a^nb^nc^n`). This is written in the
input file exactly as before:

```
start -> as bs cs { }
as -> "a" as { } | { }
bs -> "b" bs { } | { }
cs -> "c" cs { } | { }
```

Note that the `|` notation is syntactic sugar for multiple production
rules.

We will also define a range of numbers and the `inc` predicate, where
`inc(x,y)` denotes that `y` is equal to `x+1`. The `#background`
notation below allows the user to define <em>global</em> predicates,
saving the need to write the same rule in every single production rule.

```
#background {
  num(0).   num(1).   num(2).   num(3).   num(4).
  num(5).   num(6).   num(7).   num(8).   num(9).
  num(10).  num(11).  num(12).  num(13).  num(14).
  num(15).  num(16).  num(17).  num(18).  num(19).

  inc(X, X+1) :- num(X), num(X+1).
}
```

### Examples

The next component of an ASG learning task is a set of positive examples
(which must be in the language of the learned grammar) and negative
examples (which must not be in the language of the learned grammar).

Each example string is expressed as a list of tokens. The positive
examples are annotated with `+` and the negative examples are annotated
with `-`.

The following set of examples are sufficient to learn the annotations in
the `a^nb^nc^n` ASG.

```
+ []
+ [ "a", "b", "c" ]
+ [ "a", "a", "b", "b", "c", "c" ]
- ["a"]
- ["a", "b"]
- ["b"]
- ["b", "c"]
- ["c"]
- ["a", "c"]
- [ "a", "a", "b", "c" ]
- [ "a", "b", "b", "c" ]
- [ "a", "b", "c", "c" ]
```
### Hypothesis Space

The final component of a learning task is a hypothesis space. This is
defined using mode declarations, which are similar to those used by the
[ILASP](http://www.ilasp.com) system -- we refer the reader to the ILASP
manual for introductory information on mode declarations. There are two
key differences:

1. The scope of a mode declaration can be given in a list following the
   mode declaration. This expresses which production rules can contain
   the mode declaration in their annotations; for instance, the first
   mode declaration can be used in all production rules except the
   first.

2. There are two different types of mode body declaration: `#modeba`'s
   are for those `#modeb`'s which are to be used with `@` annotations,
   or are defined in the same production rule and `#modebb`'s are to be
   used without `@` annotations (i.e. for those predicates which are
   defined in the `background` section.


The following mode declarations define a space of possible rules which
can be added to the annotations of the production rules. We call this
set of rules the <em>hypothesis space</em>.

```
#modeh(size(var(num))):[2,3,4,5,6,7].
#modeh(size(0)):[2,3,4,5,6,7].
#modeba(size(var(num))).
#modebb(1, inc(var(num), var(num))):[1,2,3,4,5,6,7].
```

This space can be constrained using an ILASP concept called <em>bias
constraints</em>, which are used to eliminate rules from the hypothesis
space. As these are an experimental feature, which are due to be changed
in the next version of ILASP, we omit discussion of them here. The bias
constraints in the `a^nb^nc^n` learning task are:

```
#bias(":- body(holds_at_node(size(anon(_)), _)).").
#bias(":- body(inc(anon(_), _)).").
#bias(":- body(inc(_, anon(_))).").
#bias(":- body(naf(holds_at_node(size(_), _))), head(holds_at_node(size(_), _)).").
```

### Example learning task usage

The ASG solver can be used to learn a set of annotations (which are in
a given hypothesis space) such that all of the positive examples (an no
negative examples) are in the language of the extended ASG.

The ASG learning task described above is available
[here](https://github.com/spike-imperial/ASG/tree/master/data/anbncn.lasg).
We can run the ASG solver on this input file using the command:

```
ASG --mode=learn --depth=4 aibjck.asg
```

If the ASG solver and ILASP are correctly installed, this command should
output the following ASG:

```
start -> as bs cs {
   :- not size(V0_user)@2, size(V0_user)@1.
   :- not size(V0_user)@3, size(V0_user)@1.
}
as -> "a" as {
  size(V2_user) :- inc(V0_user, V2_user), size(V0_user)@2.
}
as -> {
  size(0).
}
bs -> "b" bs {
  size(V2_user) :- inc(V0_user, V2_user), size(V0_user)@2.
}
bs -> {
  size(0).
}
cs -> "c" cs {
  size(V2_user) :- inc(V0_user, V2_user), size(V0_user)@2.
}
cs -> {
  size(0).
}

#lexer -> {
}

#background {
  num(0).
  num(1).
  num(2).
  num(3).
  num(4).
  num(5).
  num(6).
  num(7).
  num(8).
  num(9).
  num(10).
  num(11).
  num(12).
  num(13).
  num(14).
  num(15).
  num(16).
  num(17).
  num(18).
  num(19).
  inc(X_user, X_user + 1) :- num(X_user + 1), num(X_user).
}

```

This ASG is equivalent to `a^nb^nc^n`.

